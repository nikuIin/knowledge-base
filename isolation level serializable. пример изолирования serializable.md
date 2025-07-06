[[postgreSQL]]

Пример с дежурством врачей.

Допустим, у нас есть таблица, содержащая график дежурств врачей, и мы должны гарантировать, что в каждую смену дежурит хотя бы один врач.

Предположим, два врача (ID 1 и 2) одновременно хотят снять себя с дежурства. Мы открываем **два сеанса** (`Session 1` и `Session 2`), где они действуют независимо, но из-за уровня Repeatable Read видят устаревшее состояние данных.

```sql
CREATE TABLE duty_schedule (
    doctor_id INT PRIMARY KEY,
    on_duty BOOLEAN
);

INSERT INTO duty_schedule (doctor_id, on_duty) VALUES (1, TRUE), (2, TRUE);

```

**Session 1 (врач 1)**:

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Проверяем, есть ли хотя бы один врач на смене
SELECT COUNT(*) FROM duty_schedule WHERE on_duty = TRUE;
-- (Результат: 2, так как врач 2 тоже еще на смене)

-- Врач 1 снимает себя с дежурства
UPDATE duty_schedule SET on_duty = FALSE WHERE doctor_id = 1;

-- Пока не коммитим

```

**Session 2 (врач 2, параллельно)**:

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Проверяем, есть ли хотя бы один врач на смене
SELECT COUNT(*) FROM duty_schedule WHERE on_duty = TRUE;
-- (Результат: 2, так как врач 1 тоже еще на смене)

-- Врач 2 снимает себя с дежурства
UPDATE duty_schedule SET on_duty = FALSE WHERE doctor_id = 2;

-- Пока не коммитим

```

Теперь оба врача коммитят свои транзакции:  
**Session 1**:

```sql
COMMIT;

```

**Session 2**:

```sql
COMMIT;

```

В результате в таблице нет дежурных врачей. Нарушение бизнес-логики. При этом при использовании уровня `Serializable` такого бы не произошло. При коммите Session 2 упала бы с ошибкой сериализации.