##### Работа с сессиями через dishka
В коде приложения:
```python
class InfrastructureProvider(dishka.Provider):
    @dishka.provide(scope=dishka.Scope.APP)
    async def session_maker(self, config: Config) -> async_sessionmaker:
        engine = create_async_engine(url=config.postgres.url)
        return async_sessionmaker(bind=engine)

    @dishka.provide(scope=dishka.Scope.REQUEST)
    async def get_session(
	    self, session_maker: async_sessionmaker
	) -> AsyncIterable[AsyncSession]:
        async with session_maker() as session:
            yield session
```

В тестах:
```python
@pytest.fixture
async def session_maker() -> AsyncGenerator[async_sessionmaker]:
    postgres_config = PostgresConfig()
    engine = create_async_engine(url=postgres_config.url)
    async with engine.connect() as connection, connection.begin() as transaction:
        yield async_sessionmaker(
	        bind=connection, join_transaction_mode="create_savepoint"
		)
        await transaction.rollback()

@pytest.fixture
async def session(
	session_maker: async_sessionmaker
) -> AsyncGenerator[AsyncSession]:
    async with session_maker() as session:
        yield session
```
##### Clean Architecture Repository
```python
class UserRepository(interfaces.UserRepository):
    def __init__(self, session: AsyncSession) -> None:
        self._session = session

    async def get_by_username(self, username: str) -> User | None:
        query = text(
            """
            SELECT uuid, username, created_at
            FROM users
            WHERE username = :username
            LIMIT 1
            """,
        )
        result = await self._session.execute(
	        statement=query, params={"username": username}
		)
        row = result.fetchone()
        if not row:
            return None
        return self._from_row_to_user(row=row)
```
##### Related objects
`lazy` - как по умолчанию в django: проблема n+1 запроса при доступе к объектам
`selectin` - примерно как prefetch в django: делает один дополнительный запрос чтобы получить ассоциированные объекты для всех родительских объектов (фильтр с `where`)
`joined` - настоящий prefetch, потому что использует `join` для ассоциаций.

`selectin` может быть лучше, если связь не один к одному, потому что если много атрибутов, то в `join` будет дублироваться оригинальный объект для каждого дочернего в `joined`.
##### Создание асинхронного подключения
```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker  
from sqlalchemy.orm import declarative_base  
  
import settings  
  
engine = create_async_engine(
	f"postgresql+asyncpg://{USER}:{PASSWORD}@{HOST}:{PORT}/{NAME}",
	future=True
)  
# expire_on_commit means that after commit object will be expired and accessing them will trigger new query into database
async_session = async_sessionmaker(bind=engine, expire_on_commit=True)
Base = declarative_base()

async def get_db() -> AsyncGenerator[AsyncSession, None]:  
    async with AsyncSessionLocal() as db:  
        yield db
```
##### Alembic base commands
```sh
alembic init migrations
alembic revision --autogenerate -m "migration messager"
alembic upgrade heads
```
##### Basic User model 
```python
class User(Base):  
    __tablename__ = "user"  
  
    id = mapped_column(Integer, primary_key=True, autoincrement=True)  
    email = mapped_column(String(127), unique=True, nullable=False)  
    name = mapped_column(String(31), nullable=True)  
    surname = mapped_column(String(63), nullable=True)  
    role = mapped_column(Enum(Role))  
    registered_at = mapped_column(TIMESTAMP, default=datetime.now)  
  
    @validates("email")  
    def validate_email(self, key, value):  
        try:  
            email = validate_email(value, check_deliverability=False).normalized 
        except EmailNotValidError as e:  
            raise ValueError(str(e))  
        else:  
            return email
```
##### Committing changes, closing session and rolling back if errors occuring
```python
Session = sessionmaker(engine)

with Session.begin() as session:
    session.add(some_object)
    session.add(some_other_object)
```
##### Basic models with foreign keys
```python
from datetime import datetime  
from typing import List  
  
from sqlalchemy import (  
    Integer,  
    String,  
    DateTime,  
    func,  
    ForeignKey,  
    BIGINT,  
    UniqueConstraint,  
)  
from sqlalchemy.orm import mapped_column, Mapped, relationship  
  
from src.database.main import Base  
  
  
class BaseModel(Base):  
    __abstract__ = True  
  
    id: Mapped[int] = mapped_column(Integer, primary_key=True, index=True)  
  
  
class CreatedAtMixin(Base):  
    __abstract__ = True  
  
    created_at: Mapped[datetime] = mapped_column(  
        DateTime(timezone=True), server_default=func.now()  
    )  
  
  
class UpdatedAtMixin(Base):  
    __abstract__ = True  
  
    updated_at: Mapped[datetime] = mapped_column(  
        DateTime(timezone=True), onupdate=func.now(), server_default=func.now()  
    )  
  
  
class User(BaseModel, CreatedAtMixin, UpdatedAtMixin):  
    __tablename__ = "user"  
  
    username: Mapped[str] = mapped_column(String, unique=True, nullable=False)  
  
    goals: Mapped[List["Goal"]] = relationship(  
        "Goal", back_populates="user", cascade="all, delete-orphan"  
    )  
    accounts: Mapped[List["Account"]] = relationship(  
        "Account", back_populates="user", cascade="all, delete-orphan"  
    )  
  
    def __str__(self):  
        return f"<User: [{self.id}] {self.username}>"  
  
  
class Goal(BaseModel, CreatedAtMixin, UpdatedAtMixin):  
    __tablename__ = "goal"  
  
    user_id: Mapped[int] = mapped_column(  
        ForeignKey("user.id", name="fk_user_goal"), nullable=False  
    )  
    title: Mapped[str] = mapped_column(String, unique=True, nullable=False)  
    desired_amount: Mapped[int] = mapped_column(BIGINT, default=0, nullable=False)  
    expected_by: Mapped[datetime] = mapped_column(  
        DateTime(timezone=True), nullable=False  
    )  
  
    user: Mapped["User"] = relationship("User", back_populates="goals", lazy="joined")  
    accounts: Mapped[List["Account"]] = relationship("Account", back_populates="goal")  
  
    __table_args__ = (  
        UniqueConstraint("user_id", "title", name="unique_user_goal_title"),  
    )  
  
    def __str__(self):  
        return f"<Goal: [{self.id}] {self.title} ({self.user.username})>"  
  
    __repr__ = __str__  
  
  
class Account(BaseModel, CreatedAtMixin, UpdatedAtMixin):  
    __tablename__ = "account"  
  
    user_id: Mapped[int] = mapped_column(  
        ForeignKey("user.id", name="fk_user_account"), nullable=False  
    )  
    goal_id: Mapped[int] = mapped_column(  
        ForeignKey("goal.id", ondelete="SET NULL", name="fk_goal_account"),  
        nullable=True,  
    )  
    title: Mapped[str] = mapped_column(String, unique=True, nullable=False)  
    current_amount: Mapped[int] = mapped_column(Integer, default=0, nullable=False)  
  
    user: Mapped["User"] = relationship(  
        "User", back_populates="accounts", lazy="joined"  
    )  
    goal: Mapped["Goal"] = relationship("Goal", back_populates="accounts")  
  
    __table_args__ = (  
        UniqueConstraint("user_id", "title", name="unique_user_account_title"),  
    )  
  
    def __str__(self):  
        return f"<Account: [{self.id}] {self.title} ({self.user.username})>"  
  
    __repr__ = __str__
```