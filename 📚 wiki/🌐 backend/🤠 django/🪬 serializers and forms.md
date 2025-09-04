#### Basic example of DRF serializer
Also you can use *validate_field*, *save*, *validate* and etc
```python
class QuizWithQuestionsSerializer(serializers.ModelSerializer):
	# получение определенного поля через существующее поле
	user = serializers.CharField(source="user.email")
	# получение всех атрибутов (определенного поля), которые записаны в модели как many-to-many
	tags = serializers.SlugRelatedField(many=True, slug_field="title", queryset=Tag.objects.all())
	# получение объектов, у которых есть поле foreign key с данным объектом
    questions = QuestionSerializer(
        many=True,
        read_only=True
    )
	# поле, которое высчиывается автоматически
    user_rating = serializers.SerializerMethodField("get_user_rating", read_only=True)

    def get_user_rating(self, obj):
        user_rating = Reaction.objects.filter(
	        Q(quiz=obj) & Q(user=self.context["request"].user)
		).first()
        return {"points_number": user_rating.points_number, "id": user_rating.id} if user_rating else None

    class Meta:
        model = Quiz
        fields = ["tags", "questions", "user_rating"]
```
#### Basic example of Django form
```python
class UserForm(forms.ModelForm):
	safe = forms.BooleanField()

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.label_suffix = "" # приставка после лейбла
        self.fields["email"].widget.attrs["readonly"] = "" # замьюченный инпут
        self.fields["password"].required = False # необязательный атрибут
        for attr, value in self.fields.items():
            if attr == "name":
                self.fields[attr].widget.attrs.update({"class": "form-control mb-3"})
            else:
                self.fields[attr].widget.attrs.update({"class": "form-control mb-2"})

    def clean(self):
        cleaned_data = super().clean()
        unhashed_password = cleaned_data["password"]
        if unhashed_password:
            cleaned_data["password"] = make_password(unhashed_password)
        else:
            cleaned_data.pop("password")
        return cleaned_data

    class Meta:
        model = User
        fields = ("email", "name", "password")
        widgets = {"password": forms.PasswordInput()}
```