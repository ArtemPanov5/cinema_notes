---  
title: "Backlog"  
type: overview  
status: draft  
created: "2026-04-06"  
updated: "2026-04-06"  
related_docs:  
  - architecture/system-overview.md 
  - architecture/core-system.md  
  - architecture/domain-model.md 
  - architecture/endpoints.md 
  - architecture/algorithms/algoritm-generatsii-kursa.md 
  - requirements/technical-specification.md tags: [architecture, api, contracts] 
summary: "[Stage 1 backlog системы Nastavnik]"  
---  
  
#### Functional topic tegs:  
- system graph preparation - [SystemPreReq]  
- course generation - [CoursGen]  
- topic generation - [TopicGen]  
- study track generation - [TrackGen]  
- documents uploading - [DocUp]  
- DBs (S3, pgvector, OLTP DB - PostgreSQL) - [DB]  
- Loggs collection - [logs]  
  
---  
#### Stage 0 (методолог):  
##### 1. [SystemPreReq]: создание Discipline методологом (back)  
Status: done  
  
User story: как методолог, я хочу дать системе эталонную предметную базу по дисциплине, чтобы далее можно было привязывать к ней эталонные компетенции темы и KSA  
  
Functional topic tegs: [SystemPreReq]  
  
Result: в системе создана `Discipline`, доступная для дальнейшего наполнения системного графа и формирования дисциплинарного шаблона  
  
Acceptance criteria:  
- методолог может создать `Discipline`  
- после создания у `Discipline` сохраняются `name`, `description`  
- `Discipline` в системе получает поля `id`,`created_at`, `updated_at`  
- `Discipline` должна быть доступна по FK Discipline при создании `DisciplineConfig`, `SystemDisciplineCompetency`, `SystemDiscriplineTopic`  
  
---  
#### Stage 1:  
##### 1. [CoursGen]: создание Course создателем курса (back)  
Status: done  
  
User story: Как создатель курса, я хочу создать новый курс, указав базовую информацию о нём, чтобы задать контекст для загрузки материалов и последующей генерации графа. (US-117)  
  
Functional topic tegs: [CoursGen]    
Result: в системе создан `Course`, доступный для загрузки материалов, выбора источника структуры и генерации графа курса.    
  
Acceptance criteria:  
- создатель курса может создать `Course`  
- при создании указываются базовые поля курса:`title`, `discipline`,`description`, `target_audience`, `audience_level`  
- у `Course` сохраняются: `id`, `owner`, `status`, `graph_gen_status`, `created_at`, `updated_at`  
- при первичном создании: `status = draft`, `graph_gen_status = not_started`, `source_structure = NULL`  
- созданный `Course` должен быть доступен по FK Course при создании `Document`, `CourseDocument`, последующей генерации `Topic`, `Competency` и связей графа курса.  
##### 2. [DocUp]: загрузка Document создателем курса (back)  
Status: done  
  
User story: Как создатель курса, я хочу загрузить материалы курса (PDF, DOCX) перед выбором структуры, чтобы система могла использовать их для автоматической генерации графа. (US-118а/US-125)  
  
Functional topic tegs: [DocUp]  
  
Result: в системе создана сущность `Document`, доступная для дальнейшего использования системой для автоматической генерации графа  
  
Acceptance criteria:  
- создатель курса может создать (загрузить) `Document` в формате PDF/DOCX  
- при загрузке создаётся запись `Document`, связанная с `Course`  
- `Document` в системе получает поля `id`, `course`, `uploaded_by`, `title`, `storage_key`, `file_type`, `parse_status`, `created_at`, `updated_at`  
- при первичной загрузке `topic = NULL`, если документ загружен на уровень курса, а не прикреплён к конкретной теме  
- после загрузки `parse_status` устанавливается в `pending`  
- `Document` должен быть доступен по FK Document при создании `DocumentSection` после успешного парсинга  
##### 3. [CoursGen]: выбор источника структуры курса (back)  
Status: done  
  
User story: Как создатель курса, я хочу выбрать источник структуры курса — автогенерация из загруженных материалов, копия существующего курса, шаблон домена знаний или ручной ввод, чтобы выбрать удобный способ построения программы. (US-118б)  
  
Functional topic tegs: [CoursGen]  
  
Result: выбран источник структуры курса, на основе которого будет сформирован контекст генерации графа курса  
  
Acceptance criteria:   
- создатель курса может выбрать один из источников структуры:  
    - выбрать дисциплинарный шаблон из системы. универсальный - применяется всегда  
    - копия существующего курса  
    - ручной ввод  
- при выборе автогенерации система использует загруженные материалы курса (`Document`, `DocumentSection`)  
- при выборе копии существующего курса мы используем граф этого курса (`Course`, `Topic`, `TopicDependency`, `Competency`, `CompetencyKSA`, `TopicKSA`, `KSADependency`, `Document`, `DocumentSection`)  
- в алгоритмическом описании для привязки материалов к курсу/теме также используется обозначение `CourseDocument`  
- при выборе дисциплинарного шаблона используется: `UniversalTemplate` (by defolt) и `SystemGraph` (срез по `discipline_id` + `DisciplineConfig`: `Discipline`, `DisciplineConfig`, `SystemCompetency`, `SystemKSA`, `SystemTopic`, `SystemTopicDependency` и связующие таблицы в DB)  
- в `Course.source_structure` сохраняется выбранный тип источника  
##### 4. [CoursGen]: формирование `contentmap` из материалов курса (back)  
Status: done  
  
User story: Как система, я хочу сформировать `contentmap` из считанных секций документов курса, чтобы использовать структурированное содержание материалов как источник для генерации графа курса.    
  
Functional topic tegs: [CoursGen]    
Result: в системе сформирован `contentmap` на основе `DocumentSection` прикреплённых документов курса.    
  
Acceptance criteria:  
- в `contentmap` включаются данные из `DocumentSection` документов, привязанных к `Course`  
- для формирования `contentmap` используются только успешно обработанные материалы курса  
- `contentmap` используется при сборке пользовательского промпта генерации графа курса  
##### 5. [CoursGen]: сборка системного промпта генерации графа Course (back)  
Status: done  
  
User story: Как система, я хочу собрать системный промпт генерации графа курса на основе универсальных, дисциплинарных и методологических ограничений, чтобы задать LLM правила построения структуры курса.    
  
Functional topic tegs: [CoursGen]    
Result: в системе сформирован системный промпт для генерации графа курса.    
  
Acceptance criteria:  
- системный промпт собирается из:  
  - жёстких ограничений `UniversalTemplate`  
    - дисциплинарных ограничений `DisciplineConfig`  
    - методологических инструкций `ADDIE + 4C/ID`  
    - генерационных умолчаний `UniversalTemplate`  
    - ориентиров дисциплинарного шаблона  
- `UniversalTemplate` используется как источник жёстких ограничений и генерационных умолчаний  
- `DisciplineConfig` используется как источник дисциплинарных ограничений  
- системный промпт используется при вызове генерации графа курса  
##### 6. [CoursGen]: сборка пользовательского промпта генерации графа Course (back)  
Status: done  
  
User story: Как система, я хочу собрать пользовательский промпт генерации графа курса из метаданных курса и `contentmap`, чтобы передать в LLM фактическое задание на построение структуры курса.   
  
Functional topic tegs: [CoursGen]    
Result: в системе сформирован пользовательский промпт генерации графа курса.    
  
Acceptance criteria:  
- пользовательский промпт собирается из:  
  - метаданных курса: `title`, `description`, `discipline`, `audience_level`  
    - `contentmap`  
    - ориентира по объёму: `min_topics`, `max_topics`  
    - `DisciplineConfig`  
- пользовательский промпт используется при вызове `POST ml-llm generate-course-graph`  
##### 7. [CoursGen]: LLM-генерация графа Course (back)  
Status: done  
  
User story: Как создатель курса, я хочу чтобы система сгенерировала черновой граф курса по материалам и системному графу дисциплины, чтобы получить исходную структуру курса для последующей проверки и редактирования.    
  
Functional topic tegs: [CoursGen]    
Result: в системе сгенерирован черновой граф курса (`Topic`, `TopicKSA`, `TopicDependency`, `Competency`, `CompetencyKSA`), связанный с `Course`.    
  
Acceptance criteria:  
- генерация запускается через LLM после подготовки контекста и системного промпта  
- перед запуском генерации `Course.graph_gen_status` переводится в `processing`  
- LLM при генерации:  
  - выбирает релевантные `SystemCompetency` для материалов курса  
  - выбирает `SystemKSA` из системного графа  
  - группирует KSA в `Topic`  
    - расставляет `TopicDependency`  
    - назначает сложность и часы  
- в результате генерации создаются:  
    - `Competency`  
    - `CompetencyKSA`  
    - `Topic`  
    - `TopicKSA`  
    - `TopicDependency`  
- направление маппинга сохраняется:  
    - `SystemCompetency -> Competency.system_competency`  
    - `SystemKSA -> TopicKSA.system_ksa`  
    - `SystemTopic -> Topic.system_topic`  
- для созданных `Topic`, `TopicKSA`, `Competency` поле `source` устанавливается в `generated`  
##### 8. [CoursGen]: валидация графа Course по `UniversalTemplate` (back)  
Status: done  
  
User story: Как система, я хочу автоматически провести валидацию сгенерированного графа курса по жёстким ограничениям шаблона, чтобы не сохранять невалидную структуру курса.    
  
Functional topic tegs: [CoursGen]    
Result: в системе выполнена проверка чернового графа курса на соответствие шаблону и методологическим ограничениям.    
  
Acceptance criteria:  
- после генерации `FastAPI ML Inference Services` выполняет валидацию графа курса  
- валидация проверяет:  
  - соответствие `UniversalTemplate`  
    - покрытие компетенций через `CompetencyKSA`  
    - привязку `TopicKSA` к темам  
  - отсутствие циклов в `TopicDependency`  
- `TopicDependency` должен быть ориентированным ациклическим графом  
- при установлении зависимостей учитывается только логическая зависимость между темами  
- при валидации запрещено использовать:  
  - субъективную оценку сложности студентом  
  - расписание и временные ограничения  
  - предпочтения по формату  
- валидация выполняется средствами `Verification` внутри `FastAPI ML Inference Services`  
##### 9. [CoursGen]: сохранение графа Course и фиксация статуса генерации (back)  
Status: in progress  
  
User story: Как система, я хочу сохранить валидный граф курса в БД и зафиксировать статус генерации, чтобы курс можно было открыть на редактирование и последующую публикацию.    
  
Functional topic tegs: [CoursGen]    
Result: в системе сохранён валидный черновой граф курса, обновлены статус генерации и версия универсального шаблона.    
  
Acceptance criteria:  
- при успешной генерации и успешной валидации в БД сохраняются: `Topic`, `TopicKSA`, `TopicDependency`, `Competency`, `CompetencyKSA`  
- в `Course.universal_template_version` фиксируется версия `UniversalTemplate`, использованная при генерации  
- при успешной генерации `Course.graph_gen_status = done`  
- при неуспешной генерации или неуспешной валидации `Course.graph_gen_status = failed`  
- после успешного сохранения граф доступен создателю для редактирования и последующей публикации. 
##### 9.1 [CoursGen]: генерация `LearningOutcome` для `TopicKSA`
Status: draft task

User story: Как система, я хочу сгенерировать образовательные результаты для каждого `TopicKSA`, чтобы граф курса включал измеримые цели обучения.

Functional topic tegs: [CoursGen]

Result: в системе созданы `LearningOutcome`, связанные с `TopicKSA`.

Acceptance criteria:
- для каждого релевантного `TopicKSA` создаётся 1–2 `LearningOutcome`
- `LearningOutcome` содержит `title`, `description`, `bloom_level`, `display_order`, `source`
- автоматически созданные `LearningOutcome` получают `source = generated`
- после ручного редактирования графа создатель курса может добавлять и менять `LearningOutcome`

##### 9.2 [CoursGen]: генерация `LearningTask` для `LearningOutcome`
Status: draft task

User story: Как система, я хочу сгенерировать учебные задачи для образовательных результатов, чтобы в курсе появились конкретные задания, ведущие к достижению результата.

Functional topic tegs: [CoursGen]

Result: в системе созданы `LearningTask`, связанные с `LearningOutcome`.

Acceptance criteria:
- для каждого релевантного `LearningOutcome` создаётся 1–2 `LearningTask`
- `LearningTask` содержит `title`, `description`, `correct_answer`, `display_order`, `source`
- автоматически созданные `LearningTask` получают `source = generated`
- после ручного редактирования графа создатель курса может добавлять и менять `LearningTask`

##### 9.3 [CoursGen]: регенерация графа `Course` при неуспешной валидации
Status: draft task

User story: Как система, я хочу повторно запускать генерацию графа курса при неуспешной валидации или ошибке LLM, чтобы попытаться получить валидный граф без ручного перезапуска со стороны создателя.

Functional topic tegs: [CoursGen]

Result: для генерации графа курса реализован retry-контур с ограничением числа попыток и фиксацией финального статуса.

Acceptance criteria:
- при ошибке `422/504/5xx` или неуспешной валидации запускается повторная попытка генерации
- число попыток генерации ограничено конфигурацией
- между попытками выдерживается задержка
- каждая попытка логируется отдельно
- если лимит попыток исчерпан, `Course.graph_gen_status = failed`
- создатель курса получает уведомление о неуспешной генерации
##### 10. [DB]: Обработка Document: parse PDF/DOCX → DocumentSection  
Status: done  
  
User story: Как система, я хочу произвести извлечение контента (парсинг) из загруженного `Document` и выделить из него `DocumentSection`, чтобы использовать структурированные части документа при формировании `content_map` и дальнейшем RAG.  
  
Functional topic tegs: [DB]  
  
Result: в системе созданы `DocumentSection` для успешно обработанного документа курса  
  
Acceptance criteria:   
- после загрузки `Document` ставится в асинхронную обработку через `Redis/Celery`  
- обработка включает `parse PDF/DOCX` и извлечение логических секций документа  
- для каждого успешно обработанного документа создаются записи `DocumentSection`, связанные с `Document`  
- `Document.parse_status` обновляется в соответствии с результатом обработки (`pending → processing → completed/failed`)  
- только `DocumentSection` из успешно обработанных документов используются при формировании `content_map`  
- структура `DocumentSection` соответствует доменной модели (содержит текст секции и связь с документом)  
##### 11. [DB]: Embedding и RAG-индексация обработанных секций  
Status: done  
  
User story: Как система, я хочу получить эмбеддинги для `DocumentSection` и проиндексировать их в `pgvector`, чтобы использовать материалы курса в `RAG profile 1: doc_context` при генерации графа курса.  
  
Functional topic tegs: [DB]  
  
Result: успешно обработанные секции документа имеют эмбеддинги и доступны для RAG-поиска в `doc_context`  
  
Acceptance criteria:   
- в embedding pipeline попадают только успешно обработанные `DocumentSection`  
- `Celery worker` вызывает внутренний `embedding` интерфейс `FastAPI ML Inference Services`  
- embedding-сервис возвращает векторы для переданных текстов секций  
- результаты записываются в векторное хранилище `pgvector`  
- на текущем этапе используется только `RAG profile 1: doc_context`  
- данные индекса доступны для вызова `POST /ml-llm/generate-course-graph`  
### 12. [DB]: OLTP schema + Django migrations for Stage 0 / Stage 1 course graph generation flow  
Status: done  
  
User story: Как разработчик системы, я хочу иметь согласованную PostgreSQL-схему, описанную через Django models и migrations, чтобы backend и ML-сервисы могли поддерживать Stage 0 (системный граф дисциплины) и Stage 1 flow генерации графа курса: создание `Course`, загрузку и обработку `Document`, хранение `DocumentSection`, использование `UniversalTemplate`/`DisciplineConfig`/`SystemGraph` и сохранение результата генерации курса.  
  
Functional topic tegs: [DB]  
  
Result: В репозитории есть Django models, миграции и генерируемый из них SQL bootstrap/diff, покрывающие Stage 0 + Stage 1 flow до сохранения графа курса.  
  
Acceptance criteria:   
-  Django models покрывают Stage 0 + Stage 1 flow генерации графа курса.  
- Все таблицы создаются и изменяются через Django migrations.  
- На пустой PostgreSQL БД схема поднимается одной командой `migrate`.  
- Есть отдельная migration на `CREATE EXTENSION IF NOT EXISTS vector`.  
- Есть seed/fixture хотя бы для одного `UniversalTemplate`.  
- Генерируется `bootstrap.sql` из миграций для handoff ML/backend.  
- happy path проходит: `Discipline` → `Course` → `Document` → `DocumentSection` → generate graph → save `Topic/TopicKSA/Competency/...  
##### 13. [logs]: Langfuse tracing для каждой попытки LLM-генерации графа  
Status: in progress  
  
User story: Как система, я хочу сохранять trace каждой попытки вызова LLM-генерации графа курса в `Langfuse`, чтобы отслеживать выполнение внутреннего ML-flow.  
  
Functional topic tegs: [logs]  
  
Result: каждый вызов `POST /ml-llm/generate-course-graph` логируется в `Langfuse`  
  
Acceptance criteria:   
- `FastAPI ML Inference Services` создаёт trace для каждого вызова `POST /ml-llm/generate-course-graph`  
- trace создаётся как для успешной генерации, так и для неуспешного вызова  
- trace связан с конкретным вызовом генерации графа курса  
- логирование ограничивается `Langfuse`

##### 14. [CoursGen]: модуль discipline_template
Status: draft task
  
User story: Как система, я хочу собирать `DisciplineTemplate` как runtime-срез системного графа по `discipline` и `main_topic`, чтобы дать LLM дисциплинарный ориентир при генерации графа курса.
  
Functional topic tegs: [CoursGen]
  
Result: в системе формируется `DisciplineTemplate`, который используется при сборке системного промпта генерации графа курса.
  
Acceptance criteria: 
- `DisciplineTemplate` строится на основе `SystemTopic`, `SystemTopicDependency`, `SystemCompetency`, `SystemKSA` и связей системного графа  
- если для курса задан `main_topic`, система ищет ближайшую системную тему семантическим поиском  
- если релевантная системная тема найдена, в `DisciplineTemplate` попадают ближайшие темы, компетенции и KSA  
- если релевантная тема не найдена, генерация графа курса продолжается без `DisciplineTemplate`  
- `DisciplineTemplate` используется как один из входов при сборке system prompt
##### 15. [CoursGen]: модуль для summary для `Document` и  `DocumentSection`
Status: draft task
  
User story: Как система, я хочу формировать summary для документа и его секций, чтобы сократить шум в `contentmap` и улучшить качество course/topic generation, а также сократить размер `ContentMap`
  
Functional topic tegs: [CoursGen]
  
Result: у успешно обработанных `Document` и `DocumentSection` доступны краткие summary для дальнейшего использования в `contentmap` и RAG.
  
Acceptance criteria: 
- summary строятся только для секций документов, у которых `Document.parse_status = done`  
- для каждого `DocumentSection` сохраняется краткое `summary`  
- для каждого `Document` может сохраняться агрегированное `summary` по его секциям  
- при сборке `contentmap` используется `summary` вместо полного текста там, где это требуется pipeline

##### 16. [CoursGen]: подтверждение готовности графа курса и запуск асинхронной генерации `TopicContent`
Status: draft task
  
User story: Как создатель курса, я хочу подтвердить готовность отредактированного графа курса, чтобы система запустила генерацию контента по темам.
  
Functional topic tegs: [CoursGen]
  
Result: после подтверждения графа курс переходит в этап контентной генерации, и для него запускается асинхронный pipeline генерации `TopicContent`.
  
Acceptance criteria: 
- перед подтверждением создатель может вручную редактировать `Topic`, `TopicKSA`, `TopicDependency`, `Competency`, `CompetencyKSA`, `LearningOutcome`, `LearningTask`  
- если создатель не подтверждает граф, курс остаётся в черновом состоянии  
- после подтверждения графа Django ставит Celery-задачу `generate_topic_content_for_course(course_id)`  
- после запуска задачи `Course.content_gen_status = processing`  
- подтверждение графа является пользовательским действием, переводящим курс в этап контентной генерации
##### 17. [TopicGen]: выбор `content_source` для `TopicContent`
Status: draft task

User story: Как система, я хочу определять источник контента для каждой темы курса, чтобы выбрать корректный pipeline генерации `TopicContent`.

Functional topic tegs: [TopicGen]

Result: для каждой темы определён `content_source`, на основе которого выбирается режим генерации `TopicContent`.

Acceptance criteria:
- если к теме прикреплены документы, выбирается источник `topic_docs`
- если у темы нет своих документов, но есть документы курса, выбирается источник `course_docs`
- если документов нет, выбирается источник `model_only`
- если для темы вручную задан `TopicContent`, система учитывает ручной режим как приоритетный
- выбранный источник используется downstream при генерации `TopicContent`

##### 18. [TopicGen]: создание и редактирование `TopicContent`
Status: draft task

User story: Как создатель курса, я хочу вручную задать или отредактировать `TopicContent` для темы, чтобы использовать авторский longread вместо полностью автоматической генерации.

Functional topic tegs: [TopicGen]

Result: в системе можно создать, сохранить, отредактировать и удалить `TopicContent` для конкретной темы курса.

Acceptance criteria:
- создатель курса может создать `TopicContent` для `Topic`
- создатель курса может редактировать уже существующий `TopicContent`
- создатель курса может удалить `TopicContent`
- создатель курса может прикрепить документы к теме перед повторной генерацией
- после ручного редактирования `TopicContent` его чанки пересобираются
- ручной `TopicContent` имеет `source = manual`

##### 19. [TopicGen]: RAG/non-RAG generation pipeline для `TopicContent`
Status: draft task

User story: Как система, я хочу автоматически генерировать `TopicContent` для каждой темы с учётом доступного контекста, чтобы у курса был базовый контент по всем темам.

Functional topic tegs: [TopicGen]

Result: для темы автоматически создаётся `TopicContent` через `RAG` или без `RAG`, в зависимости от доступных материалов.

Acceptance criteria:
- если есть документы темы, генерация идёт по `Document` и `DocumentSection`, прикреплённым к теме
- если документов темы нет, но есть документы курса, генерация идёт по документам курса
- если документов нет, генерация идёт без `RAG`
- выбранный режим контекста фиксируется в `TopicContent.context_level`
- если для темы выбран manual-режим и `TopicContent` отсутствует, создаётся запись `TopicContentGenerationError` и создателю отправляется уведомление
- при ошибке генерации создаётся запись `TopicContentGenerationError` с `error_type`, `error_message`, `topic`, `course`
- активной считается ошибка, у которой `resolved_at IS NULL`

##### 20. [TopicGen]: сохранение результата генерации `TopicContent` в DB
Status: draft task

User story: Как система, я хочу сохранять результат генерации `TopicContent` в БД, чтобы тема имела доступный контент для дальнейшего использования в уроках и треках.

Functional topic tegs: [TopicGen]

Result: в системе сохранён `TopicContent`, связанный с конкретной темой курса, а курс получает итоговый статус контентной генерации.

Acceptance criteria:
- при успешной генерации создаётся или обновляется `TopicContent`
- автоматически сгенерированный `TopicContent` имеет `source = generated`
- при успешной перегенерации ошибки по теме соответствующая запись `TopicContentGenerationError` закрывается (`resolved_at` заполняется)
- при ошибке одной темы остальные темы продолжают обрабатываться
- после завершения генерации по всем темам `Course.content_gen_status` переводится в `done` или `done_with_errors`
- если Celery-задача контентной генерации упала целиком, `Course.content_gen_status = failed`
- после завершения обработки всех тем создателю отправляется одно уведомление о результате контентной генерации по курсу

##### 21. [TopicGen]: разбиение `TopicContent` на `TopicContentChunk` и embedding
Status: draft task

User story: Как система, я хочу разбивать `TopicContent` на чанки и индексировать их в `pgvector`, чтобы использовать контент темы в последующем retrieval.

Functional topic tegs: [TopicGen]

Result: `TopicContent` разбит на `TopicContentChunk`, а чанки доступны для retrieval.

Acceptance criteria:
- после сохранения `TopicContent` запускается pipeline chunking
- для каждой темы создаются `TopicContentChunk`
- для чанков вычисляются embeddings
- embeddings сохраняются в `pgvector`
- при ручном редактировании `TopicContent` чанки и embeddings пересобираются

##### 22. [TrackGen]: сохранение анкеты пользователя для генерации трека
Status: draft task

User story: Как пользователь, я хочу заполнить анкету перед генерацией трека, чтобы система учла мою цель, дедлайн, нагрузку и параметры персонализации трека.

Functional topic tegs: [TrackGen]

Result: в системе сохранены данные анкеты пользователя, используемые для генерации `UserTrack`.

Acceptance criteria:
- пользователь может указать цель обучения
- пользователь может указать дедлайн
- пользователь может указать недельную нагрузку
- пользователь может указать субъективно простые и сложные темы
- пользователь может указать, готов ли он учиться короткими слотами по 10–15 минут
- пользователь может указать, по какому признаку он поймёт, что навык действительно освоен
- данные анкеты сохраняются в сущностях, используемых для генерации `UserTrack`
- `UserTrack.goal`, `UserTrack.deadline`, `UserTrack.weekly_hours` доступны downstream при генерации трека

##### 23. [TrackGen]: генерация диагностического теста
Status: draft task

User story: Как система, я хочу сгенерировать диагностический тест по опубликованному курсу, чтобы оценить стартовый уровень пользователя перед построением трека.

Functional topic tegs: [TrackGen]

Result: в системе создан `DiagnosticTest`, связанный с курсом и покрывающий ключевые темы и KSA.

Acceptance criteria:
- диагностический тест создаётся только для опубликованного курса
- для каждого диагностического вопроса создаётся `ContentArtifact` с `created_for = diagnostic`
- `DiagnosticQuestion.content_id` ссылается на соответствующий `ContentArtifact`
- `DiagnosticQuestion` связывается с `Topic`, а при необходимости — с `Competency` и `primary_ksa_id`
- пользователь может пройти `DiagnosticAttempt`
- ответы пользователя сохраняются в `DiagnosticAnswer`

##### 24. [TrackGen]: инициализация графа знаний пользователя
Status: draft task

User story: Как система, я хочу создать граф знаний пользователя на основе опубликованного курса, чтобы иметь стартовую персональную модель тем, компетенций и KSA.

Functional topic tegs: [TrackGen]

Result: в системе созданы `UserKnowledgeNode` и `UserKSANode`, а также `UserCompetencyNode` там, где возможен маппинг на системную компетенцию.

Acceptance criteria:
- при запуске `TrackGen` система создаёт `UserKnowledgeNode` для тем курса
- при запуске `TrackGen` система создаёт `UserKSANode` для KSA курса
- `UserCompetencyNode` создаётся для тех компетенций курса, которые имеют маппинг на `SystemCompetency`
- `UserKnowledgeNode` получает обязательные поля `name`, `embedding`, `source`
- все темы курса попадают в `UserKnowledgeNode` со статусом `not_evaluated`
- стартовое значение `bkt_probability = 0.0`
- если соответствующие узлы уже существуют, система не создаёт дубликаты
- `UserTrackTopic.knowledge_node` должен ссылаться на соответствующий `UserKnowledgeNode`

##### 25. [TrackGen]: инициализация BKT и обновление графа знаний пользователя на основе прохождения диагностического тестирования
Status: draft task

User story: Как система, я хочу обновить стартовые значения BKT по результатам диагностического теста, чтобы точнее построить персональный учебный трек.

Functional topic tegs: [TrackGen]

Result: после прохождения диагностики обновлены `UserKSANode`, `UserKnowledgeNode` и агрегированные оценки освоения.

Acceptance criteria:
- после проверки `DiagnosticAnswer` вычисляются стартовые значения BKT для релевантных `TopicKSA`
- обновляется `UserKSANode.bkt_probability`
- агрегированно пересчитываются `UserKnowledgeNode.bkt_probability`
- агрегированно пересчитываются `UserCompetencyNode.mastery_level`
- если пользователь не проходит диагностику, генерация трека всё равно возможна, но менее точна

##### 26. [TrackGen]: генерация персонального учебного трека
Status: draft task

User story: Как пользователь, я хочу получить персональный учебный трек по курсу, анкете и диагностике, чтобы видеть порядок изучения тем и приоритеты.

Functional topic tegs: [TrackGen]

Result: в системе создан `UserTrack` и упорядоченный набор `UserTrackTopic`.

Acceptance criteria:
- `UserTrack` создаётся на основе курса, анкеты, графа знаний пользователя и результатов диагностики
- при генерации учитываются пререквизиты, сложность, мотивация и расписание
- в `UserTrack` сохраняются `goal`, `deadline`, `weekly_hours`, `status`
- для трека создаются `UserTrackTopic`
- темы упорядочиваются через `order_index`
- для трека создаются `UserTrackTopicDependency`
- стартовый статус трека — `draft`

##### 27. [TrackGen]: возможность для пользователя принимать или отклонять рекомендованные темы
Status: draft task

User story: Как пользователь, я хочу принимать или отклонять дополнительные темы, предложенные системой, чтобы персональный трек учитывал мои реальные потребности.

Functional topic tegs: [TrackGen]

Result: система может предложить дополнительные темы, а пользователь может подтвердить или отклонить их включение в трек.

Acceptance criteria:
- система может добавлять в трек рекомендованные темы
- рекомендованные темы создаются с `is_recommended = true`
- пользователь может принять или отклонить рекомендацию
- решение пользователя сохраняется в `recommendation_status`
- при принятии тема остаётся в треке
- при отклонении тема не участвует в активном плане

##### 28. [CoursGen]: публикация `Course` после завершения контентной генерации
Status: draft task

User story: Как создатель курса, я хочу опубликовать курс после завершения генерации контента и проверки ошибок, чтобы курс стал доступен для генерации треков.

Functional topic tegs: [CoursGen]

Result: курс опубликован, граф курса зафиксирован, и курс доступен для дальнейшего student flow.

Acceptance criteria:
- если `Course.content_gen_status = done`, публикация разрешена
- если `Course.content_gen_status = done_with_errors`, публикация требует явного подтверждения при наличии активных `TopicContentGenerationError`
- если `Course.content_gen_status = failed`, публикация запрещена
- если для темы выбран manual-режим и обязательный `TopicContent` отсутствует, публикация запрещена
- после публикации `Course.status = published`
- дата публикации фиксируется в `Course.published_at`
- после публикации граф курса считается зафиксированным источником для `TrackGen`