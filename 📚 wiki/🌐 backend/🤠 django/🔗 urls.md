### DRF
```python
from users.views import RegistrationView, UserViewSet, NotificationViewSet

router = SimpleRouter()
# первый способ подключения Viewset
router.register("notifications", NotificationViewSet, basename="notifications")

urlpatterns = [
    path('registration/', RegistrationView.as_view(), name="registration"),
    path('profile/<int:pk>/', UserViewSet.as_view(
	    {'get': 'retrieve', 'put': 'update', 'delete': 'destroy'}
	), name="profile"), # второй способ подключения Viewset
]
```
### Django
```python
from django.urls import path

from web.views import MainView

urlpatterns = [
    path("", MainView.as_view(), name="main")
]
```
### Common
```python
from django.urls import path, include

urlpatterns = [
    path("", include("web.urls")),
]
```