# ИДЗ №4 
###  Губайдуллин Роберт
#### Желаемая оценка: 10/10.
## Запуск

```bash
docker compose up --build
```
Сервисы будут доступны по следующим адресам:
1) AuthService - http://localhost:80.
2) TicketService - http://localhost:81.


## Описание
Проект состоит из 2 микросервисов `AuthService` и `TicketService` и базы данных.

`AuthService` - авторизация и регистрация пользователей.  
`TicketService` - создание заказов на покупку билетов.  
`PostgreSQL` - база данных.
###

## Сервисы
### AuthService

1) Регистрация нового пользователя: `POST /auth/register`. При регистрации пользователь указывает свой nickname, email и пароль. Пароль `хэшируется` перед сохранением в базу данных. После успешной регистрации пользователю выдается `JWT токен`.
2) Аутентификация пользователя: `POST /auth/login`. Пользователь указывает свой email и пароль. Если пользователь с таким email и паролем существует, ему выдается новый `JWT токен`.
3) Получение информации о текущем пользователе: `GET /auth/info`. Для выполнения запроса необходимо передать `JWT токен` в `заголовке Authorization` или в `cookie` с именем `token`. Если токен валиден, в ответе возвращается информация о пользователе.

### TicketService

1) Создание заказа: `POST /order/create`. Пользователь указывает станцию отправления, станцию прибытия и количество билетов. После создания заказа пользователю выдается вся информация о созданном заказе.
2) Получение информации о заказе: `GET /order/{orderId}`. Пользователь указывает идентификатор заказа. Если заказ с таким идентификатором существует, в ответе возвращается вся информация о заказе.
3) Получение всех заказов пользователя: `GET /order/all`.
4) Получение списка всех станций: `GET /station/all`. В ответе возвращается список всех станций.

Во всех ручках сервиса `TicketService` необходимо передавать `JWT токен` в `заголовке Authorization` или в `cookie` с именем `token`.

Статус заказа может быть одним из следующих: `Checked`, `Success`, `Rejected`. За изменение статуса заказа отвечает `ProcessOrderService`.
Он обновляет статусы недавно созданных заказов (`Checked`) каждые 10 секунд и обновляет их статусы на `Success` или `Rejected` в зависимости от случайного значения.
## Компоненты сервиса

### Model
Модели представляют собой классы, которые используются для описания структуры данных, хранящихся в базе данных. Они включают в себя поля и методы, необходимые для работы с этими данными. Модели служат основой для создания таблиц в базе данных.

### Repository
Репозитории предоставляют абстракцию для работы с базой данных, обеспечивая методы для выполнения CRUD (создание, чтение, обновление, удаление) операций. Они взаимодействуют с моделями и позволяют сервисам получать доступ к данным, хранящимся в базе данных.

### Service
Сервисы содержат бизнес-логику приложения. Они используют репозитории для выполнения операций с базой данных и обеспечивают выполнение всех необходимых операций с данными. Сервисы также могут включать в себя дополнительную логику, такую как валидация данных, отправка уведомлений и т.д.

### Util
Утилиты представляют собой вспомогательные классы и функции, которые используются в различных частях приложения для выполнения общих задач, таких как хэширование паролей, генерация токенов, форматирование данных и т.д.

### Controller
Контроллеры обрабатывают HTTP-запросы, полученные от клиентов. Они вызывают соответствующие методы сервисов для выполнения операций и возвращают результаты в виде HTTP-ответов. Контроллеры также отвечают за обработку ошибок и возврат соответствующих сообщений об ошибках клиентам.

### Exception
Исключения используются для обработки ошибок, возникающих во время выполнения приложения. Они позволяют централизованно управлять ошибками и обеспечивать корректную обработку и возврат сообщений об ошибках клиентам.

### DTO
Объекты передачи данных используются для передачи данных между различными слоями приложения, такими как контроллеры, сервисы и репозитории. DTO обеспечивают удобный способ упаковки данных и помогают предотвратить передачу лишней информации между слоями.

### Дополнительные улучшения сверх требований
1) Все 3 котейнера покрыты healthcheck'ами. Они позволяют сделать так, чтобы сервисы не начинали обрабатывать запросы, пока не будут готовы к этому.
2) Были настроены `cookie` для передачи `JWT токена`. Это позволяет избежать постоянной вставки параметра `JWT` в каждый эндпоинт.
3) Был добавлен скрипт `migrations.sql` для автоматических миграций при запуске. Так же скрипт заполняет таблицу `station` тестовыми данными.
