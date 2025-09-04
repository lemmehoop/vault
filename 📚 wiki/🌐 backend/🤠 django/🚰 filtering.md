### DRF
```python
import coreapi
from django_filters import FilterSet, rest_framework, NumberFilter


class AppointmentsSpecialistFilterSet(FilterSet):
    specialist = NumberFilter(field_name="specialist")

    class Meta:
        fields = ["specialist"]

class AppointmentsSpecialistFilterBackend(rest_framework.DjangoFilterBackend):
    def filter_queryset(self, request, queryset, view):
        specialist = request.query_params.get("specialist", None)
        kwargs = {"specialist_id": specialist}
        if specialist:
            queryset = queryset.filter(**kwargs)

        return queryset

    def get_schema_fields(self, view):
        return [coreapi.Field(
			name="specialist", description="Specialist ID",
			required=False, location="query")]
```
And in View
```python
class AppointmentsListView(ListAPIView):
    permission_classes = [AllowAny]
    serializer_class = AppointmentBaseInfoSerializer
    filter_backends = (AppointmentsSpecialistFilterBackend,)
    filterset_class = AppointmentsSpecialistFilterSet
```
### Django
```python
class AliasFilterSet(django_filters.FilterSet):
    search = django_filters.CharFilter(
        method="filter_search",
        label="",
        widget=forms.TextInput(attrs={"class": "form-control", "placeholder": "Search..."})
    )

	def filter_search(self, queryset, name, value):
			return queryset.filter(Q(**{"name__icontains": value}))

    class Meta:
        model = Alias
        fields = ("search",)
```
And in View
```python
class AliasListView(LoginRequiredMixin, BaseFilterView, ListView):
    model = Alias
    template_name = "shortener/alias_list.html"
    filterset_class = AliasFilterSet

    def get_queryset(self):
        return Alias.objects.filter(user=self.request.user).order_by("-created_at")
```