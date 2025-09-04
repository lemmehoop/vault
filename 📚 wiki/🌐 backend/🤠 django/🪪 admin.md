[Book with tips on django admin](https://drive.google.com/file/d/1xu2r47EsYYWnMskAIGoJcCuAoSXEHCCP/view?usp=sharing)
##### Easiest example
```python
class BookAdmin(admin.ModelAdmin):
	list_display = ('id', 'name', 'author', 'is_available', 'delete')
	
	def delete(self, obj):
		view_name = "admin:{}_{}_delete".format(
			obj._meta.app_label, obj._meta.model_name
		)
		link = reverse(view_name, args=[book.pk])
		html = '<button onclick="location.href=\'{link}\'" value="Delete" />'
		return format_html(html)
```
#### Cool button on change_list.html
```html
{% block object-tools-items %}
    <li>
        <a href="{% url 'admin:custom_pages_cleantaskresults_changelist' %}" class="deletelink btn-danger">
            Удалить все результаты задач
        </a>
    </li>
    {{ block.super }}
{% endblock %}
```

