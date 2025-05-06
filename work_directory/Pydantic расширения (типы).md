[[FastAPI.canvas|FastAPI Union]]
[[Pydantic]]
* `conlist` — обертка вокруг обычного `list`, которая добавляет валидацию, такую как:
	  1. Минимальная и максимальная длина (`min_length`, `max_length`)
	  2. Тип (`item_type`)
* `conset` — тоже самое как и `conlist`, но для `set`
```python
from pydantic import BaseModel, conlist

class SomeModel(BaseModel):
	height: int
	width: int
	point_x: conlist(int, min_length=1, max_length=height)
	point_y: conset(int, min_length=1, max_length=widht)
	
```
