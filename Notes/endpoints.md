---  
title: "API Endpoints"  
type: overview  
status: draft  
created: "2026-04-06"  
updated: "2026-04-06"  
related_docs:  
- architecture/system-overview.md 
- architecture/core-system.md  
- architecture/domain-model.md 
- architecture/algorithms/algoritm-generatsii-kursa.md 
- requirements/technical-specification.mdtags: [architecture, api, contracts]  
summary: "[Контракты внешних и внутренних API эндпоинтов системы Nastavnik]"  
---  
  
## Django Backend / Orchestrator:  
### 1. Cоздание сущности `Discipline`  
1) Назначение: cоздать сущность `Discipline` и сохранить её базовые поля `name`, `description` для последующего наполнения системного графа и создания `DisciplineConfig`/`SystemDisciplineCompetency`/`SystemDiscriplineTopic`  
2) Метод и путь: `POST /api/v1/disciplines`  
3) Кто вызывает: методолог  
4) Права: `access_token` авторизованного пользователя с правами на изменение системного графа  
5) Input: JSON  
```json  
{  "name": "Математическая статистика",  
"description": "Эталонная дисциплина для курсов по статистике и анализу данных"  
}  
```  
  
6) Output: JSON  
```json  
{  
  "id": "UUID",  "name": "Математическая статистика",  "description": "Эталонная дисциплина для курсов по статистике и анализу данных",  "created_at": "2026-03-21T20:10:00Z",  "updated_at": "2026-03-21T20:10:00Z"}  
```  
7) sub_actions:  
- проверка `access_token`  
- проверка прав пользователя  
- валидация `name`  
- проверка уникальности `name`  
- создание `Discipline`  
- возврат созданной записи  
- обеспечение доступности `Discipline` по FK для `DisciplineConfig`, `SystemDisciplineCompetency`, `SystemDisciplineTopic`  
8) Errors:  
- `400` — invalid входные данные  
- `401` — пользователь не авторизован  
- `403` — нет прав на изменение системного графа  
- `409` — дисциплина с таким названием уже существует  
### 2. Cоздание сущности `Course`  
1) Назначение: создать сущность `Course` и сохранить базовые поля курса, чтобы задать контекст для последующей загрузки материалов, выбора источника структуры и генерации графа курса.  
2) Метод и путь: POST /api/v1/courses  
3) Кто вызывает: создатель курса  
4) Права: `access_token` авторизованного пользователя с правами создателя курса  
5) Input: JSON  
```json  
{  
  "title": "Основы математической статистики",  "discipline_id": "UUID",  "description": "Курс по базовым методам математической статистики",  "target_audience": "Студенты 2-3 курса технических направлений",  "audience_level": 3,  "main_topic": "Вероятность и статистика",  "goals": ["Студент сможет рассчитать p-value"],  "included_topics": ["Линейная алгебра"],  "excluded_topics": ["Введение в Python"],  "target_hours": 40}  
```  
6) Output: JSON  
```json  
{  
  "id": "UUID",  "owner": "USER_ID",  "title": "Основы математической статистики",  "discipline_id": "UUID",  "description": "Курс по базовым методам математической статистики",  "target_audience": "Студенты 2-3 курса технических направлений",  "status": "draft",  "graph_gen_status": "not_started",  "audience_level": 3,  "source_structure": "",  "main_topic": "Вероятность и статистика",  "goals": ["Студент сможет рассчитать p-value"],  "included_topics": ["Линейная алгебра"],  "excluded_topics": ["Введение в Python"],  "target_hours": 40,  "created_at": "2026-03-26T08:22:24Z",  "updated_at": "2026-03-26T08:22:24Z"}  
```  
7) sub_actions:   
- проверка `access_token`  
- проверка прав пользователя  
- валидация `title`  
- валидация `discipline_id`  
- валидация `audience_level`  
- создание `Course`  
- установка стартовых значений:  
    - `status = draft`  
    - `graph_gen_status = not_started`  
    - `course_type = course`  
    - `enrollment_type = invite_only`  
    - `is_file = false`  
- возврат созданной записи  
- обеспечение доступности `Course` по FK для `Document`, `CourseDocument`, `Topic`, `Competency` и других сущностей графа курса  
8) Errors:   
- `400` — invalide входные данные  
- `401` — пользователь не авторизован  
- `403` — нет прав на создание курса  
- `404` — `Discipline` не найдена  
### 3. Загрузка Document  
1) Назначение: загрузить файл курса (`PDF`, `DOCX`) как учебный материал, сохранить сущность `Document`, связать её с `Courses` и поставить файл в пайплайн обработки для дальнейшего парсинга, чанкинга, эмбеддингов и RAG.  
2) Метод и путь: POST /api/v1/courses/{course_id}/documents  
3) Кто вызывает: создатель курса  
4) Права: `access_token` авторизованного пользователя с правами на редактирование данного `Course`/`Courses`  
5) Input: multipart/form-data  
```json  
{  
  "title": "lecture_01_intro.pdf",  "file": "<binary PDF/DOCX>"}  
```  
6) Output: JSON  
```json  
{  
  "id": "UUID",  "courses": ["UUID"],  "topic": null,  "uploaded_by": "USER_ID",  "title": "lecture_01_intro.pdf",  "storage_key": "courses/UUID/documents/UUID/lecture_01_intro.pdf",  "file_type": "pdf",  "parse_status": "pending",  "created_at": "2026-03-27T08:40:00Z",  "updated_at": "2026-03-27T08:40:00Z"}  
```  
7) sub_actions:   
- проверка `access_token`  
- проверка прав пользователя на редактирование `Course`  
- проверка существования `Course`  
- валидация файла:  
  - файл передан  
  - тип файла входит в `pdf/docx`  
- загрузка файла в объектное хранилище и формирование `storage_key`  
- создание `Document`  
- установка:  
    - `Document ↔ Course` / `courses`  
    - `topic = NULL`  
    - `uploaded_by = current_user`  
    - `parse_status = pending`  
- постановка фоновых задач на обработку файла:  
    - `parse PDF/DOCX`  
    - извлечение секций документа  
    - `chunking`  
    - `embeddings`  
    - `RAG`-индексация  
- возврат созданной записи  
8) Errors:   
- `400` — invalide входные данные / неподдерживаемый формат файла  
- `401` — пользователь не авторизован  
- `403` — нет прав на редактирование курса  
- `404` — `Course` не найден  
### 4. Выбор источника структуры курса  
1) Назначение: сохранить в `Course` выбранный источник структуры курса перед запуском генерации графа  
2) Метод и путь: PATCH /api/v1/courses/{course_id}/source-structure  
3) Кто вызывает: создатель курса  
4) Права: `access_token` авторизованного пользователя с правами создателя курса  
5) Input: JSON  
```json  
{  
  "source_structure": "auto_generation | course_copy | manual",  "source_course_id": "UUID | null"}  
```  
6) Output: JSON  
```json  
{  
  "course_id": "UUID",  "source_structure": "auto_generation",  "updated_at": "2026-03-23T15:10:00Z"}  
```  
7) sub_actions:   
- проверка `access_token`  
- проверка прав на редактирование `Course`  
- проверка существования `Course`  
- валидация выбранного типа `source_structure`  
- если выбран `course_copy`, проверка `source_course_id`  
- сохранение выбранного значения в `Course.source_structure`  
- возврат обновлённого состояния `Course.source_structure`  
8) Errors:   
- `400` — невалидный `source_structure` или неполный payload  
- `401` — пользователь не авторизован  
- `403` — нет прав на редактирование курса  
- `404` — `Course` не найден / `source_course_id` не найден  
  
### 5. Запустить генерацию (курса)  
1) Назначение: запустить генерацию графа курса через `Django Backend / Orchestrator`  
2) Метод и путь: POST /api/v1/courses/{course_id}/generate-graph  
3) Кто вызывает: создатель курса  
4) Права: `access_token` авторизованного пользователя с правами создателя курса  
5) Input: JSON  
```json  
{  
  "min_topics": 6,  "max_topics": 12}  
```  
6) Output: JSON  
```json  
{  
  "course_id": "UUID",  "graph_gen_status": "processing"}  
```  
7) sub_actions:   
- проверка `access_token`  
- проверка прав на редактирование `Course`  
- проверка существования `Course`  
- проверка, что `source_structure` выбран  
- если `source_structure = manual`, автоматическая генерация не запускается  
- при `auto_generation` backend собирает `content_map` из успешно обработанных `DocumentSection`  
- при `course_copy` backend использует граф и материалы курса-источника  
- backend собирает системный и пользовательский промпты;  
- backend вызывает `POST /ml-llm/generate-course-graph`  
- при успешном ответе сохраняет граф курса и переводит `Course.graph_gen_status` в `done`  
- при ошибке генерации или валидации переводит `Course.graph_gen_status` в `failed`  
8) Errors:   
- `400` — неполный payload генерации  
- `401` — пользователь не авторизован  
- `403` — нет прав на редактирование курса  
- `404` — `Course` не найден  
- `409` — `source_structure = manual` или источник структуры ещё не выбран  
  
### 6. Получить статус генерации (курса)  
1) Назначение: получить текущий статус генерации графа курса  
2) Метод и путь: GET /api/v1/courses/{course_id}/generation-status  
3) Кто вызывает: создатель курса  
4) Права: `access_token` авторизованного пользователя с правами создателя курса  
5) Input: NONE  
6) Output: JSON  
```json  
{  
  "course_id": "UUID",  "graph_gen_status": "not_started | processing | done | failed",  "universal_template_version": "vX.Y | null"}  
```  
7) sub_actions:   
- проверка `access_token`  
- проверка прав на просмотр `Course`  
- чтение `Course.graph_gen_status`  
- если граф успешно сохранён, возврат `universal_template_version`  
8) Errors:   
- `401` — пользователь не авторизован  
- `403` — нет прав на просмотр курса  
- `404` — `Course` не найден  
### 7. Получить черновой граф курса  
1) Назначение: получить текущий черновой граф `Course` для просмотра и ручного редактирования перед подтверждением и запуском `TopicGen`  
2) Метод и путь: GET /api/v1/courses/{course_id}/graph  
3) Кто вызывает: создатель курса  
4) Права: `access_token` авторизованного пользователя с правами на просмотр и редактирование данного `Course`  
5) Input: NONE  
6) Output: JSON  
```json  
{  
  "course_id": "UUID",  
  "graph_gen_status": "done",  
  "topics": [  
    {  
      "id": "UUID",  
      "system_topic_id": "UUID | null",  
      "title": "Описательная статистика",  
      "description": "Базовые меры положения и разброса",  
      "difficulty": 2,  
      "estimated_hours": 6,  
      "order_index": 1,  
      "source": "generated"  
    }  
  ],  
  "topic_ksa": [  
    {  
      "id": "UUID",  
      "topic_id": "UUID",  
      "system_ksa_id": "UUID | null",  
      "name": "Среднее значение",  
      "description": "Понимать смысл среднего значения",  
      "type": "knowledge",  
      "complexity": 2,  
      "order_index": 1,  
      "source": "generated"  
    }  
  ],  
  "topic_dependencies": [  
    {  
      "id": "UUID",  
      "from_topic_id": "UUID",  
      "to_topic_id": "UUID",  
      "type": "required"  
    }  
  ],  
  "competencies": [  
    {  
      "id": "UUID",  
      "topic_id": "UUID | null",  
      "system_competency_id": "UUID | null",  
      "title": "Интерпретировать статистические показатели",  
      "description": "Использовать статистические меры для анализа данных",  
      "source": "generated"  
    }  
  ],  
  "competency_ksa": [  
    {  
      "competency_id": "UUID",  
      "ksa_id": "UUID"  
    }  
  ],  
  "learning_outcomes": [  
    {  
      "id": "UUID",  
      "topic_ksa_id": "UUID",  
      "title": "Объяснить смысл среднего значения",  
      "description": "Студент умеет объяснить, что показывает среднее значение",  
      "bloom_level": "understand",  
      "display_order": 1,  
      "source": "generated"  
    }  
  ],  
  "learning_tasks": [  
    {  
      "id": "UUID",  
      "learning_outcome_id": "UUID",  
      "title": "Решить задачу на вычисление среднего",  
      "description": "Вычислить среднее по выборке",  
      "correct_answer": "42.5",  
      "display_order": 1,  
      "source": "generated"  
    }  
  ]  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на просмотр `Course`
- проверка существования `Course`
- чтение актуального чернового графа курса
- возврат агрегированного состояния графа курса для UI/редактирования
8. Errors:
- `401` — пользователь не авторизован
- `403` — нет прав на просмотр курса
- `404` — `Course` не найден

### 8. Обновить черновой граф курса
1. Назначение: сохранить ручные изменения создателя в черновом графе `Course` перед подтверждением графа и запуском генерации `TopicContent`
2. Метод и путь: PATCH /api/v1/courses/{course_id}/graph
3. Кто вызывает: создатель курса
4. Права: `access_token` авторизованного пользователя с правами на редактирование данного `Course`
5. Input: JSON
```json
{  
  "topics": [  
    {  
      "id": "UUID",  
      "title": "Описательная статистика",  
      "description": "Базовые меры положения и разброса",  
      "difficulty": 2,  
      "estimated_hours": 6,  
      "order_index": 1,  
      "is_active": true  
    }  
  ],  
  "topic_ksa": [  
    {  
      "id": "UUID",  
      "topic_id": "UUID",  
      "name": "Среднее значение",  
      "description": "Понимать смысл среднего значения",  
      "type": "knowledge",  
      "complexity": 2,  
      "order_index": 1  
    }  
  ],  
  "topic_dependencies": [  
    {  
      "id": "UUID",  
      "from_topic_id": "UUID",  
      "to_topic_id": "UUID",  
      "type": "required"  
    }  
  ],  
  "competencies": [  
    {  
      "id": "UUID",  
      "topic_id": "UUID | null",  
      "title": "Интерпретировать статистические показатели",  
      "description": "Использовать статистические меры для анализа данных"  
    }  
  ],  
  "competency_ksa": [  
    {  
      "competency_id": "UUID",  
      "ksa_id": "UUID"  
    }  
  ],  
  "learning_outcomes": [  
    {  
      "id": "UUID",  
      "topic_ksa_id": "UUID",  
      "title": "Объяснить смысл среднего значения",  
      "description": "Студент умеет объяснить, что показывает среднее значение",  
      "bloom_level": "understand",  
      "display_order": 1  
    }  
  ],  
  "learning_tasks": [  
    {  
      "id": "UUID",  
      "learning_outcome_id": "UUID",  
      "title": "Решить задачу на вычисление среднего",  
      "description": "Вычислить среднее по выборке",  
      "correct_answer": "42.5",  
      "display_order": 1  
    }  
  ]  
}
```
6. Output: JSON
```json
{  
  "course_id": "UUID",  
  "updated_at": "2026-04-07T18:10:00Z",  
  "graph_state": "draft_saved"  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на редактирование `Course`
- проверка существования `Course`
- валидация структуры payload
- валидация связности графа:
    - все `TopicDependency` ссылаются на темы данного курса
    - все `CompetencyKSA` ссылаются на существующие `Competency` и `TopicKSA`
    - все `LearningOutcome` ссылаются на существующие `TopicKSA`
    - все `LearningTask` ссылаются на существующие `LearningOutcome`
- сохранение изменений в сущностях графа курса
- обновление `Course.updated_at`
- возврат подтверждения сохранения
7. Errors:
- `400` — невалидный payload / нарушение структуры графа
- `401` — пользователь не авторизован
- `403` — нет прав на редактирование курса
- `404` — `Course` не найден
- `409` — конфликт зависимостей / циклическая структура графа
### 9. Подтвердить граф курса и запустить генерацию `TopicContent`
1. Назначение: подтвердить готовность чернового графа курса и запустить асинхронный pipeline генерации `TopicContent` по темам курса
2. Метод и путь: POST /api/v1/courses/{course_id}/confirm-graph
3. Кто вызывает: создатель курса
4. Права: `access_token` авторизованного пользователя с правами создателя курса
5. Input: NONE
6. Output: JSON
```json
{  
  "course_id": "UUID",  
  "content_gen_status": "processing",  
  "queued": true  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на редактирование `Course`
- проверка существования `Course`
- проверка, что черновой граф курса существует
- постановка Celery-задачи на генерацию `TopicContent` для тем курса
- перевод `Course.content_gen_status` в `processing`
- возврат подтверждения запуска процесса
7. Errors:
- `401` — пользователь не авторизован
- `403` — нет прав на редактирование курса
- `404` — `Course` не найден
- `409` — граф курса ещё не готов к подтверждению
### 10. Получить `TopicContent`
1. Назначение: получить текущий `TopicContent` для конкретной темы курса
2. Метод и путь: GET /api/v1/topics/{topic_id}/content
3. Кто вызывает: создатель курса
4. Права: `access_token` авторизованного пользователя с правами на просмотр курса, к которому относится `Topic`
5. Input: NONE
6. Output: JSON
```json
{  
  "topic_id": "UUID",  
  "topic_content": {  
    "id": "UUID",  
    "source": "manual | generated",  
    "context_level": "topic_docs | course_docs | model_only",  
    "content_text": "string",  
    "status": "draft | verified | rejected",  
    "created_at": "2026-04-07T18:30:00Z",  
    "updated_at": "2026-04-07T18:30:00Z"  
  }  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на просмотр курса
- проверка существования `Topic`
- чтение актуального `TopicContent`
- возврат содержимого темы
7. Errors:
- `401` — пользователь не авторизован
- `403` — нет прав на просмотр курса
- `404` — `Topic` не найден / `TopicContent` не найден
### 11. Создать `TopicContent` вручную
1. Назначение: создать ручной `TopicContent` для темы курса
2. Метод и путь: POST /api/v1/topics/{topic_id}/content
3. Кто вызывает: создатель курса
4. Права: `access_token` авторизованного пользователя с правами на редактирование курса, к которому относится `Topic`
5. Input: JSON
```json
{  
  "content_text": "string",  
  "status": "draft"  
}
```
6. Output: JSON
```json
{  
  "id": "UUID",  
  "topic_id": "UUID",  
  "source": "manual",  
  "context_level": "model_only",  
  "content_text": "string",  
  "status": "draft",  
  "created_at": "2026-04-07T18:35:00Z",  
  "updated_at": "2026-04-07T18:35:00Z"  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на редактирование курса
- проверка существования `Topic`
- валидация `content_text`
- создание `TopicContent`
- установка `source = manual`
- возврат созданной записи
7. Errors:
- `400` — невалидный payload / пустой `content_text`
- `401` — пользователь не авторизован
- `403` — нет прав на редактирование курса
- `404` — `Topic` не найден
- `409` — `TopicContent` для темы уже существует
### 12. Обновить `TopicContent`
1. Назначение: обновить существующий `TopicContent` темы курса
2. Метод и путь: PATCH /api/v1/topics/{topic_id}/content
3. Кто вызывает: создатель курса
4. Права: `access_token` авторизованного пользователя с правами на редактирование курса, к которому относится `Topic`
5. Input: JSON
```json
{  
  "content_text": "string",  
  "status": "draft | verified | rejected"  
}
```
6. Output: JSON
```json
{  
  "id": "UUID",  
  "topic_id": "UUID",  
  "source": "manual | generated",  
  "context_level": "topic_docs | course_docs | model_only",  
  "content_text": "string",  
  "status": "draft | verified | rejected",  
  "updated_at": "2026-04-07T18:40:00Z"  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на редактирование курса
- проверка существования `Topic` и `TopicContent`
- валидация полей обновления
- сохранение обновлённого `TopicContent`
- постановка фоновой задачи на пересборку `TopicContentChunk` и embeddings
- возврат обновлённой записи
7. Errors:
- `400` — невалидный payload
- `401` — пользователь не авторизован
- `403` — нет прав на редактирование курса
- `404` — `Topic` не найден / `TopicContent` не найден
### 13. Удалить `TopicContent`
1. Назначение: удалить `TopicContent` темы курса
2. Метод и путь: DELETE /api/v1/topics/{topic_id}/content
3. Кто вызывает: создатель курса
4. Права: `access_token` авторизованного пользователя с правами на редактирование курса, к которому относится `Topic`
5. Input: NONE
6. Output: JSON
```json
{  
  "topic_id": "UUID",  
  "deleted": true  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на редактирование курса
- проверка существования `Topic` и `TopicContent`
- удаление `TopicContent`
- удаление / инвалидирование связанных `TopicContentChunk`
- возврат результата удаления
7. Errors:
- `401` — пользователь не авторизован
- `403` — нет прав на редактирование курса
- `404` — `Topic` не найден / `TopicContent` не найден
### 14. Запустить генерацию `TopicContent` для темы
1. Назначение: запустить генерацию или перегенерацию `TopicContent` для конкретной темы курса
2. Метод и путь: POST /api/v1/topics/{topic_id}/generate-content
3. Кто вызывает: создатель курса
4. Права: `access_token` авторизованного пользователя с правами на редактирование курса, к которому относится `Topic`
5. Input: JSON
```json
{  
  "force_regenerate": true  
}
```
6. Output: JSON
```json
{  
  "topic_id": "UUID",  
  "queued": true,  
  "content_gen_status": "processing"  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на редактирование курса
- проверка существования `Topic`
- определение `content_source`:
    - `topic_docs`
    - `course_docs`
    - `model_only`
- постановка Celery-задачи на генерацию `TopicContent`
- при необходимости создание / обновление `TopicContentGenerationError`
- возврат подтверждения запуска генерации
7. Errors:
- `400` — невалидный payload
- `401` — пользователь не авторизован
- `403` — нет прав на редактирование курса
- `404` — `Topic` не найден
- `409` — генерация для темы уже запущена
### 15. Получить активные ошибки контентной генерации по курсу
1. Назначение: получить список активных ошибок генерации `TopicContent` по курсу перед публикацией курса или повторной генерацией
2. Метод и путь: GET /api/v1/courses/{course_id}/topic-content-errors
3. Кто вызывает: создатель курса
4. Права: `access_token` авторизованного пользователя с правами на просмотр данного `Course`
5. Input: NONE
6. Output: JSON
```json
{  
  "course_id": "UUID",  
  "errors": [  
    {  
      "id": "UUID",  
      "topic_id": "UUID",  
      "error_type": "llm_timeout | invalid_json | llm_error | validation_failed | no_documents",  
      "error_message": "string",  
      "created_at": "2026-04-07T18:50:00Z",  
      "resolved_at": null  
    }  
  ]  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на просмотр курса
- проверка существования `Course`
- выборка активных `TopicContentGenerationError` по курсу (`resolved_at IS NULL`)
- возврат списка ошибок
7. Errors:
- `401` — пользователь не авторизован
- `403` — нет прав на просмотр курса
- `404` — `Course` не найден
### 16. Опубликовать курс
1. Назначение: опубликовать `Course` после завершения генерации `TopicContent` и проверки наличия активных ошибок
2. Метод и путь: POST /api/v1/courses/{course_id}/publish
3. Кто вызывает: создатель курса
4. Права: `access_token` авторизованного пользователя с правами создателя курса
5. Input: JSON
```json
{  
  "publish_with_errors": false  
}
```
6. Output: JSON
```json
{  
  "course_id": "UUID",  
  "status": "published",  
  "published_at": "2026-04-07T19:00:00Z"  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на публикацию `Course`
- проверка существования `Course`
- чтение `Course.content_gen_status`
- проверка наличия активных `TopicContentGenerationError`
- если `content_gen_status = done`, публикация разрешена
- если `content_gen_status = done_with_errors`, публикация разрешена только при `publish_with_errors = true`
- если `content_gen_status = failed`, публикация запрещена
- если обязательный `TopicContent` отсутствует в manual-режиме, публикация запрещена
- перевод `Course.status` в `published`
- установка `Course.published_at`
- возврат опубликованного состояния курса
7. Errors:
- `400` — невалидный payload
- `401` — пользователь не авторизован
- `403` — нет прав на публикацию курса
- `404` — `Course` не найден
- `409` — курс нельзя публиковать в текущем состоянии
### 17. Сгенерировать диагностический тест
1. Назначение: сгенерировать `DiagnosticTest` по опубликованному курсу для последующей оценки стартового уровня пользователя
2. Метод и путь: POST /api/v1/courses/{course_id}/diagnostic-tests
3. Кто вызывает: система / пользователь перед запуском `TrackGen`
4. Права: `access_token` авторизованного пользователя с правами на просмотр опубликованного `Course`
5. Input: JSON
```json
{  
  "questions_count": 12  
}
```
6. Output: JSON
```json
{  
  "diagnostic_test_id": "UUID",  
  "course_id": "UUID",  
  "status": "draft",  
  "questions_count": 12,  
  "created_at": "2026-04-07T19:10:00Z"  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на просмотр курса
- проверка существования `Course`
- проверка, что курс опубликован
- постановка Celery-задачи на генерацию `DiagnosticTest`
- создание записи `DiagnosticTest`
- возврат идентификатора теста
7. Errors:
- `400` — невалидный payload
- `401` — пользователь не авторизован
- `403` — нет прав на просмотр курса
- `404` — `Course` не найден
- `409` — курс ещё не опубликован

### 18. Создать попытку прохождения диагностического теста
1. Назначение: создать `DiagnosticAttempt` для пользователя перед прохождением диагностического теста
2. Метод и путь: POST /api/v1/diagnostic-tests/{test_id}/attempts
3. Кто вызывает: пользователь
4. Права: `access_token` авторизованного пользователя
5. Input: NONE
6. Output: JSON
```json
{  
  "attempt_id": "UUID",  
  "test_id": "UUID",  
  "user_id": "UUID",  
  "status": "in_progress",  
  "started_at": "2026-04-07T19:20:00Z"  
}
```
7. sub_actions:
- проверка `access_token`
- проверка существования `DiagnosticTest`
- создание `DiagnosticAttempt`
- возврат созданной попытки
7. Errors:
- `401` — пользователь не авторизован
- `404` — `DiagnosticTest` не найден
- `409` — активная попытка уже существует
### 19. Сохранить ответы пользователя на диагностический тест
1. Назначение: сохранить ответы пользователя в `DiagnosticAnswer` в рамках конкретной попытки диагностического тестирования
2. Метод и путь: POST /api/v1/diagnostic-attempts/{attempt_id}/answers
3. Кто вызывает: пользователь
4. Права: `access_token` авторизованного пользователя — владельца `DiagnosticAttempt`
5. Input: JSON
```json
{  
  "answers": [  
    {  
      "question_id": "UUID",  
      "answer_text": "string"  
    }  
  ]  
}
```
6. Output: JSON
```json
{  
  "attempt_id": "UUID",  
  "saved_answers": 3,  
  "updated_at": "2026-04-07T19:25:00Z"  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на `DiagnosticAttempt`
- проверка существования `DiagnosticAttempt`
- валидация списка ответов
- создание / обновление `DiagnosticAnswer`
- возврат количества сохранённых ответов
7. Errors:
- `400` — невалидный payload
- `401` — пользователь не авторизован
- `403` — нет прав на изменение попытки
- `404` — `DiagnosticAttempt` не найден
### 20. Завершить диагностический тест
1. Назначение: завершить попытку диагностического тестирования, запустить оценку ответов и обновление графа знаний пользователя
2. Метод и путь: POST /api/v1/diagnostic-attempts/{attempt_id}/complete
3. Кто вызывает: пользователь
4. Права: `access_token` авторизованного пользователя — владельца `DiagnosticAttempt`
5. Input: NONE
6. Output: JSON
```json
{  
  "attempt_id": "UUID",  
  "status": "completed",  
  "checked": true  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на `DiagnosticAttempt`
- проверка существования `DiagnosticAttempt`
- перевод `DiagnosticAttempt` в завершённое состояние
- запуск проверки `DiagnosticAnswer`
- запуск обновления `UserKSANode`, `UserKnowledgeNode`, `UserCompetencyNode`
- возврат результата завершения попытки
7. Errors:
- `401` — пользователь не авторизован
- `403` — нет прав на завершение попытки
- `404` — `DiagnosticAttempt` не найден
- `409` — попытка уже завершена
### 21. Сохранить анкету пользователя для генерации трека
1. Назначение: сохранить анкету пользователя перед генерацией персонального учебного трека
2. Метод и путь: POST /api/v1/courses/{course_id}/track-questionnaire
3. Кто вызывает: пользователь
4. Права: `access_token` авторизованного пользователя с правами на прохождение данного `Course`
5. Input: JSON
```json
{  
  "goal": "Подготовиться к собеседованию по статистике",  
  "deadline": "2026-06-01",  
  "weekly_hours": 6,  
  "easy_topics": ["Описательная статистика"],  
  "hard_topics": ["Проверка гипотез"],  
  "short_sessions_ok": true,  
  "mastery_signal": "Смогу уверенно решать типовые задачи без подсказок"  
}
```
6. Output: JSON
```json
{  
  "course_id": "UUID",  
  "user_id": "UUID",  
  "questionnaire_saved": true,  
  "updated_at": "2026-04-07T19:35:00Z"  
}
```
7. sub_actions
- проверка `access_token`
- проверка прав пользователя на прохождение курса
- проверка существования `Course`
- валидация анкеты
- сохранение данных анкеты для последующей генерации `UserTrack`
- возврат подтверждения сохранения
7. Errors:
- `400` — невалидный payload
- `401` — пользователь не авторизован
- `403` — нет прав на прохождение курса
- `404` — `Course` не найден
### 22. Сгенерировать персональный учебный трек
1. Назначение: сгенерировать `UserTrack` на основе курса, анкеты пользователя и результатов диагностики
2. Метод и путь: POST /api/v1/courses/{course_id}/tracks
3. Кто вызывает: пользователь
4. Права: `access_token` авторизованного пользователя с правами на прохождение данного `Course`
5. Input: JSON
```json
{  
  "use_diagnostic": true  
}
```
6. Output: JSON
```json
{  
  "track_id": "UUID",  
  "course_id": "UUID",  
  "user_id": "UUID",  
  "status": "draft",  
  "goal": "Подготовиться к собеседованию по статистике",  
  "deadline": "2026-06-01",  
  "weekly_hours": 6  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав пользователя на прохождение курса
- проверка существования `Course`
- проверка публикации курса
- чтение анкеты пользователя
- чтение результатов диагностики, если `use_diagnostic = true`
- инициализация / обновление пользовательского графа знаний
- генерация `UserTrack`, `UserTrackTopic`, `UserTrackTopicDependency`
- возврат созданного трека
7. Errors:
- `400` — невалидный payload
- `401` — пользователь не авторизован
- `403` — нет прав на прохождение курса
- `404` — `Course` не найден
- `409` — курс ещё не опубликован / отсутствуют обязательные входные данные для генерации трека
### 23. Получить персональный учебный трек
1. Назначение: получить сгенерированный `UserTrack` с темами, зависимостями и рекомендациями
2. Метод и путь: GET /api/v1/tracks/{track_id}
3. Кто вызывает: пользователь
4. Права: `access_token` авторизованного пользователя — владельца `UserTrack`
5. Input: NONE
6. Output: JSON
```json
{  
  "track_id": "UUID",  
  "course_id": "UUID",  
  "user_id": "UUID",  
  "status": "draft | active | completed",  
  "goal": "Подготовиться к собеседованию по статистике",  
  "deadline": "2026-06-01",  
  "weekly_hours": 6,  
  "topics": [  
    {  
      "track_topic_id": "UUID",  
      "topic_id": "UUID",  
      "order_index": 1,  
      "status": "not_started",  
      "is_recommended": false,  
      "recommendation_status": "none",  
      "knowledge_node_id": "UUID"  
    }  
  ]  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на просмотр `UserTrack`
- чтение `UserTrack`, `UserTrackTopic`, `UserTrackTopicDependency`
- возврат состояния трека пользователю
7. Errors:
- `401` — пользователь не авторизован
- `403` — нет прав на просмотр трека
- `404` — `UserTrack` не найден
### 24. Принять или отклонить рекомендованную тему в треке
1. Назначение: сохранить решение пользователя по рекомендованной теме в рамках `UserTrack`
2. Метод и путь: PATCH /api/v1/track-topics/{track_topic_id}/recommendation-status
3. Кто вызывает: пользователь
4. Права: `access_token` авторизованного пользователя — владельца соответствующего `UserTrack`
5. Input: JSON
```json
{  
  "recommendation_status": "accepted | rejected"  
}
```
6. Output: JSON
```json
{  
  "track_topic_id": "UUID",  
  "is_recommended": true,  
  "recommendation_status": "accepted"  
}
```
7. sub_actions:
- проверка `access_token`
- проверка прав на изменение `UserTrackTopic`
- проверка существования `UserTrackTopic`
- проверка, что тема действительно является рекомендованной
- сохранение решения пользователя
- возврат обновлённого состояния рекомендованной темы
7. Errors:
- `400` — невалидный `recommendation_status`
- `401` — пользователь не авторизован
- `403` — нет прав на изменение трека
- `404` — `UserTrackTopic` не найден
- `409` — тема не является рекомендованной
---  
## FastAPI ML Inference Services  
### 1. LLM graph course reytration  
1) Назначение: запустить LLM-генерацию чернового графа курса по материалам курса, `UniversalTemplate`, `DisciplineConfig`, дисциплинарному шаблону и метаданным курса, чтобы получить структуру курса: темы, компетенции, KSA, зависимости и оценку трудоёмкости.  
2) Метод и путь: POST /ml-llm/generate-course-graph  
3) Кто вызывает: Celery workers  
4) Права: внутренний сервисный вызов от Django Backend / Orchestrator в FastAPI ML Inference Services  
5) Input: JSON  
```json  
{  
  "course_id": "UUID",
  "discipline_id": "UUID",
  "audience_level": 3,
  "rag_profile_mode": "doc_context",
  "system_prompt": "string",
  "user_prompt": "string",
  "course_metadata": {"title": "Основы математической статистики",    "description": "Курс по базовым методам математической статистики",    "discipline": "Математическая статистика",    "audience_level": 3  },  "content_map": [    {      "document_id": "UUID",      "document_section_id": "UUID",      "title": "Введение",      "content": "..."    }  ],  "generation_constraints": {    "min_topics": 6,    "max_topics": 12  }}  
```  
6) Output: JSON  
```json  
{  
  "course_id": "UUID",  "topics": [    {      "temp_id": "t1",      "system_topic_id": "UUID",      "title": "Описательная статистика",      "description": "Базовые меры положения и разброса",      "difficulty": 2,      "estimated_hours": 6,      "order_index": 1,      "source": "generated"    }  ],  "competencies": [    {      "temp_id": "c1",      "system_competency_id": "UUID",      "course_temp_ref": "UUID",      "title": "Интерпретировать статистические показатели",      "description": "Использовать базовые статистические меры для анализа данных",      "source": "generated"    }  ],  "topic_ksa": [    {      "topic_temp_id": "t1",      "system_ksa_id": "UUID",      "title": "Среднее значение",      "type": "knowledge",      "source": "generated"    }  ],  "competency_ksa": [    {      "competency_temp_id": "c1",      "system_ksa_id": "UUID"    }  ],  "topic_dependencies": [    {      "prerequisite_topic_temp_id": "t1",      "dependent_topic_temp_id": "t2"    }  ]}  
```  
7) sub_actions:   
- `Django Backend / Orchestrator` переводит `Course.graph_gen_status` в `processing`  
- backend формирует `content_map`, system prompt и user prompt  
- backend передаёт в `FastAPI ML Inference Services` метаданные курса, `audience_level`, материалы курса через `RAG profile = doc_context`, дисциплинарные и универсальные ограничения  
- `FastAPI ML Inference Services` выполняет генерацию по `ADDIE + 4C/ID`  
- после генерации `FastAPI ML Inference Services` выполняет валидацию результата в блоке `Verification` (`pydantic validation` + `LLM-as-judge`)  
- только валидный результат возвращается в `Django Backend / Orchestrator`  
- при ошибке валидации сервис возвращает ошибку, а не невалидный граф  
- при вызове создаётся trace в `Langfuse`  
7) Errors:   
- `400` — invalide входные данные / неполный payload генерации  
- `408` — timeout генерации (`120 sec`)  
- `422` — LLM вернула невалидный JSON / структура ответа не проходит schema validation  
- `500` — внутренняя ошибка ML-сервиса  
- `502` — Django Backend / Orchestrator не получил корректный ответ от FastAPI ML Inference Services  
### 2. Internal ML endpoint embedding  
1) Назначение: принять список текстов секций документов и вернуть эмбеддинги для последующей записи в `pgvector`  
2) Метод и путь: POST /ml/embedding  
3) Кто вызывает: Celery worker  
4) Права: внутренний сервисный вызов от worker-а в `FastAPI ML Inference Services`  
5) Input: JSON  
``` json{  
  "items": [    {      "document_id": "UUID",      "document_section_id": "UUID",      "content": "..."    }  ]}  
```  
6) Output: JSON  
``` json{  
  "items": [    {      "document_id": "UUID",      "document_section_id": "UUID",      "embedding": [0.123, -0.456, 0.789]    }  ]}  
```  
7) sub_actions:   
- проверка корректности payload  
- получение эмбеддингов для переданных текстов  
- возврат embeddings в worker  
- worker сохраняет embedded-представления в `pgvector`  
8) Errors:   
- `400` — невалидный payload  
- `500` — внутренняя ошибка embedding-сервиса
### 3. Internal ML endpoint topic content generation
1. Назначение: сгенерировать `TopicContent` по теме курса с использованием `topic_docs`, `course_docs` или `model_only`
2. Метод и путь: POST /ml-llm/generate-topic-content
3. Кто вызывает: Celery worker
4. Права: внутренний сервисный вызов от `Django Backend / Orchestrator` в `FastAPI ML Inference Services`
5. Input: JSON
```json
{  
  "course_id": "UUID",  
  "topic_id": "UUID",  
  "content_source": "topic_docs | course_docs | model_only",  
  "system_prompt": "string",  
  "user_prompt": "string",  
  "topic_metadata": {  
    "title": "Описательная статистика",  
    "description": "Базовые меры положения и разброса",  
    "difficulty": 2,  
    "estimated_hours": 6  
  },  
  "rag_context": [  
    {  
      "document_id": "UUID",  
      "document_section_id": "UUID",  
      "content": "..."  
    }  
  ]  
}
```
6. Output: JSON
```json
{  
  "course_id": "UUID",  
  "topic_id": "UUID",  
  "source": "generated",  
  "context_level": "topic_docs | course_docs | model_only",  
  "content_text": "string",  
  "status": "draft"  
}
```
7. sub_actions:
- проверка корректности payload
- выбор режима генерации по `content_source`
- генерация `TopicContent`
- базовая валидация структуры ответа
- возврат результата в worker
- при вызове создаётся trace в `Langfuse`
7. Errors:
- `400` — невалидный payload
- `408` — timeout генерации
- `422` — невалидная структура ответа / невалидный JSON
- `500` — внутренняя ошибка ML-сервиса

### 4. Internal ML endpoint diagnostic test generation
1. Назначение: сгенерировать `DiagnosticTest` и набор `DiagnosticQuestion` по опубликованному курсу
2. Метод и путь: POST /ml-llm/generate-diagnostic-test
3. Кто вызывает: Celery worker
4. Права: внутренний сервисный вызов от `Django Backend / Orchestrator` в `FastAPI ML Inference Services`
5. Input: JSON
```json
{  
  "course_id": "UUID",  
  "questions_count": 12,  
  "course_graph": {  
    "topics": [  
      {  
        "topic_id": "UUID",  
        "title": "Описательная статистика"  
      }  
    ],  
    "topic_ksa": [  
      {  
        "ksa_id": "UUID",  
        "topic_id": "UUID",  
        "name": "Среднее значение"  
      }  
    ]  
  },  
  "rag_context": [  
    {  
      "document_id": "UUID",  
      "document_section_id": "UUID",  
      "content": "..."  
    }  
  ]  
}
```
6. Output: JSON
```json
{  
  "course_id": "UUID",  
  "questions": [  
    {  
      "temp_id": "dq1",  
      "topic_id": "UUID",  
      "competency_id": "UUID | null",  
      "primary_ksa_id": "UUID | null",  
      "question_text": "Что показывает среднее значение выборки?",  
      "question_type": "open"  
    }  
  ]  
}
```
7. sub_actions:
- проверка корректности payload
- генерация диагностических вопросов по графу курса и материалам курса
- базовая валидация структуры ответа
- возврат результата в worker
- при вызове создаётся trace в `Langfuse`
7. Errors:
- `400` — невалидный payload
- `408` — timeout генерации
- `422` — невалидная структура ответа / невалидный JSON
- `500` — внутренняя ошибка ML-сервиса