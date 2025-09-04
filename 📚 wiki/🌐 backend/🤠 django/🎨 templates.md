#### Location
Создаются внутри `<app>/templates/<app>/<here>`
#### Base template example
```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    {% block styles %}
        <link rel="stylesheet" href="{% static 'web/bootstrap.min.css' %}">
        <link rel="stylesheet" href="{% static 'web/style.css' %}">
    {% endblock %}
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <div class="navbar-nav ms-auto me-5">
        {% if user.is_authenticated %}
            <a class="nav-link" href="{% url 'profile' %}">Профиль</a>
        {% else %}
            <a class="nav-link" href="{% url 'login' %}">Войти</a>
        {% endif %}
    </div>
    {% block body %}{% endblock %}
    <script src="{% static 'web/bootstrap.bundle.min.js' %}"></script>
</body>
</html>
```
#### Extended page example
```html
{% extends "web/base.html" %}

{% block title %}{{ title }}{% endblock %}

{% block body %}
    <div class="container d-flex align-items-center">
        <div class="centered w-100">
            <div class="form w-50 mx-auto p-3 rounded-3">
                {% include "web/snippets/form.html" with button="Сохранить" type="submit" %}
            </div>
            <div class="text-center mt-3">
                <form method="post" action="{% url 'delete_reminder' object.id %}">
                   {% csrf_token %}
                    <input type="submit" class="btn btn-danger" value="Удалить">
                </form>
            </div>
        </div>
    </div>
{% endblock %}
```
#### Form template
```html
<form action="" method="post" id="{{ id }}">
    {% csrf_token %}

    {% for field in form %}
        {{ field.label_tag }}
        {{ field }}
        {% if field.errors.0 %}
            <script>
                let name = '{{ field.name }}';
                let field = document.getElementById("id_" + name);
                field.style.border = "1px solid red";
                field.className = "mb-0 form-control";
            </script>
            <p class="text-danger m-0 mb-2 text-center fs-6" style="font-size: 2px">{{ field.errors.0 }}</p>
            {{ field.id }}
        {% endif %}
    {% endfor %}
    <div class="text-center">
        <button class="btn btn-primary" id="add" type="{{ type }}">{{ button }}</button>
    </div>
</form>
```