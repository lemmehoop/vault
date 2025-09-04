#### Work model
**Prefork** - worker starts and by default creates it's copies (the amount of CPUs)
Better to use small tasks that can run concurrently
#### Start commands
```bash
celery --app <app_name> worker --loglevel=INFO # --pool=solo
celery --app <app_name> beat
```
#### Settings
```python
CELERY_BROKER_URL = os.environ.get("CELERY_BROKER_URL", "redis://localhost:6379/0")
CELERY_BROKER_TRANSPORT_OPTIONS = {"is_secure": os.environ.get("CELERY_BROKER_IS_SECURE", "false").lower() == "true"}
```
#### Configuring in core/celery.py
```python
import os
from celery import Celery

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "<core>.settings")

app = Celery("kapoot")
app.config_from_object("django.conf:settings", namespace="CELERY")

app.autodiscover_tasks()
```
#### Task example
```python
# так создаем тогда, когда чисто для одного проекта
@app.task
def send_password_reset_email(link: str, recipient_email: str):
    send_mail(
        subject="Password Reset for {title}".format(title="Kapoot!"),
        message="",
        html_message=render_to_string("password_reset_message.html", context={"link": link}),
        from_email=settings.EMAIL_HOST_USER,
        recipient_list=[recipient_email]
    )

# так создаем тогда, когда хотим переиспользовать код в другом проекте
# или когда пишем либу, и у нас нет доступного приложения app
@shared_task(name="send_notification")
def send_notification(notification_id):
    link = "{}".format(os.environ.get("FRONTEND_HOST", "http://localhost:5173"))
    notification = models.Notification.objects.filter(id=notification_id).first()
	...
```
#### Troubleshooting
1. Заходим в контейнер c *Redis*
2. Запускаем `redis-cli`
3. `LLEN celery` - количество задач в очереди
   `LRANGE celery 0 -1` - вывести все задачи
   `LRANGE celery -10 -1` - вывести последние задачи
   `DEL celery` - очистить очередь
   
Подсчитать задачи по имени
```bash
tasks=$(redis-cli LRANGE celery 0 -1)
task_name='my_task_name'
task_count=$(echo "$tasks" | grep -c "$task_name")

echo "Количество задач с именем $task_name: $task_count"
```
### Deduplicated task example
```python
class DeduplicatedTask(Task):  
    """  
    Deduplicated Celery task implementation.    It means that only one unique (name, args and kwargs combination) task will be running at given time.  
    For usage just set the `base=DeduplicatedTask` argument in celery task decorator.  
    It uses redis's `setnx` function for setting locks. We can't use redis.Lock:    Lock is being set in `apply_async`, that is being called by main process (Django or Celery Beat),    but `on_<result>` method, that releases lock, is called in worker process.    Redis won't allow to release lock that was created in other process.    """  
    def get_lock_name(self, args: tuple | None = None, kwargs: dict | None = None):  
        """Getting lock name with prefix, task name, args and kwargs set"""  
        args: tuple | list = self.request.args or args or ()  # request.args can be list  
        kwargs: dict = self.request.kwargs or kwargs or {}  
  
        return f"{LOCK_NAME_PREFIX} {self.name} | {tuple(args)} | {kwargs}"  
  
    @staticmethod  
    def acquire_lock(lock_name: str) -> bool:  
        """Setting lock by its name"""  
        # 'nx' - lock will be set only if it doesn't exist  
        # 'ex' - expiration time for lock (to avoid deadlock)        return redis_client.set(  
            name=lock_name, value="locked", nx=True, ex=settings.CELERY_LOCK_EXPIRE_TIME  
        )  
  
    @staticmethod  
    def release_lock(lock_name: str) -> None:  
        """Removing lock by name"""  
        redis_client.delete(lock_name)  
  
    def apply_async(self, *args, **kwargs):  
        full_name: str = self.get_lock_name(args[0], args[1])  
        is_lock_acquired: bool = self.acquire_lock(lock_name=full_name)  
  
        logger.info(  
            f"Lock status for '{full_name}': '{'FREE' if is_lock_acquired else 'USED'}'"  
        )  
  
        if not is_lock_acquired:  
            logger.info(f"Skipping duplicating task: '{full_name}'")  
            return EagerResult("", None, "IGNORED")  
  
        return super(DeduplicatedTask, self).apply_async(*args, **kwargs)  
  
    def on_success(self, retval, task_id, args, kwargs):  
        self.release_lock(self.get_lock_name())  
        super(DeduplicatedTask, self).on_success(retval, task_id, args, kwargs)  
  
    def on_failure(self, exc, task_id, args, kwargs, einfo):  
        self.release_lock(self.get_lock_name())  
        super(DeduplicatedTask, self).on_failure(exc, task_id, args, kwargs, einfo)  
  
    def on_retry(self, exc, task_id, args, kwargs, einfo):  
        self.release_lock(self.get_lock_name())  
        super(DeduplicatedTask, self).on_retry(exc, task_id, args, kwargs, einfo)    
```