Модули и сущности

auth:

- User (пользователь)
  - id number — уникальный идентификатор
  - email string — для логина
  - password string — хранится в зашифрованном виде

- RefreshToken (пропуск для каждого устройства)
  - id number — уникальный идентификатор
  - token string — токен сгенерированный библиотекой JWT
  - userId number — кому принадлежит
  - expiresAt TIMESTAMP — когда истекает

notes:

- Note (заметка)
  - id number — уникальный идентификатор
  - userId number — кому принадлежит
  - text string — текст заметки
  - createdAt TIMESTAMP — дата и время создания

evening-question:

- EveningQuestionConfig (настройка вопроса)
  - id number — уникальный идентификатор
  - userId number — кому принадлежит
  - questionText string — текст вопроса
  - notificationTime TIME — время уведомления
  - checklistTitle string — название задачи для чек-листа
  - isAnswered boolean — состояние ответила/нет

- EveningAnswer (ответ)
  - id number — уникальный идентификатор
  - userId number — кому принадлежит
  - answerText string — текст ответа
  - createdAt TIMESTAMP — дата и время ответа

morning-reminder:

- MorningReminderConfig (настройка уведомления)
  - id number — уникальный идентификатор
  - userId number — кому принадлежит
  - reminderText string — текст напоминания
  - notificationTime TIME — время уведомления
  - checklistTitle string — название задачи для чек-листа
  - isRead boolean — состояние прочитано/не прочитано

checklist:

- Task (задача)
  - id number — уникальный идентификатор
  - userId number — кому принадлежит
  - title string — название задачи
  - schedule string[] — расписание (дни недели)
  - repeatType string — тип (разовая/повторяющаяся)
  - isEditable boolean — можно ли редактировать
  - isActive boolean — активна ли задача (для разовых: после показа становится false)

- DailyTask (задача на конкретный день)
  - id number — уникальный идентификатор
  - taskId number — ссылка на Task
  - date DATE — дата
  - isDone boolean — состояние выполнена/нет

export:

- своих сущностей нет, читает Note и EveningAnswer

---

Архитектурные решения

- Состояния модулей (прочитано/отвечено) не привязаны к дню — это текущее состояние, сбрасывается при наступлении нового дня. Чек-лист просто читает эти состояния. Исключает ошибку когда действие относится к "вчерашнему" дню.
- Периодическая очистка старых данных (DailyTask + неактивные разовые Task). Каждый день создаются новые DailyTask, старые не нужны (статистика выполнения не собирается), без очистки таблица растёт.
- При регистрации пользователя создаются дефолтные настройки (EveningQuestionConfig, MorningReminderConfig, автоматические Task). Приложение работает сразу после регистрации без ручной настройки, пользователь меняет только если хочет.
- Refresh token обновляется при каждом использовании — чтобы не выкидывало пока пользуешься.

---

API-эндпоинты

auth:
POST /auth/register + { email, password } — зарегистрироваться
POST /auth/login + { email, password } — залогиниться, возвращает access + refresh токены
POST /auth/logout — выйти, удаляет refresh token из базы
POST /auth/refresh + { refreshToken } — обновить токены
DELETE /auth/:userId — удалить аккаунт

notes:
POST /notes + { text } — создать заметку
GET /notes?date=2026-03-28 — заметки за конкретную дату
GET /notes?from=2026-03-01&to=2026-03-28 — заметки за период
PATCH /notes/:id + { text } — изменить заметку
DELETE /notes/:id — удалить заметку

evening-question:
GET /evening-question/config — получить настройки
POST /evening-question/config — создать настройку
PATCH /evening-question/config — изменить настройку
PATCH /evening-question/config + { isAnswered: true } — отметить выполнение
GET /evening-question/answers — получить ответы
POST /evening-question/answers — сохранить ответ
PATCH /evening-question/answers/:id — редактировать ответ
DELETE /evening-question/answers/:id — удалить ответ

morning-reminder:
GET /morning-reminder/config — получить настройки
POST /morning-reminder/config — создать настройку
PATCH /morning-reminder/config — изменить настройку
PATCH /morning-reminder/config + { isRead: true } — отметить прочтение

checklist:
GET /checklist/tasks — получить все задачи
POST /checklist/tasks — создать задачу
PATCH /checklist/tasks/:id — изменить задачу
DELETE /checklist/tasks/:id — удалить задачу
GET /checklist/daily-tasks — получить задачи на сегодня
PATCH /checklist/daily-tasks/:id — поставить/снять галочку

export:
GET /export?from=2026-03-01&to=2026-03-28 — скачать файл за период

---

Тех.стек

- Node.js
- NestJS
- PostgreSQL
- TypeORM
