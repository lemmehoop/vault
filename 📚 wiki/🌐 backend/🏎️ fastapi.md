*Starlette* - отвечает за работу с веб
*Pydantic* - отвечает за сериализацию и валидацию данных
##### Path params
```python
@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```
Можно указать типы данных
```python
@app.get("/items/{item_id}")
async def read_item(item_id: int):
	return {"item_id": item_id}
```
##### Поиск совпадающих url
Идет по порядку написания, поэтому если что-то может совпасть, то нужно объявить это раньше
##### Query params
Необязательные аргументы
```python
@app.get("/item")
async def read_item(limit: int | None = 10, offset: int | None = None):
	return {"item_id": item_id}
```
##### Post request + pydantic model
```python
class Item(BaseModel):  
    title: str = Field(max_length=5)
    price: float = Field(ge=0) # чтобы было больше нуля

@app.post("/items/", response_model=list[Item])
async def read_item(item: Item):
	return {"item": item}
```