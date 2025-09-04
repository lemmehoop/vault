#### Set the configs
```python
EMAIL_HOST = os.environ.get("EMAIL_HOST")
EMAIL_HOST_USER = os.environ.get("EMAIL_HOST_USER")
EMAIL_HOST_PASSWORD = os.environ.get("EMAIL_HOST_PASSWORD")
EMAIL_PORT = os.environ.get("EMAIL_PORT")
EMAIL_USE_SSL = True
DEFAULT_FROM_EMAIL = os.environ.get("DEFAULT_FROM_EMAIL", EMAIL_HOST_USER)
```
##### Render message and send it
```python
rendered = render_to_string(
    "web/emails/comment_notification.html",
    {
        "user_name": note.user.name,
        "link": link, # создается в другом месте фнукцией build_absolute_uri
        "note_title": note.title,
        "comment_text": note_comment.text,
    },
)

send_mail(
	"Новый комментарий", "", settings.DEFAULT_FROM_EMAIL,
	[note.user.email], html_message=rendered
)
```