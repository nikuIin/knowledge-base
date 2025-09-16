[[FastAPI.canvas|FastAPI]]
[[SQLAlchemy ORM]]

******
> **filter** = **where**


Давайте рассмотрим подробный пример построения динамических запросов с операторами `AND` и `OR` в SQLAlchemy.

### Задача

Предположим, нам нужно написать запрос для поиска сотрудников по нескольким параметрам:

- Поиск по фамилии сотрудника.
- Поиск по должности.
- Возможность ограничить диапазон зарплат.
- Опциональный поиск по городу проживания.

При этом важно учесть следующие требования:

- Все указанные условия (например, фамилия и должность) являются обязательными и объединяются через оператор `AND`.
- Но есть дополнительная возможность ограничивать выбор сотрудников по диапазону зарплаты или по городскому региону через оператор `OR`.

### Таблица сотрудников (Employee):

| Column          | Type      |
|-----------------|-----------|
| employee_id     | Integer   |
| first_name      | String    |
| last_name       | String    |
| position        | String    |
| salary          | Float     |
| city            | String    |

### 🛠️ Пример реализации на Python с SQLAlchemy

```python
from sqlalchemy import create_engine, select, and_, or_
from sqlalchemy.orm import declarative_base, Session
from sqlalchemy import Column, Integer, String, Float

Base = declarative_base()

class Employee(Base):
    __tablename__ = 'employees'
    employee_id = Column(Integer, primary_key=True)
    first_name = Column(String)
    last_name = Column(String)
    position = Column(String)
    salary = Column(Float)
    city = Column(String)

engine = create_engine('sqlite:///:memory:')
Base.metadata.create_all(engine)

def dynamic_employee_search(session: Session, last_name=None, position=None, min_salary=None, max_salary=None, city=None):
    """
    Функционал поиска сотрудников по множеству параметров.
    """
    query = select(Employee)

    # Основная группа обязательных условий (AND)
    conditions_and = []

    # Имя обязательно проверяем
    if last_name:
        conditions_and.append(Employee.last_name == last_name)

    # Должность также обязательное условие
    if position:
        conditions_and.append(Employee.position == position)

    # Применяем базовые обязательные условия через AND
    if conditions_and:
        query = query.where(and_(*conditions_and))

    # Группируем дополнительные условия через OR
    conditions_or = []

    # Диапазон зарплат (опционально)
    if min_salary and max_salary:
        conditions_or.append((Employee.salary.between(min_salary, max_salary)))
    elif min_salary:
        conditions_or.append((Employee.salary >= min_salary))
    elif max_salary:
        conditions_or.append((Employee.salary <= max_salary))

    # Город проживания (опционально)
    if city:
        conditions_or.append(Employee.city == city)

    # Применяем дополнительные условия через OR
    if conditions_or:
        query = query.where(or_(*conditions_or))

    return session.scalars(query).all()

# Пример использования
with Session(engine) as session:
    employees = dynamic_employee_search(
        session,
        last_name="Иванов",
        position="Менеджер",
        min_salary=50000,
        city="Москва"
    )

for emp in employees:
    print(f"{emp.first_name}, {emp.last_name}, {emp.position}, зарплата: {emp.salary}")
```

### 🗃️ Разбор примера:

1. **Обязательные условия (`AND`)**
   - Если указаны значения для поля `last_name` и `position`, они включаются в группу обязательных условий, объединяемых через оператор `AND`.

2. **Дополнительные условия (`OR`)**
   - Для дополнительной группы условий используются операторы сравнения по зарплате (`>=`, `<=`, `BETWEEN`) и поиск по городу (`city`).
   - Эти условия объединены через оператор `OR`.

### 😎 Резюме:

Используя возможности SQLAlchemy, вы можете гибко управлять структурой ваших запросов, создавая динамические условия, состоящие из произвольного числа требований, связанных операторами `AND` и `OR`. Такой подход существенно повышает удобство разработки и поддерживает масштабирование системы в будущем.