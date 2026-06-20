# Лабораторная работа: Middleware и расширенная логика Django

Проект: **Корпоративный портал заявок**.

## Что реализовано по ТЗ

1. **Модели**:
   - `Ticket` — заявка с файлом, статусом, приоритетом, сервисом, владельцем и ответственным.
   - `Comment` — комментарии к заявкам, включая внутренние комментарии модераторов.
   - `ChangeHistory` — история изменений заявки.
   - `UserProfile` — роль пользователя: обычный пользователь, модератор, администратор.
   - `UserSettings` — пользовательские настройки.
   - `AuditLog` — журнал аудита запросов.

2. **Интерфейс**:
   - список заявок с фильтрами по статусу, сервису, приоритету и поиском;
   - карточка заявки;
   - создание заявки;
   - редактирование заявки;
   - панель модератора;
   - страница входа.

3. **REST API**:
   - `/api/tickets/`
   - `/api/comments/`
   - `/api/settings/`
   - `/api/audit-logs/` — доступно только администратору.

4. **Права доступа**:
   - обычный пользователь видит свои заявки и может редактировать только свои новые/отклоненные заявки, измененные за последние 24 часа;
   - модератор видит и обрабатывает все заявки, кроме закрытых;
   - администратор имеет полный доступ;
   - внутренние комментарии видны только модератору и администратору.

5. **Middleware**:
   - `RequestIdMiddleware` — создает `request_id`, добавляет его в ответ `X-Request-ID`, пробрасывает в логи;
   - `AuditMiddleware` — пишет IP, user agent, метод, путь, статус ответа и результат в `AuditLog`;
   - `RateLimitMiddleware` — ограничивает частоту запросов для `/api/tickets/` и `/tickets/create/`;
   - `AutoUserMiddleware` — автоматически определяет пользователя, создает профиль и настройки, а в DEBUG поддерживает вход через заголовок `X-Auto-User`;
   - `WorkingHoursMiddleware` — запрещает POST/PUT/PATCH/DELETE вне рабочего времени для не-администраторов.

## Как запустить

```bash
cd corporate_portal_requests
python -m venv venv
```

Windows PowerShell:

```powershell
venv\Scripts\activate
pip install -r requirements.txt
python manage.py migrate
python manage.py create_demo_data
python manage.py runserver
```

Linux/macOS:

```bash
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py create_demo_data
python manage.py runserver
```

Открыть сайт: `http://127.0.0.1:8000/`

## Демо-пользователи

После команды `python manage.py create_demo_data` создаются:

| Роль | Логин | Пароль |
|---|---|---|
| Администратор | `admin` | `admin12345` |
| Модератор | `moderator` | `moderator12345` |
| Обычный пользователь | `user` | `user12345` |

## Где проверять middleware

- Request ID: открыть любую страницу и посмотреть заголовок ответа `X-Request-ID`.
- Audit log: зайти в админку или `/api/audit-logs/` под администратором.
- Rate limit: часто обновлять `/api/tickets/`.
- Auto user: в DEBUG можно отправить заголовок `X-Auto-User: testuser`.
- Working hours: изменить `PORTAL_WORK_START_HOUR` и `PORTAL_WORK_END_HOUR` в `settings.py`, затем попробовать создать/изменить заявку обычным пользователем вне разрешенного времени.
