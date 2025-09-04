##### Query with field that is created automatically
```python
Test.objects.annotate(
	is_passed=Case(When(id__in=passed_tests, then=True), default=False)
).all()
```
##### Query with filter and returning as JSON
```python
Appointment.objects.filter(start_time__isnull=False)
            .annotate(date=F("start_time__date"))
            .annotate(json_data=JSONObject(
				id="id", specialist="specialist", type="type", start_time="start_time"
			))
            .values("date")
            .annotate(appointments=ArrayAgg("json_data"))
```
##### DB connection setting
```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.environ.get("DB_NAME", "notes"),
        "USER": os.environ.get("DB_USER", "notes"),
        "PASSWORD": os.environ.get("DB_PASSWORD", "notes"),
        "HOST": os.environ.get("DB_HOST", "localhost"),
        "PORT": int(os.environ.get("DB_PORT", 5432)),
    }
}
```
##### Abstract class
```python
class BaseModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        abstract = True
        using = "default". # name of db from connection settings
```
##### Cool custom User model
```python
class UserManager(DjangoUserManager):
    def _create_user(self, email, password, commit=True, **extra_fields):
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        if commit:
            user.save(using=self._db)
        return user

    def create_user(self, email, password=None, **extra_fields):
        return self._create_user(email, password, **extra_fields)

    def create_superuser(self, email, password=None, **extra_fields):
        return self._create_user(email, password, role=Role.admin, **extra_fields)


class User(AbstractBaseUser, PermissionsMixin):
    objects = UserManager()

    email = models.EmailField(unique=True, verbose_name="адресом почты")
    name = models.CharField(max_length=63, null=True, blank=True)
    role = models.CharField(choices=Role.choices, max_length=10, default=Role.user)
    organization = models.ForeignKey(Organization, on_delete=models.SET_NULL, null=True)
    subscription = models.ForeignKey(Subscription, on_delete=models.SET_NULL, null=True)
    sub_expire_at = models.DateTimeField(null=True, blank=True)

    @property
    def is_superuser(self):
        return self.role == Role.admin

    @property
    def is_staff(self):
        return self.role == Role.admin

    EMAIL_FIELD = "email"
    USERNAME_FIELD = "email"
    REQUIRED_FIELDS = []

    class Meta:
        verbose_name = "пользователь"
        verbose_name_plural = "пользователи"
```
##### Constraints
```python
class Event(models.Model):
    start_date = models.DatetimeField() 
    end_date = models.DatetimeField()

    class Meta:
        constraints = [
            CheckConstraint(
                check = Q(end_date__gt=F('start_date')), 
                name = 'check_start_date',
            ),
        ]
		unique_together = ('start_date', 'end_date',)
```
##### OuterRef
```python
AttackVector.objects.annotate(  
    latest_launch_id=Subquery(  
        queryset=AttackLaunch.objects.filter(  
            attack_vector_id=OuterRef("pk")  
        ).order_by("-created_at").values("id")[:1],  
    )  
).all()
```