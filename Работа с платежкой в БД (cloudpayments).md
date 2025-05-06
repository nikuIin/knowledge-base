[[Платежка]]
[[postgreSQL]]


# Ответ grok

Для реализации ежемесячной подписки с использованием CloudPayments, основываясь на их документации ([developers.cloudpayments.ru](https://developers.cloudpayments.ru)), необходимо хранить в базе данных минимальный набор данных, обеспечивающий безопасность, функциональность автоплатежей и соответствие стандартам PCI DSS. CloudPayments активно использует **токенизацию** для рекуррентных платежей, что позволяет не хранить чувствительные данные карт, а также предоставляет API для управления подписками. Ниже описан оптимальный набор данных для хранения в базе данных, рекомендации по их использованию и интеграция с вашей существующей структурой БД для тарифов и премиум-скинов.

---

### Обзор подхода CloudPayments к рекуррентным платежам
CloudPayments поддерживает **рекуррентные платежи** (автоплатежи для подписок) через токенизацию карт и управление подписками через API. Основные моменты из документации:
- **Токенизация**: После первой успешной оплаты картой CloudPayments возвращает токен (`Token`), который используется для последующих автоплатежей без необходимости повторного ввода данных карты ([Оплата по токену](https://developers.cloudpayments.ru/#oplata-po-tokenu-rekarring)).
- **Создание подписки**: Метод `CreateSubscription` позволяет создать подписку с указанием периода (например, ежемесячная) и суммы ([Создание подписки](https://developers.cloudpayments.ru/#sozdanie-podpiski-na-rekurrentnye-platezhi)).
- **Управление подписками**: Методы `UpdateSubscription`, `CancelSubscription`, `GetSubscription` и `GetSubscriptionsList` позволяют изменять, отменять или проверять статус подписок.
- **Отмена подписки**: Пользователь может отменить подписку через сайт [my.cloudpayments.ru](https://my.cloudpayments.ru) или через API ([Отмена подписки](https://developers.cloudpayments.ru/#izmenenie-podpiski-na-rekurrentnye-platezhi)).
- **Безопасность**: CloudPayments сертифицирован по PCI DSS Level 1, использует SSL/TLS и RSA-шифрование (2048 бит), а также 3-D Secure для защиты транзакций ([developers.cloudpayments.ru](https://developers.cloudpayments.ru)).

Для вашей системы с тарифами (`plans`) и премиум-скинами (`premium_skins`), данные в базе должны поддерживать интеграцию с CloudPayments, обеспечивать автоплатежи и отслеживать статус подписок.

---

### Данные для хранения в базе данных
Ниже приведен список данных, которые необходимо хранить в базе данных для поддержки ежемесячной подписки с CloudPayments, с учетом вашей текущей структуры (`users`, `plans`, `subscriptions`, `premium_skins`, `skin_purchases`, `payments`).

#### 1. Таблица `payment_tokens` (хранение токенов карт)
Для автоплатежей необходимо хранить токены, возвращаемые CloudPayments после успешной оплаты. Это позволяет выполнять рекуррентные платежи без хранения данных карты.

```sql
CREATE TABLE payment_tokens (
    token_id SERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(user_id),
    token VARCHAR(255) NOT NULL, -- Токен от CloudPayments (например, success_1111a3e0-2428-48fb-a530-12815d90d0e8)
    payment_method VARCHAR(50) NOT NULL, -- mir_card, visa, mastercard
    last_four_digits VARCHAR(4), -- Последние 4 цифры карты (например, 4242)
    card_type VARCHAR(50), -- Тип карты (Visa, MasterCard, Мир)
    expiry_date CHAR(7), -- Срок действия карты (MM/YYYY, например, 12/2025)
    is_active BOOLEAN DEFAULT TRUE, -- Активность токена
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_payment_tokens_user_id ON payment_tokens(user_id);
```

**Что хранить**:
- `token`: Уникальный токен от CloudPayments, возвращаемый в ответе API (например, в поле `Token` при оплате по криптограмме или токену).
- `last_four_digits`: Поле `CardLastFour` из ответа API (например, `4242`), для отображения в интерфейсе (например, «**** 4242»).
- `card_type`: Поле `CardType` из ответа (например, `Visa`, `MasterCard`, `Mir`).
- `expiry_date`: Поле `CardExpDate` (например, `12/25`), для проверки срока действия.
- `payment_method`: Указывает тип карты, соответствующий вашей системе (например, `mir_card`).

**Источник**: [Пример ответа API](https://developers.cloudpayments.ru/#oplata-po-tokenu-rekarring), [developers.cloudpayments.ru](https://developers.cloudpayments.ru).

#### 2. Таблица `subscriptions` (расширение для CloudPayments)
Ваша текущая таблица `subscriptions` уже содержит необходимые поля, но нужно добавить поле для хранения идентификатора подписки CloudPayments (`SubscriptionId`).

```sql
ALTER TABLE subscriptions
ADD COLUMN cloudpayments_subscription_id VARCHAR(50), -- ID подписки в CloudPayments
ADD COLUMN cloudpayments_token_id INT REFERENCES payment_tokens(token_id); -- Связь с токеном
```

**Что хранить**:
- `cloudpayments_subscription_id`: Уникальный идентификатор подписки, возвращаемый методом `CreateSubscription` (например, `success_1111a3e0-2428-48fb-a530-12815d90d0e8`).
- `cloudpayments_token_id`: Связь с токеном карты, используемым для подписки.
- Остальные поля (`user_id`, `plan_id`, `start_date`, `end_date`, `status`) остаются без изменений и используются для локального учета.

**Почему это нужно**:
- `cloudpayments_subscription_id` позволяет управлять подпиской через API (изменение, отмена, проверка статуса).
- `cloudpayments_token_id` связывает подписку с токеном для автоплатежей.

**Источник**: [Создание подписки](https://developers.cloudpayments.ru/#sozdanie-podpiski-na-rekurrentnye-platezhi).

#### 3. Таблица `payments` (расширение для CloudPayments)
Таблица `payments` должна хранить информацию о транзакциях, включая данные от CloudPayments для отслеживания и сверки.

```sql
ALTER TABLE payments
ADD COLUMN cloudpayments_transaction_id BIGINT, -- TransactionId от CloudPayments
ADD COLUMN cloudpayments_status VARCHAR(20), -- Completed, Declined, Authorized
ADD COLUMN cloudpayments_reason_code INT, -- Код ошибки (если транзакция отклонена)
ADD COLUMN cloudpayments_reason VARCHAR(255); -- Описание ошибки
```

**Что хранить**:
- `cloudpayments_transaction_id`: Уникальный ID транзакции из ответа API (поле `TransactionId`, например, `897728064`).
- `cloudpayments_status`: Статус транзакции (поле `Status`, например, `Completed`, `Declined`, `Authorized`).
- `cloudpayments_reason_code`: Код причины отказа (поле `ReasonCode`, например, `0` для успешной транзакции).
- `cloudpayments_reason`: Описание причины отказа (поле `Reason`, например, `Approved` или `Insufficient funds`).

**Почему это нужно**:
- Эти данные позволяют отслеживать историю платежей, диагностировать ошибки и предоставлять пользователям информацию о статусе транзакций.
- `cloudpayments_transaction_id` используется для сверки с CloudPayments и обработки возвратов.

**Источник**: [Пример ответа API](https://developers.cloudpayments.ru/#oplata-po-tokenu-rekurring), [Проверка статуса платежа](https://developers.cloudpayments.ru/#proverka-statusa-platezha).

#### 4. Таблица `users` (дополнительные данные плательщика)
Для соответствия требованиям НСПК (для телекоммуникационных услуг) и удобства интеграции с CloudPayments, рекомендуется хранить данные плательщика.

```sql
ALTER TABLE users
ADD COLUMN phone VARCHAR(20), -- Номер телефона плательщика
ADD COLUMN first_name VARCHAR(100), -- Имя
ADD COLUMN last_name VARCHAR(100), -- Фамилия
ADD COLUMN middle_name VARCHAR(100), -- Отчество
ADD COLUMN birth_date DATE, -- Дата рождения
ADD COLUMN address TEXT, -- Адрес
ADD COLUMN city VARCHAR(100), -- Город
ADD COLUMN country VARCHAR(50), -- Страна
ADD COLUMN postcode VARCHAR(20); -- Почтовый индекс
```

**Что хранить**:
- `phone`: Обязателен для телекоммуникационных услуг (MCC 4814), передается в объекте `Payer.Phone` в запросах API.
- `first_name`, `last_name`, `middle_name`, `birth_date`, `address`, `city`, `country`, `postcode`: Используются в объекте `Payer` для повышения безопасности и идентификации плательщика.

**Почему это нужно**:
- НСПК требует передачи номера телефона для некоторых транзакций, иначе банки-эквайеры могут отклонить платеж.
- Данные плательщика улучшают защиту от мошенничества и могут использоваться для персонализации.

**Источник**: [Требования НСПК](https://developers.cloudpayments.ru/#oplata-po-kriptogramme).

---

### Минимальный набор данных для ежемесячной подписки
Для реализации ежемесячной подписки с CloudPayments достаточно хранить:
1. **Токен карты** (`payment_tokens`):
   - `token`, `last_four_digits`, `card_type`, `expiry_date`, `payment_method`.
2. **Идентификатор подписки** (`subscriptions`):
   - `cloudpayments_subscription_id`, `cloudpayments_token_id`.
3. **Данные транзакции** (`payments`):
   - `cloudpayments_transaction_id`, `cloudpayments_status`, `cloudpayments_reason_code`, `cloudpayments_reason`.
4. **Данные плательщика** (`users`):
   - `phone` (обязательно), остальные поля опциональны.

**Примечание**: Полные данные карты (номер, CVV) **не хранятся**, так как CloudPayments использует токенизацию. Это соответствует PCI DSS и снижает риски.

---

### Интеграция с CloudPayments
#### 1. Первая оплата и создание подписки
1. Пользователь выбирает тариф (`plans`) и вводит данные карты в виджете CloudPayments.
   - Используйте метод `charge` или `auth` для оплаты ([Оплата по криптограмме](https://developers.cloudpayments.ru/#oplata-po-kriptogramme)).
   - Пример запроса:
     ```json
     {
       "Amount": 500,
       "Currency": "RUB",
       "InvoiceId": "sub_123",
       "Description": "Подписка на тариф Премиум",
       "AccountId": "user_123",
       "CardCryptogramPacket": "01492500008719030128SMfLeYdKp5dSQVIiO5l6ZCJiPdel4uDjdFTTz1UnXY+3QaZcNOW8lmXg0H670MclS4lI+qLkujKF4pR5Ri+T/E04Ufq3t5ntMUVLuZ998DLm+OVHV7FxIGR7snckpg47A73v7/y88Q5dxxvVZtDVi0qCcJAiZrgKLyLCqypnMfhjsgCEPF6d4OMzkgNQiynZvKysI2q+xc9cL0+CMmQTUPytnxX52k9qLNZ55cnE8kuLvqSK+TOG7Fz03moGcVvbb9XTg1oTDL4pl9rgkG3XvvTJOwol3JDxL1i6x+VpaRxpLJg0Zd9/9xRJOBMGmwAxo8/xyvGuAj85sxLJL6fA==",
       "Payer": {
         "FirstName": "Иван",
         "LastName": "Иванов",
         "Phone": "+71234567890"
       }
     }
     ```
2. CloudPayments возвращает `Token`, `CardLastFour`, `CardType`, `CardExpDate`. Сохраните их в `payment_tokens`.
3. Создайте подписку через метод `CreateSubscription`:
   - Пример запроса:
     ```json
     {
       "Token": "success_1111a3e0-2428-48fb-a530-12815d90d0e8",
       "AccountId": "user_123",
       "Description": "Ежемесячная подписка на тариф Премиум",
       "Email": "user@example.com",
       "Amount": 500,
       "Currency": "RUB",
       "RequireConfirmation": false,
       "StartDate": "2025-04-25T00:00:00",
       "Interval": "Month",
       "Period": 1
     }
     ```
   - Ответ содержит `SubscriptionId`, который сохраняется в `subscriptions.cloudpayments_subscription_id`.

#### 2. Автоплатежи
- CloudPayments автоматически списывает средства в соответствии с интервалом подписки (`Interval: Month`, `Period: 1`).
- После каждого списания проверяйте статус транзакции через метод `findPaymentByInvoiceId` или `getPayment` ([Проверка статуса платежа](https://developers.cloudpayments.ru/#proverka-statusa-platezha)).
- Обновляйте `payments` и `subscriptions.end_date` после успешного платежа:
  ```sql
  INSERT INTO payments (user_id, amount, payment_type, reference_id, payment_method, status, cloudpayments_transaction_id, cloudpayments_status)
  VALUES (123, 500.00, 'subscription', 1, 'mir_card', 'completed', 897728064, 'Completed');

  UPDATE subscriptions
  SET end_date = end_date + INTERVAL '30 days',
      status = 'active'
  WHERE subscription_id = 1;
  ```

#### 3. Отмена подписки
- Пользователь может отменить подписку через [my.cloudpayments.ru](https://my.cloudpayments.ru) или через ваш интерфейс, вызвав метод `CancelSubscription`:
  ```json
  {
    "Id": "success_1111a3e0-2428-48fb-a530-12815d90d0e8"
  }
  ```
- Обновите `subscriptions.status` на `cancelled`:
  ```sql
  UPDATE subscriptions
  SET status = 'cancelled'
  WHERE cloudpayments_subscription_id = 'success_1111a3e0-2428-48fb-a530-12815d90d0e8';
  ```

#### 4. Обработка ошибок
- Если транзакция отклонена (например, недостаточно средств), CloudPayments возвращает `ReasonCode` и `Reason`. Сохраните их в `payments.cloudpayments_reason_code` и `payments.cloudpayments_reason`.
- Уведомите пользователя и предоставьте льготный период (например, 7 дней), обновив `subscriptions.status` на `pending`.

---

### Рекомендации по реализации
1. **Безопасность**:
   - Не храните данные карт (номер, CVV) в базе данных. Используйте токены CloudPayments.
   - Шифруйте поле `payment_tokens.token` с помощью AES-256, если требуется дополнительная защита.
   - Обеспечьте TLS 1.2+ для всех API-запросов.

2. **Интерфейс**:
   - Отображайте пользователю последние 4 цифры карты (`last_four_digits`) и тип карты (`card_type`) для идентификации.
   - Предоставьте кнопку «Отписаться» с вызовом `CancelSubscription` и ссылку на [my.cloudpayments.ru](https://my.cloudpayments.ru).

3. **Соответствие требованиям**:
   - Передавайте номер телефона в объекте `Payer.Phone` для телекоммуникационных услуг (MCC 4814).
   - Соблюдайте ФЗ-152 «О персональных данных», храня данные пользователей на серверах в РФ или с согласия на трансграничную передачу.

4. **Тестирование**:
   - Используйте тестовые карты CloudPayments для проверки интеграции ([Тестовые данные](https://developers.cloudpayments.ru/#widget)).
   - Настройте тестовый режим (`TestMode: true`) в запросах API.

5. **Мониторинг**:
   - Регулярно проверяйте статус подписок через `GetSubscriptionsList` для синхронизации с CloudPayments.
   - Настройте вебхуки для обработки уведомлений о платежах ([Обработка уведомлений](https://developers.cloudpayments.ru/#obrabotka-uvedomleniy)).

---

### Итоговый список данных для хранения
Для ежемесячной подписки с CloudPayments храните:
- **В `payment_tokens`**:
  - `user_id`, `token`, `payment_method`, `last_four_digits`, `card_type`, `expiry_date`, `is_active`.
- **В `subscriptions`**:
  - Добавить `cloudpayments_subscription_id`, `cloudpayments_token_id`.
- **В `payments`**:
  - Добавить `cloudpayments_transaction_id`, `cloudpayments_status`, `cloudpayments_reason_code`, `cloudpayments_reason`.
- **В `users`**:
  - Добавить `phone` (обязательно), опционально `first_name`, `last_name`, `middle_name`, `birth_date`, `address`, `city`, `country`, `postcode`.

**Примечание**: Эти изменения минимально расширяют вашу текущую структуру БД, обеспечивая полную интеграцию с CloudPayments для ежемесячных подписок.

**Источник**: Документация CloudPayments ([developers.cloudpayments.ru](https://developers.cloudpayments.ru)), включая разделы об оплате по токену, создании подписок и обработке транзакций.