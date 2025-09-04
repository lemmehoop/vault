#### Install
```bash
pip install drf-yasg
```
#### Add to installed apps
```python
INSTALLED_APPS = [..., drf_yasg, ...]
```
#### Configure in settings
```python
schema_view = get_schema_view(
   openapi.Info(
      title="Notes API",
      default_version='v1',
      description="django test project",
      contact=openapi.Contact(email="developer@notes.com"),
   ),
   public=True,
   permission_classes=[],
)

urlpatterns = [
   path('swagger<format>/', schema_view.without_ui(cache_timeout=0), name='swagger-json'),
   path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='swagger-ui'),
   path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), name='swagger-redoc'),
   ...
]
```