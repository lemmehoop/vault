```python
@pytest.fixture  
def create_user_interactor() -> CreateUserInteractor:  
    return CreateUserInteractor(  
        user_repository=create_autospec(interfaces.UserRepository),  
        uuid_generator=create_autospec(interfaces.UUIDGenerator),  
        dt_generator=create_autospec(interfaces.DTGenerator),  
    )  
  
  
async def test_create_user(  
    create_user_interactor: CreateUserInteractor,  
    faker: Faker,  
) -> None:  
    create_user_interactor._user_repository.get_by_username.return_value = None  
  
    username = faker.user_name()  
    await create_user_interactor(username=username)  
  
	create_user_interactor._user_repository.\
		get_by_username.assert_called_once_with(username=username)  
    create_user_interactor._user_repository.create.assert_called_once_with(  
        user=User(  
            uuid=create_user_interactor._uuid_generator(),  
            username=username,  
            created_at=create_user_interactor._dt_generator(),  
        ),  
    )
```
***
```python
@pytest.fixture  
async def user_factory(session: AsyncSession, faker: Faker) -> Callable:  
    async def create_user(  
        uuid: UUID | None = None,  
        username: str | None = None,  
        created_at: datetime | None = None,  
        _create: bool = False,  
    ) -> User:  
        user = User(  
            uuid=uuid or faker.uuid4(cast_to=None),  
            username=username or faker.user_name(),  
            created_at=created_at or faker.date_time(),  
        )  
  
        if _create:  
            await session.execute(  
                insert(UserModel).values(**asdict(obj=user)),  
            )  
            await session.commit()  
  
        return user  
  
    return create_user
```