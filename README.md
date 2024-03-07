## Описание проекта
restaurant_rest_api - это REST-API приложение без фронтенда, написанное на языке Java с использованием фреймворка SpringBoot и хранением данных в динамической памяти (репозитории - это ArrayList-ы, в которые занесены данные для тестов)

## Реализованные требования
- Аутентификация пользователей (user и admin) происходит по логинам (см. UserRepository и AdminRepository). Для разных типов пользователей реализованы свои контроллеры. В каждом запросе, кроме получения меню, необходимо указывать login пользователя в параметрах запроса.
### - Администратор
#### Эндпоинты (AdminController)
Во всех запросах, если есть параметр {login}, требуется существующий login админа (см. AdminRepository). Иначе выведется сообщение о том, что аутентификация не прошла.
- **GET /admin/menu** : возвращает для админа список доступных блюд.
- **POST /menu/update{dishId}{dishDurationInMin}{dishPrice}{login}**: по id блюда происходит изменение двух переданных параметров блюда (оба не являются обязательными).
- **POST /menu/delete{dishId}{login}**: удаляет блюда по id из репозитория. Если блюда нет, отправляется ответ о невозможности выполнить это действие.
- **POST /menu/add{login}**: добавляет новое блюдо. Если такое же блюдо (совпадают все поля) было добавлено ранее, отправляется ответ о невозможности выполнить это действие.

### - Пользователь
#### Эндпоинты (UserController)
Во всех запросах, если есть параметр {login}, требуется существующий login юзера (см. AdminRepository). Иначе отправляется ответ о том, что аутентификация не прошла)
- **POST /order/create{login}**: создает новый заказ (переданный в теле запроса) у данного (с указанным login-ом) пользователя. Каждому заказу (в т.ч. некорректному, который не будет добавлен) автоматически присваивается id. Поэтому валидные заказы не обязательно имеют подряд идущие id. Созданный заказ обрабатывается в многопоточном режиме.
- **GET /order/list{login}**: выводит список всех заказов у пользователя с указанным login-ом.
- **POST /order/update{orderId}{login}** : добавляет переданное в теле запроса блюдо в заказ по id заказа. Заказ должен иметь статус PREPARING. Если блюда нет в меню или статус заказа иной, отправляется ответ о невозможности выполнить это действие.
- **POST /order/cancel{orderId}{login}** : отменяет заказ со статусом PREPARING по id, т.е. статус заказа устанавливается в CANCELED, но заказ не удаляется из списка. Если такого заказа нет или статус заказа иной, отправляется ответ о невозможности выполнить это действие.
- **GET /order/status{orderId}{login}** : возвращает статус заказа по его id. Если такого заказа нет, отправляется ответ о невозможности выполнить это действие.
- **POST /order/pay{orderId}{login}** : устанавливает статус заказа в PAYED (делает заказ оплаченным). Для выполнения этого действия статус переданного заказа должен быть READY. Если такого заказа нет или статус заказа иной, отправляется ответ о невозможности выполнить это действие.
- **GET /menu**: возвращает список доступных для заказа блюд для юзера.

### Система приоритетов для обработки заказов
Заказы обрабатываются в порядке добавления (чем раньше добавлен, тем раньше обработается).

### Самое главное! - как тестировать
В корне проекта находятся 2 файла с http-запросами: adminRequests.http и userRequests.http для тестирования эндпоинтов AdminController-а и UserController-а соответственно.
<br/>Просьба - выполнять запросы по порядку, потому что результат выполнения последующих запросов может зависеть от предыдущих (например, тестируется удаление ранее добавленного блюда в меню). И сначала нужно запустить adminRequests, а затем UserRequests (если в одном запуске тестируются оба файла). Либо нужно менять параметры запросов.
<br/>Также из-за многопоточной обработки результат одного и того же запроса может отличаться от теста к тесту (в частности - отмена заказа может не сработать. Тогда можно несколько раз подряд попытаться выполнить запрос на отмену заказа. Или заново начать тестировать все эндпоинты админа).
<br/>При тестировании манипуляций с заказами просьба обращать внимание на их id (так как они не обязательно будут идти подряд без пропусков). При добавлении заказа лучше вывести о нем информацию (order/list), чтобы посмотреть его id. 