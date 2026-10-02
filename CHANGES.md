# CHANGES — что изменено в v3 (текст и диаграммы)

Сергей, в `…_v3.docx` правки точечные: неверные имена, числа, роли, маршруты и методы заменены на настоящие, нереализованное (база знаний, FAQ, поиск, оповещения, вложения, экраны заявителя и нотариуса) помечено «(проектируется)». Таблицы 1–7 и скриншоты не менялись. Ссылки — на файлы eaafit/WebDevelopment (main `92f2aac`).

Короткие имена файлов:

- support.proto — `libs/shared/api-contracts/proto/notary/support/v1alpha1/support.proto`
- support.service.ts, support-rpc.service.ts, support.module.ts — `libs/api/support/src/lib/`
- connect-router.registry.ts, main.ts — `apps/api/src/app/, apps/api/src/`
- schema.prisma — `libs/api/shared/prisma/schema.prisma`
- admin-support.component.*, create-ticket-modal.component.ts, ticket-queue.component.*, operator-chat.component.*, support-api.service.ts, support.types.ts — `libs/web/admin/src/lib/features/support/…`
- admin.routes.ts, applicant.routes.ts, notary.routes.ts, app.routes.ts — `libs/web/{admin,applicant,notary}/src/lib/, apps/web/src/app/`

## Текст: было → стало

### Введение

- «Данный компонент полностью переносит» → «Данный компонент переносит»; «в единый контур веб-приложения Notary.» → «в единый контур веб-приложения Notary (интерфейс пользователей — проектируется).» — реализовано рабочее место оператора /admin/support, экраны поддержки заявителя и нотариуса — заглушки (admin.routes.ts:150–153, applicant.routes.ts:115–117, notary.routes.ts:67–69). <!-- v2:75 -->
- «Со стороны заявителя компонент» → «Со стороны заявителя (интерфейс — проектируется) компонент»; «в базе знаний посредством» → «в базе знаний (проектируется) посредством»; « с последующим ведением текстового диалога в реальном времени.» → « с последующим ведением текстового диалога.» — экраны заявителя /applicant/support и /applicant/faq — заглушки (серверная часть для заявителя есть: CreateTicket, AddMessage); базы знаний и поиска нет и на сервере — в контракте SupportService нет RPC по статьям, модели Article в схеме нет; «реального времени» нет — веб-сокетов нет, экран оператора опрашивает сервер раз в 10 с (applicant.routes.ts:115–121, support.service.ts:121–226, support.proto:155–165, schema.prisma:771–809, admin-support.component.ts:50–52). <!-- v2:76 -->
- «для расширения базы знаний авторами контента.» → «для расширения базы знаний (проектируется).» — статей и их редактирования в коде нет; роли «автор контента» нет — роли Applicant, Notary, Admin (schema.prisma:18–24). <!-- v2:77 -->
- «интерфейсы заявителя и оператора» → «интерфейсы заявителя (проектируется) и оператора»; «и модуль оповещений.» → «и модуль оповещений (проектируется).» — реальный интерфейс — только /admin/support; SupportService уведомлений не создаёт (модуль импортирует только PrismaModule) (admin.routes.ts:150–153, applicant.routes.ts:115–121, support.module.ts:7). <!-- v2:78 -->
- «ведения базы знаний и поиска со встроенным механизмом контроля SLA и нотификаций.» → «ведения базы знаний и поиска (проектируется) со встроенным механизмом контроля SLA и нотификаций (проектируется).» — статей, поиска и уведомлений по тикетам в коде нет (support.proto:155–165, support.module.ts:7). <!-- v2:87 -->

### 1.1. Диаграмма развертывания UML

- «Он принимает входящий HTTPS-трафик на портах 80/443 и перенаправляет» → «Он принимает входящий HTTP-трафик на порту 80 (HTTPS при необходимости обеспечивает Nginx Proxy Manager) и перенаправляет»; «, избавляя от необходимости дополнительной настройки CORS-политик.» → «.» — portal слушает только порт 80, TLS — у Nginx Proxy Manager; CORS в API всё равно настраивается переменной CORS_ORIGIN (apps/web/nginx/portal.conf:7, apps/web/DOCKER.md:36–46, 62, 75, apps/api/src/main.ts:43–67). <!-- v2:93 -->

### 1.2. Описание развертывания на виртуальном сервере

- «описывается в декларативном файле docker-compose.yaml.» → «описывается в декларативных файлах docker-compose.yaml и apps/web/docker-compose.portal.yml.»; «выступает единым хранилищем для всех системных модулей.» → «выступает единым реляционным хранилищем для всех системных модулей.»; «текстового содержимого статей базы знаний и лога системных уведомлений.» → «текстового содержимого статей базы знаний (проектируется) и лога системных уведомлений (для тикетов — проектируется).» — контейнеры portal, api и postgres портала описаны в apps/web/docker-compose.portal.yml; файлы хранит MinIO; для поддержки в схеме есть только tickets и ticket_messages, модели Article нет, Notification поддержка не пишет (apps/web/docker-compose.portal.yml:15–63, docker-compose.yaml:134–146, schema.prisma:771–809). <!-- v2:100 -->
- «а операторы не имеют прав на создание тикетов от чужого имени.» → «а создавать тикеты от чужого имени может только оператор (Admin).» — author_email принимается только от Admin, окно создания тикета на /admin/support передаёт email пользователя (support.service.ts:369–382, support-api.service.ts:97–108). <!-- v2:102 -->
- «JWT_SECRET» → «JWT_ACCESS_SECRET» — переменная называется JWT_ACCESS_SECRET (libs/api/auth/src/lib/auth/token.service.ts:16, .env.example:9). <!-- v2:105 -->
- Удалён пункт списка «SLA_DEFAULT_REPLY_MINUTES — регламентное время на первичный ответ оператора (по умолчанию 120 минут).» — такой переменной нет: срок SLA задан в коде по приоритету (Urgent 1 ч, High 4 ч, Medium 24 ч, Low 72 ч) (support.service.ts:420–429). <!-- v2:106 -->

### 2.1. Назначение и границы компонента

- «и предоставления справочной информации.» → «и предоставления справочной информации (проектируется).» — базы знаний в коде нет, /applicant/faq и /notary/faq — заглушки (applicant.routes.ts:119–121, notary.routes.ts:71–73). <!-- v2:148 -->
- «среди опубликованных статей базы знаний.» → «среди опубликованных статей базы знаний (проектируется).» — статей и поиска в коде нет (applicant.routes.ts:119–121). <!-- v2:150 -->
- «FAQ по категориям.» → «FAQ по категориям (проектируется).» — FAQ — заглушка (applicant.routes.ts:119–121). <!-- v2:151 -->
- «степени критичности инцидента.» → «степени критичности инцидента (экран заявителя — проектируется).» — CreateTicket доступен любой роли, но экран /applicant/support — заглушка (support.service.ts:121–161, applicant.routes.ts:115–117). <!-- v2:152 -->
- «отслеживая смену его статусов.» → «отслеживая смену его статусов (экран заявителя — проектируется).» — AddMessage доступен автору тикета, экран чата заявителя — заглушка (support.service.ts:210–219, applicant.routes.ts:115–117). <!-- v2:153 -->
- «(Support Operator)» → «(роль Admin)» — ролей SupportOperator нет: Applicant, Notary, Admin; /admin/support открыт только Admin (schema.prisma:18–24, app.routes.ts:22–26). <!-- v2:154 -->
- «ранжированных по приоритету и времени нарушения SLA.» → «с сортировкой по дате, приоритету и SLA.» — сервер отдаёт тикеты по дате создания, сортировка по приоритету и SLA — выбор в интерфейсе (support.service.ts:193, admin-support.component.html:61–67, ticket-queue.component.ts:43–53). <!-- v2:155 -->
- «(Support Author)» → «(роль Admin)» — роли SupportAuthor нет; ведение статей (проектируется) — Admin, как в таблице 5 (schema.prisma:18–24). <!-- v2:157 -->
- «Осуществляет администрирование базы знаний:» → «Осуществляет администрирование базы знаний (проектируется):» — статей в коде нет (support.proto:155–165). <!-- v2:158 -->

### 2.2. Модель данных и диаграмма классов

- «customerId» → «authorId»; «assignedToId» → «assigneeId»; «лог текстовых сообщений Message.» → «лог текстовых сообщений TicketMessage.» — поля и модель называются authorId, assigneeId, TicketMessage (schema.prisma:776–777, schema.prisma:795–809). <!-- v2:166 -->
- «Подсистема справки базируется» → «Подсистема справки (проектируется) базируется»; «порождает записи в таблице Notification.» → «порождает записи в таблице Notification (проектируется).» — модели Article в схеме нет; SupportService не создаёт Notification (schema.prisma:771–809, support.module.ts:7). <!-- v2:167 -->

### 2.3. Диаграмма компонентов и API-контракты

- «Подсистема базы знаний (отвечает за статический контент и поиск).» → «Подсистема базы знаний (проектируется; отвечает за статический контент и поиск).» — базы знаний и поиска в коде нет (support.proto:155–165). <!-- v2:173 -->
- «NestJS-контроллерами» → «RPC-сервисами NestJS» — обработчики — SupportRpcService, зарегистрированный в Connect-роутере, а не NestJS-контроллеры (connect-router.registry.ts:168–179, support-rpc.service.ts:24–54). <!-- v2:174 -->

### 2.4. Бизнес-правила и ограничения доступа

- «(slaBreachTime)» → «(slaDeadline)»; «Для приоритета Critical время ответа составляет 30 минут, для High — 2 часа, для Medium — 4 часа, для Low — 24 часа.» → «Для приоритета Urgent время ответа составляет 1 час, для High — 4 часа, для Medium — 24 часа, для Low — 72 часа.»; «Если оператор не оставляет сообщение до наступления этой метки, тикет помечается в системе как «SLA Violated» и поднимается в самый верх очереди административной панели.» → «После наступления этой метки тикет помечается в интерфейсе как «Нарушен» и поднимается в самый верх очереди административной панели при сортировке «Критичные по SLA».» — срок считает calculateSlaDeadline: Urgent 1 ч, High 4 ч, Medium 24 ч, Low 72 ч; статус «Нарушен» UI ставит только по сроку, наверх такие тикеты поднимает сортировка «Критичные по SLA» (support.service.ts:420–429, support.types.ts:35–40, ticket-queue.component.ts:47–49, 67–71, admin-support.component.html:65). <!-- v2:183 -->
- «по состояниям строго последовательно:» → «по состояниям:»; «Заявитель имеет право перевести свой тикет в статус Closed» → «Оператор (Admin) имеет право перевести тикет в статус Closed»; «на любом этапе, если проблема отпала.» → «на любом этапе (в интерфейсе — из статусов Open и InProgress), если проблема отпала.»; ««Взять в работу»» → ««Взять»» — сервер не проверяет порядок переходов, запрещено лишь менять закрытый тикет; UpdateTicketStatus и CloseTicket — только Admin; кнопка в очереди называется «Взять» (support.service.ts:296–352, ticket-queue.component.html:46–51, operator-chat.component.html:10–13). <!-- v2:184 -->

### 3.1. Перечень экранов и состояний

- «Пользовательский интерфейс модуля интегрирован в общую дизайн-систему портала Notary.» → «Интерфейс оператора поддержки встроен в оболочку административной панели портала Notary.»; «Разработаны следующие основные экраны:» → «Предусмотрены следующие основные экраны:» — страница поддержки оператора подключена маршрутом внутри оболочки Admin и использует собственные стили (цвета заданы в SCSS, токенов админ-панели нет); экраны заявителя — маршруты кабинета заявителя и пока заглушки, поэтому об оболочке админ-панели сказано только для интерфейса оператора; из перечисленных экранов реализован только /admin/support (admin.routes.ts:12–16, 150–153, admin-support.component.scss:1–30, applicant.routes.ts:12–16, 115–121). <!-- v2:212 -->
- «(/applicant/support/faq)» → «(/applicant/faq, проектируется)» — маршрут — /applicant/faq, и это заглушка (applicant.routes.ts:119–121). <!-- v2:213 -->
- «(/applicant/support/tickets)» → «(/applicant/support, проектируется)» — маршрут — /applicant/support, и это заглушка (applicant.routes.ts:115–117). <!-- v2:214 -->
- «(/admin/support/queue)» → «(/admin/support)»; «Сводная таблица всех открытых тикетов в системе.» → «Сводная таблица тикетов в системе.»; «(веб-сокеты / лонг-пуллинг)» → «(опрос сервера каждые 10 с)» — экран один — /admin/support; ListTickets запрашивается без фильтра статуса; обновление — interval(10000) (admin.routes.ts:150–153, support-api.service.ts:45–48, admin-support.component.ts:50–52). <!-- v2:215 -->
- «(/support/tickets/:id)» → «(проектируется; сейчас чат оператора — панель на /admin/support)» — маршрута /support/tickets/:id нет; чат оператора — компонент lib-operator-chat на странице /admin/support (admin-support.component.html:81–83). <!-- v2:216 -->

### 3.2. Формы, фильтры и валидация

- «(Angular Reactive Forms)» → «(шаблонная форма Angular, ngModel)»; «Поля «Тема» (минимум 10 символов) и «Описание проблемы» являются строго обязательными.» → «Поля «Тема тикета» и «Email пользователя» являются строго обязательными.» — форма на FormsModule/ngModel; isValid проверяет непустую тему и формат email, минимальной длины нет, описание необязательно (create-ticket-modal.component.ts:3, 9, 19–27, 222–226). <!-- v2:220 -->
- «Доступна сортировка по уровню критичности, по статусу, а также фильтр «Только мои тикеты».» → «Доступна сортировка по дате, критичности SLA и приоритету, а также фильтры по статусу, SLA и приоритету.»; «мягким красным фоном строки.» → «мягким красным фоном метки SLA.» — в блоке фильтрации — статус, SLA, приоритет и сортировка; фильтра «Только мои тикеты» нет; красным фоном выделяется метка SLA (admin-support.component.html:28–67, ticket-queue.component.scss:79, 102–105). <!-- v2:221 -->

### 3.3. Пользовательские сценарии

- «(Заявитель):» → «(Заявитель; проектируется):» — экраны заявителя и справки — заглушки (applicant.routes.ts:115–121). <!-- v2:228 -->
- «RPC-запрос SearchArticles.» → «RPC-запрос SearchArticles (проектируется).» — SearchArticles в контракте нет (support.proto:155–165). <!-- v2:229 -->
- «Сервис TicketRpcService регистрирует» → «Сервис SupportRpcService (метод CreateTicket) регистрирует»; «slaBreachTime» → «slaDeadline»; «в созданную карточку чата.» → «в созданную карточку чата (проектируется).» — RPC-сервис — SupportRpcService, метод CreateTicket; поле срока — slaDeadline; экрана карточки чата у заявителя нет (support-rpc.service.ts:34–35, support.service.ts:121–141, schema.prisma:778). <!-- v2:230 -->
- «(/admin/support/queue)» → «(/admin/support)»; «список активных инцидентов» → «список инцидентов»; «отсортированные по уровню приоритета и близости нарушения регламента SLA.» → «отсортированные по дате создания (сортировка по приоритету и SLA — в интерфейсе).» — маршрут /admin/support; интерфейс запрашивает все тикеты; сервер сортирует по createdAt desc (support-api.service.ts:45–48, support.service.ts:193, ticket-queue.component.ts:43–53). <!-- v2:232 -->
- «берет его на исполнение (TakeTicket).» → «берет его на исполнение кнопкой «Взять» (UpdateTicketStatus).»; «(assignedToId)» → «(assigneeId)» — метода TakeTicket нет: кнопка «Взять» вызывает UpdateTicketStatus(InProgress), который записывает assigneeId (ticket-queue.component.html:46–51, ticket-queue.component.ts:62–65, support.service.ts:317–319). <!-- v2:233 -->
- «(Оператор и Заявитель):» → «(Оператор и Заявитель; чат заявителя — проектируется):» — экран чата заявителя — заглушка (applicant.routes.ts:115–117). <!-- v2:234 -->
- «Вызывается RPC-метод SendMessage, создающий запись в таблице Message.» → «Вызывается RPC-метод AddMessage, создающий запись в таблице ticket_messages.» — метода SendMessage нет — сообщение отправляет AddMessage; таблица — ticket_messages (support-api.service.ts:87–90, support.service.ts:228–238, schema.prisma:807). <!-- v2:235 -->
- «push-уведомление для заявителя.» → «push-уведомление для заявителя (проектируется).»; «обратно в чат.» → «обратно в чат (экран заявителя — проектируется).» — SupportService не вызывает NotificationService; экран чата заявителя — заглушка (support.module.ts:7, applicant.routes.ts:115–117). <!-- v2:236 -->
- «(Заявитель):» → «(Оператор поддержки):» — закрыть тикет может только Admin (CloseTicket) (support.service.ts:329–330). <!-- v2:237 -->
- «заявитель нажимает на интерфейсную кнопку закрытия диалога. Компонент чата отправляет финальную команду обновления статуса.» → «оператор нажимает на интерфейсную кнопку «Закрыть» и подтверждает действие. Компонент чата отправляет финальную команду CloseTicket с итоговым решением.» — кнопка «Закрыть» в чате оператора: подтверждение, запрос итогового решения, затем CloseTicket (operator-chat.component.html:12, operator-chat.component.ts:34–46, support-api.service.ts:92–95). <!-- v2:238 -->
- «На клиенте форма ввода текста блокируется» → «На клиенте поле ввода ответа скрывается» — у закрытого тикета поле ответа не выводится (*ngIf status !== 'closed') (operator-chat.component.html:28). <!-- v2:239 -->

### Заключение

- «сущности тикетов, мгновенных сообщений, структурированных статей FAQ и системных пуш-нотификаций.» → «сущности тикетов, сообщений чата, структурированных статей FAQ (проектируется) и системных пуш-нотификаций (для тикетов — проектируется).» — статей в коде нет; сущность Notification и тип Push есть, но SupportService уведомлений не создаёт; сообщения не мгновенные — интерфейс опрашивает сервер раз в 10 с (schema.prisma:77–84, 491–505, support.module.ts:7, admin-support.component.ts:50–52). <!-- v2:268 -->
- «(чат коммуникации и поисковый модуль справки)» → «(чат коммуникации и поисковый модуль справки (проектируется))» — чат реализован, поиска и справки в коде нет (support.proto:155–165, admin.routes.ts:150–153). <!-- v2:269 -->
- «жесткие бизнес-правила автоматического контроля времени ответа сотрудников (SLA) и переходов по жизненному циклу обращений.» → «бизнес-правила автоматического расчёта срока ответа (SLA) по приоритету и смены статусов обращений.» — срок SLA вычисляется по приоритету при создании, время ответа не измеряется; порядок переходов сервер не проверяет (support.service.ts:296–327, 420–429, support.types.ts:35–40). <!-- v2:270 -->
- «позволяет исключить использование сторонних систем связи» → «позволяет исключить использование сторонних систем связи (интерфейс пользователей — проектируется)»; «обеспечивает прозрачный контроль качества работы операторов» → «обеспечивает контроль сроков обработки обращений (SLA)»; «механизм самопомощи через базу знаний.» → «механизм самопомощи через базу знаний (проектируется).» — экранов поддержки у пользователей нет; аудита и метрик по операторам нет — есть счётчики «Открытых», «Нарушено SLA», «Решено сегодня»; базы знаний в коде нет (applicant.routes.ts:115–121, notary.routes.ts:67–69, admin-support.component.html:5–18). <!-- v2:272 -->

### Список используемых источников

- «(Reactive Forms & Routing)» → «(Template-driven Forms & Routing)» — форма создания тикета шаблонная: FormsModule и ngModel, Reactive Forms в компоненте нет (create-ticket-modal.component.ts:3, 9, 19–27). <!-- v2:292 -->

## Рисунки и Приложение А

Диаграммы 1–4 перерисованы по исправленному PlantUML-коду из `diagrams/*.puml` в стиле Сергея (DejaVu Sans, dpi 150, без теней): исправлены только неверные имена, поля, связи и потоки, нереализованное нарисовано пунктиром или со стереотипом «проектируется». Ширина картинок на странице прежняя, высота — по пропорциям новой картинки.

**Рисунок 1 – Диаграмма развертывания (diagrams/deployment.puml)**

- оператор подписан как «Оператор поддержки (Admin)» — других ролей поддержки в коде нет (schema.prisma:18–24);
- Angular SPA: /admin/support — модуль поддержки (libs/web/admin/…/features/support вместо несуществующего libs/web/support), /applicant/support и /notary/support — каждый с пометкой «(проектируется)», связь «Заявитель → SPA» пунктиром (admin.routes.ts:150–153, applicant.routes.ts:115–117, notary.routes.ts:67–69);
- nginx portal: порт 80, проксирует Connect RPC (/notary.*) и REST (/api/*); браузер приходит на него через Nginx Proxy Manager (HTTP(S) → :80), сам portal порт наружу не публикует (apps/web/nginx/portal.conf:7, 35–56; apps/web/docker-compose.portal.yml:15–28; apps/web/DOCKER.md:36–46);
- NestJS API: SupportRpcService → SupportService вместо TicketRpcService → TicketService (connect-router.registry.ts:168–179); KnowledgeBaseService и оповещения NotificationService по тикетам — «(проектируется)» (support.module.ts:7);
- PostgreSQL: tickets, ticket_messages, users (schema.prisma:309, 788, 807); Article и notifications для тикетов — «(проектируется)».

**Рисунок 2 – Диаграмма классов (diagrams/classes.puml)**

- TicketPriority: Urgent вместо Critical; Role: Applicant, Notary, Admin вместо SupportOperator/SupportAuthor; добавлен enum TicketMessageRole (User, Ai, Support) — тип поля role сообщения (schema.prisma:18–24, 253–268);
- Ticket: subject вместо title, authorId/assigneeId вместо customerId/assignedToId, slaDeadline вместо slaBreachTime, добавлены resolution и resolvedAt; поля description в модели нет — текст обращения сохраняется первым сообщением; assigneeId необязателен, поэтому у связи «обрабатывает» кратность 0..1 (schema.prisma:771–793; support.service.ts:143–153);
- Message → TicketMessage: authorId вместо senderId, добавлены role и attachmentIds, isRead убран (schema.prisma:795–809); User.passwordHash — необязательный (schema.prisma:275);
- Notification: поля по схеме — type: NotificationType, message, sentAt, readAt вместо text, isRead, createdAt (schema.prisma:491–505);
- Article — стереотип «проектируется» и пунктир, связь User → Article пунктиром; «триггерит оповещения» — «(проектируется)» (в схеме Article нет, SupportService уведомлений не создаёт); перечисления выстроены в колонку, чтобы рисунок стал компактнее.

**Рисунок 3 – Диаграмма компонентов (diagrams/components.puml)**

- ChatComponent → OperatorChatComponent (operator-chat.component.ts:14); клиент «TicketService (Connect Client)» → «SupportService (Connect Client), SupportApiService» (support-api.service.ts:27–28);
- TicketRpcService → SupportRpcService, TicketService → SupportService (support-rpc.service.ts:25; support.service.ts:64);
- ArticleSearchComponent, ArticleViewComponent, KBService, KnowledgeBaseRpcService, KnowledgeBaseService — пунктир и «(проектируется)»: базы знаний в коде нет; разделение подсистем «чат/тикеты» и «справка/поиск» сохранено;
- SupportService → NotificationService — пунктир «оповещения (проектируется)» вместо «вызывает» (support.module.ts:7).

**Рисунок 4 – Диаграмма сценариев (diagrams/scenarios.puml)**

- участник TicketRpcService → «SupportRpcService → SupportService» (RPC-сервис только делегирует в SupportService, тот пишет в БД); поиск идёт в отдельного участника «KBService (проектируется)», экран справки и первый сценарий — «(проектируется)» (support-rpc.service.ts:24–54);
- CreateTicket(subject, text, priority) и slaDeadline вместо (title, description, priority) и slaBreachTime (support.proto:87–92; schema.prisma:778);
- ListTickets(page = 1, limit = 100), сортировка по createdAt desc, затем ListMessages для каждого тикета; диалог открывается кликом по строке очереди (support-api.service.ts:41–67, 128–134; support.service.ts:193; ticket-queue.component.html:15);
- кнопка «Взять» → UpdateTicketStatus(id, InProgress) с записью assigneeId вместо «Взять в работу» и TakeTicket (ticket-queue.component.html:46–51; support.service.ts:317–319);
- AddMessage вместо SendMessage; TicketMessage с role = Support/User вместо Message с isRead; ответ Admin заодно назначает его исполнителем и переводит Open/Resolved → InProgress; CreateTicket сохраняет text первым сообщением (support.service.ts:143–153, 225–254);
- push-уведомление, чат и ответ заявителя, а также блок 3 «чат заявителя» — «(проектируется)»; тикет закрывает оператор (Admin): «Закрыть» → CloseTicket(id, resolution), поле ответа скрывается (operator-chat.component.ts:34–46; operator-chat.component.html:28; support.service.ts:329–352);
- листинг заканчивается «@enduml» (у Сергея в конце стоял второй «@startuml»).

Код в Приложении А заменён точными текстами `diagrams/*.puml`; убраны строки «Фрагмент кода» и лишний «@startuml» в конце последнего листинга; заголовок «Исходный код диаграммы сценариев@startuml» разделён: теперь «Исходный код диаграммы сценариев (Рисунок 4)», а «@startuml» стал первой строкой кода. <!-- group:appendix --> <!-- v2:491 -->

## Оформление

- Подписи рисунков «Рисунок N – Название» по центру, шрифтом основного текста, нумерация 1–7 подряд: к диаграммам 1–4 подписи добавлены, у скриншотов подписями стали прежние метки: «Основная панель» → «Рисунок 5 – Основная панель» <!-- v2:255 -->, «Основная панель с окном чата» → «Рисунок 6 – Основная панель с окном чата» <!-- v2:257 -->, «Окно создания тикета» → «Рисунок 7 – Окно создания тикета» (пустой абзац между скриншотом и меткой убран, чтобы подпись стояла сразу под рисунком) <!-- v2:261 -->. Абзацам с рисунками задано «не отрывать от следующего», чтобы подпись не уходила на другую страницу. <!-- group:captions -->
- Пустые абзацы-распорки перед заголовками «Введение», «1.», «2.», «3.», «Заключение», «Список используемых источников», «Приложение а.» заменены на «разрыв страницы перед» у этих заголовков. <!-- group:spacers --> <!-- group:page-breaks -->
- После таблиц 3 и 6 добавлено по пустому абзацу перед заголовками 2.3 и 3.2. <!-- group:empty-after-tables -->
- Содержание: пункты в точности как заголовки в тексте (убран несуществующий «3.4. Текстовое описание интерфейсных компонентов», «3.5 Скриншоты компонентов» стал «3.4. Скриншоты компонента», у разделов 1–3 появились номера и страницы), номера страниц — по рендеру v3 в LibreOffice. <!-- group:toc -->
- Список источников: ссылка на Connect RPC вела на поиск Google — теперь ведёт на https://connectrpc.com/; добавлены пути к файлам проекта, на которые опираются таблицы и диаграммы (structure.md §10): «Контракт сервиса поддержки SupportService: libs/shared/api-contracts/proto/notary/support/v1alpha1/support.proto.»; «Серверная логика поддержки (права доступа, SLA, статусы тикетов): libs/api/support/src/lib/support.service.ts.»; «Модели данных Ticket, TicketMessage, User: libs/api/shared/prisma/schema.prisma.»; «Интерфейс оператора поддержки: libs/web/admin/src/lib/features/support/.»; «Развертывание: docker-compose.yaml, apps/web/docker-compose.portal.yml, apps/web/Dockerfile, apps/web/nginx/portal.conf.». <!-- group:sources -->

