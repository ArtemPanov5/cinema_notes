1) Утвердить общий методологический алгоритм.
	Acceptance criteria:
	1. есть один документ/страница с алгоритмом
	2. нет противоречий между ТЗ, доменной диаграммой и high-level схемой
---
2) Создать стартовую структуру docker-compose для удобной разработки.
	Description:
	1. Django Backend
	2. FastAPI ML
	3. PostgreSQL + pgvector
	4. Celery
	5. Redis 
	Acceptance criteria:
	- есть `docker-compose.yml`  
	- проект поднимается локально по README  
	- backend и ml-service проходят healthcheck  
	- Postgres и pgvector доступны из контейнеров
	- все сервисы поднимаются одной командой  
	- backend и ml-service видят PostgreSQL  
	- Celery подключается к Redis  
	- pgvector включён и готов к использованию 
---
3) Реализация Django Backend / Orchestrator модуля.
	Description:
	1. реализованы базовые domain/API flows:  
	 - загрузка материалов курса
	 - создание и публикация курса
	 - запуск диагностики
	 - генерация трека
	 - lesson step orchestration  
	2. backend пишет interaction logs и decision logs в PostgreSQL
	3. backend вызывает внутренние ML endpoints
	4. поддержана межсервисная auth через API key
	Acceptance criteria:
	- есть каркас Django проекта
	- реализованы основные endpoint groups (надо согласовать)
	- orchestration lesson loop работает
	- interaction events пишутся в OLTP DB
---
4) Design of OLTP DB schema. (PostgreSQL and pgvector).
	Description:
	1. Реализация PostgreSQL DB с основными таблицами для обслуживания Django Backend/Orchestrator, Sandbox, Acync Workers, а также записи event_logs
	2. Реализация pgvector DB для обслуживания RAG
	Acceptance criteria:
	- хранение chunks, sections, metadata и embeddings
	- фильтрация по course/topic/document
	- pgvector подключён
	- есть таблицы для chunks/embeddings
	- retrieval по embeddings и metadata работает
---
5) Реализация FastAPI ML Inference Services модуля
	**5.1** Подмодуль Content с интерфейсами/профилями RAG
	Result: RAG gateway для Этапа 1
	Description:
	- реализованы профили:
	  - doc_context
	  - inline_explanation
	  - test_generation
	  - diagnostic_test
	- retrieval идёт через pgvector
	- generation опирается на материалы курса
	Acceptance criteria:
	- реализованы профили: doc_context, inline_explanation, test_generation, diagnostic_test
	- retrieval идёт через pgvector
	- есть внутренние endpoints для retrieval/generation
	- generation можно вызвать из backend
	- каждый профиль имеет свой input contract (форматирование для ввода и вывода)
	- retrieval ограничивается metadata filters (ограничения по данным при работе RAG)
	**5.2** Подмодуль Knowledge State Provider c интерфейсом для BKT
	Result: стабильный интерфейс knowledge state для Этапа 1
	Description:
	- реализован BKT update после student response
	- сохраняются before/after значения
	- интерфейс не должен мешать будущей замене на DKT
	**Acceptance criteria:**
	- есть endpoint/update contract для BKT
	- обновление работает по topic и primary_ksa
	- результат сохраняется в БД
	- интерфейс задокументирован
	**5.3** Подмодуль Action Policy Provider с интерфейсами для rule-based
	Result: rule-based выбор следующего педагогического действия
	Description:
	- на вход подаются mastery, история урока, режим, last action
	- на выходе одно из 8 педагогических действий
	- структура совместима с будущей заменой на DQN
	Acceptance criteria:
	- есть endpoint/contract для policy decision
	- policy возвращает корректное действие
	- decision логируется
	- state snapshot сохраняется
	**5.4** Подмодуль Verification с интерфейсами для LLM-as-judge и pydantic validation
	Result: финальная проверка generated content
	Description:
	- schema validation обязательна
	- для тестов/задач поддержан verification flow
	- rejected content не уходит пользователю без fallback
	Acceptance criteria:
	- есть pydantic validation
	- есть verification endpoint/flow
	- verification result сохраняется
	- ошибки логируются явно
---
6) Подключение логирования LLM запросов через Langfuse.
	Result: трассировка LLM-вызовов в generation/verification
	Description:
	- логируются prompts, responses, errors, latency, tockens utilisation
	Acceptance criteria:
	- Langfuse подключён к ML service
	- видны traces для generation и verification
	- можно отследить запрос по lesson flow
---
7) Реализация модуля Data Export для ежедневной выгрузки данных из OLTP PostgreSQL DB.
	Result: можно отследить запрос по lesson flow
	Description:
	- экспортирует interaction logs + decision logs
	- вычисление значений ... (тут надо дополнить каких именно) и формирование dataset
	- экспорт не блокирует online path
	Acceptance criteria:
	- есть export job
	- есть export view / query
	- `event_index_export = ROW_NUMBER() OVER (PARTITION BY session_id ORDER BY event_timestamp, interaction_id)` (это нужно уточнить)
	- выгрузка версионируется
---
8) Реализовать Redis для кеширования сессий и для Celery.
	Result: Redis используется как ускоряющий слой, не как source of truth для critical logs
	Description:
	- Redis используется для:
		- Celery broker
		- cache
		- temporary session state
	- interaction events не зависят от Redis и падение Redis не должно ломать запись этих событий
	Acceptance criteria:
	- Celery подключён к Redis
	- есть минимум один рабочий async job
	- backend может использовать Redis cache/session helpers
---
9) Cогласовать API и git workflow.
	Result: формальное соглашение о процессе реализации
	Description:
	- согласовать endpoint groups
	- согласовать interface groups
	- согласовать репозитории, ветки, MR, approve, CI
	Acceptance criteria:
	- краткий API outline
	- краткий список репозиториев
	- краткий правила по наименованиям в коде, веткам в репозитории, merge requests, CI/CD
---


1. [API] Implement POST /api/v1/disciplines endpoint  
2. [API] Add serializer and input validation  
3. [SystemPreReq] Implement create_discipline service logic  
4. [Auth] Add authentication and permission checks  
5. [API] Implement error handling (400, 401, 403, 409)  
6. [DB] Ensure FK availability in DisciplineConfig/SystemGraph tables  
7. [SystemPreReq] Verify integration (Discipline → SystemGraph usage)
   
   
#### Set up:

set up:
```PowerShell
docker compose up -d
```
DB migration:
```PowerShell
docker compose exec django-api python manage.py migrate
```
Django check:
```PowerShell
docker compose exec django-api python manage.py check
```
Tests:
```PowerShell
docker compose exec django-api python manage.py test courses -v 2
```
Import DB to .sql:
```PowerShell
docker compose exec postgres pg_dump -U nastavnik -d nastavnik --schema-only --no-owner --no-privileges > infra/db/full_schema.sql
```
#### Manual tests:

```PowerShell
$BASE = "http://localhost:8000"
$TOKEN = "dev-methodologist-token"
$FILE = "C:\Users\artem\WorkProjects\Nastavnik\backlog\backlog.pdf"
```
Discipline API check
```PowerShell
$disciplineBody = @{
  name = "Backlog Discipline"
  description = "Discipline for manual upload test"
} | ConvertTo-Json

$disciplineResponse = Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/disciplines" `
  -Headers @{ Authorization = "Bearer $TOKEN" } `
  -ContentType "application/json" `
  -Body $disciplineBody

$disciplineResponse | Format-List
$DISCIPLINE_ID = $disciplineResponse.id
$DISCIPLINE_ID
```

```PowerShell
$TOKEN = "dev-course-creator-token"
```
Course API check
```PowerShell
$courseBody = @{
  title = "Backlog Copy Upload Test Course"
  discipline_id = $DISCIPLINE_ID
  description = "Course created to test coice of course source"
  target_audience = "Internal testing"
  audience_level = 3
} | ConvertTo-Json

$courseResponse = Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/courses" `
  -Headers @{ Authorization = "Bearer $TOKEN" } `
  -ContentType "application/json" `
  -Body $courseBody

$courseResponse | Format-List
$COURSE_ID = $courseResponse.id
$COURSE_ID
```
Uploading Document check
```PowerShell
$docJson = curl.exe -s -X POST "$BASE/api/v1/courses/$COURSE_ID/documents" `
  -H "Authorization: Bearer $TOKEN" `
  -F "title=backlog" `
  -F "file=@$FILE;type=application/pdf"

$docResponse = $docJson | ConvertFrom-Json
$docResponse | Format-List

$DOCUMENT_ID = $docResponse.id
$STORAGE_KEY = $docResponse.storage_key

$DOCUMENT_ID
$STORAGE_KEY
```

Discipline:
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Discipline; import json; print(json.dumps(list(Discipline.objects.filter(id='$DISCIPLINE_ID').values('id','name','description','created_at','updated_at')), default=str, ensure_ascii=False, indent=2))"
```
Course:
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Course; import json; print(json.dumps(list(Course.objects.filter(id='$COURSE_ID').values('id','owner_id','discipline_id','title','description','target_audience','audience_level','status','graph_gen_status','source_structure','created_at','updated_at')), default=str, ensure_ascii=False, indent=2))"
```
Documents of this course:
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Document; import json; print(json.dumps(list(Document.objects.filter(course_id='$COURSE_ID').values('id','course_id','topic_id','uploaded_by_id','title','storage_key','file_type','parse_status','created_at','updated_at')), default=str, ensure_ascii=False, indent=2))"
```
All documents:
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Document; import json; print(json.dumps(list(Document.objects.all().values('id','course_id','title','storage_key','file_type','parse_status')), default=str, ensure_ascii=False, indent=2))"
```
Download document from S3:
```PowerShell
docker compose exec django-api python manage.py shell -c "
import boto3
from django.conf import settings

s3 = boto3.client(
    's3',
    endpoint_url=settings.S3_ENDPOINT_URL,
    aws_access_key_id=settings.S3_ACCESS_KEY_ID,
    aws_secret_access_key=settings.S3_SECRET_ACCESS_KEY,
)
s3.download_file(settings.S3_BUCKET_NAME, '$STORAGE_KEY', '/tmp/backlog_downloaded.pdf')
print('/tmp/backlog_downloaded.pdf downloaded')
"
```

```PowerShell
docker compose cp django-api:/tmp/backlog_downloaded.pdf <downloading path> (example: C:\Users\artem\Downloads\backlog_downloaded.pdf)
```

---


прописать $filePath
```PowerShell
$filePath = "C:\Users\artem\WorkProjects\Nastavnik\backlog\test_file_prob_and_stat.docx"
```

загрузить документ по $filePath
```PowerShell
curl.exe -X POST "http://localhost:8000/api/v1/courses/$COURSE_ID/documents" -H "Authorization: Bearer dev-course-creator-token" -F "title=coursework.doc" -F "file=@$filePath"
```

посмотреть все доки для какого-то курса
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Document; import json; print(json.dumps(list(Document.objects.filter(courses__id='$COURSE_ID').distinct().values('id','topic_id','uploaded_by_id','title','storage_key','file_type','parse_status','created_at','updated_at')), default=str, ensure_ascii=False, indent=2))"
```

посмотреть contentmap для курса:
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.contentmap import build_content_map; import json; cm = build_content_map(course_id='$COURSE_ID'); print(json.dumps(cm.to_dict(), ensure_ascii=False, indent=2))"
```

посмотреть все DocumentSection для документа:
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import DocumentSection; import json; qs=DocumentSection.objects.filter(document_id='0cdbfd85-8b18-4702-93c1-549ec0afa802').order_by('order_index'); print('count=', qs.count()); print(json.dumps(list(qs.values('order_index','content')[:3]), ensure_ascii=False, indent=2))"
```

перезапуск обработки документа:
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Document, DocumentSection, DocumentParseStatus; from courses.services import enqueue_document_processing; doc=Document.objects.get(id='cb40d6c7-2738-4dd2-a1ba-b11f148c7fef'); DocumentSection.objects.filter(document=doc).delete(); doc.parse_status=DocumentParseStatus.PENDING; doc.save(update_fields=['parse_status','updated_at']); enqueue_document_processing(document_id=doc.id); print(doc.id)"
```

logs of worker
```
docker compose logs -f celery-worker
```

Удалить контейнеры и очистить тома
```
docker compose down -v
```

```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Course, UniversalTemplate; from courses.contentmap import build_content_map; from courses.prompts import build_course_generation_user_prompt, get_discipline_config; course = Course.objects.first(); content_map = build_content_map(course_id=course.id); discipline_config = get_discipline_config(discipline_id=course.discipline_id); prompt = build_course_generation_user_prompt(course=course, content_map=content_map, min_topics=6, max_topics=12, discipline_config=discipline_config); print(prompt)"
```

```PowerShell
curl.exe -X POST "http://localhost:8000/api/v1/courses/50f4b5ad-57be-45e3-a049-cbcc8d0cda18/generate-graph" `
-H "Content-Type: application/json" `
-H "Authorization: Bearer test-course-creator-token" `  
-d "{\"min_topics\": 6, \"max_topics\": 12}"
```

```PowerShell
$courseId = "50f4b5ad-57be-45e3-a049-cbcc8d0cda18"
```

генерация графа курса
```PowerShell
$courseId = $COURSE_ID
$body = @{
  min_topics = 6
  max_topics = 12
} | ConvertTo-Json

Invoke-RestMethod `
  -Method POST `
  -Uri "http://localhost:8000/api/v1/courses/$courseId/generate-graph" `
  -Headers @{ Authorization = "Bearer $TOKEN" } `
  -ContentType "application/json" `
  -Body $body
```

получить все дисциплины
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Discipline; import json; print(json.dumps(list(Discipline.objects.all().values()), default=str, ensure_ascii=False, indent=2))"
```

получить все курсы
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Course; import json; print(json.dumps(list(Course.objects.all().values()), default=str, ensure_ascii=False, indent=2))"
```

посмотреть статус курса и что создалось
```PowerShell
docker compose exec django-api python manage.py shell -c "  
from courses.models import Course, Topic, TopicKSA, LearningOutcome, LearningTask  
import json  
  
course = Course.objects.get(id='$COURSE_ID')  
  
print(json.dumps({  
'course_id': str(course.id),  
'graph_gen_status': course.graph_gen_status,  
'topics_count': Topic.objects.filter(course=course).count(),  
'topic_ksa_count': TopicKSA.objects.filter(topic__course=course).count(),  
'learning_outcomes_count': LearningOutcome.objects.filter(ksa__topic__course=course).count(),  
'learning_tasks_count': LearningTask.objects.filter(outcome__ksa__topic__course=course).count(),  
}, ensure_ascii=False, indent=2))  
"
```

выставить `source_structure = auto_generation`:
```PowerShell
$body = @{
  source_structure = "auto_generation"
  source_course_id = $null
} | ConvertTo-Json

Invoke-RestMethod `
  -Method PATCH `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/source-structure" `
  -Headers @{ Authorization = "Bearer $TOKEN" } `
  -ContentType "application/json" `
  -Body $body
```

посмотреть граф курса после генерации:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import Course, Topic, TopicKSA, LearningOutcome, LearningTask
import json

course = Course.objects.get(id='$COURSE_ID')
learning_tasks = list(
    LearningTask.objects.filter(outcome__ksa__topic__course_id=course.id)
    .order_by(
        'outcome__ksa__topic__order_index',
        'outcome__ksa__order_index',
        'outcome__display_order',
        'display_order',
    )
    .values(
        'id',
        'outcome_id',
        'title',
        'description',
        'correct_answer',
        'display_order',
        'source',
        'created_at',
        'updated_at',
    )
)
payload = {
    'course': {
        'id': str(course.id),
        'title': course.title,
        'graph_gen_status': course.graph_gen_status,
        'source_structure': course.source_structure,
        'discipline_id': str(course.discipline_id),
        'audience_level': course.audience_level,
        'universal_template_id': str(course.universal_template_id),
        'created_at': str(course.created_at),
        'updated_at': str(course.updated_at),
    },
    'topics': list(
        Topic.objects.filter(course=course)
        .order_by('order_index')
        .values(
            'id', 'title', 'description', 'difficulty', 'estimated_hours',
            'order_index', 'source', 'system_topic_id', 'created_at', 'updated_at',
        )
    ),
    'topic_ksa': list(
        TopicKSA.objects.filter(topic__course=course)
        .order_by('topic__order_index', 'order_index')
        .values(
            'id', 'topic_id', 'name', 'description', 'type', 'complexity',
            'is_critical', 'order_index', 'source', 'system_ksa_id',
            'created_at', 'updated_at',
        )
    ),
    'learning_outcomes': list(
        LearningOutcome.objects.filter(ksa__topic__course=course)
        .order_by('ksa__topic__order_index', 'ksa__order_index', 'display_order')
        .values(
            'id', 'ksa_id', 'title', 'description', 'bloom_level',
            'display_order', 'source', 'created_at', 'updated_at',
        )
    ),
    'learning_tasks': learning_tasks,   # добавленное поле
}

print(json.dumps(payload, ensure_ascii=False, indent=2, default=str))
"
```

загрузить в DB Universal template:
```PowerShell
docker compose exec django-api python manage.py loaddata courses/fixtures/universal_template.json
```

загрузить в DB SystemConfig:
```PowerShell
docker compose exec django-api python manage.py loaddata courses/fixtures/system_config.json
```
импорт текущей схемы DB
```PowerShell
docker compose exec postgres pg_dump -U nastavnik -d nastavnik --schema-only --no-owner --no-privileges > infra/db/full_schema.sql
```


BigData:
```
cd /home/team22/project/bigdata-final-project

python3 -m venv venv
source venv/bin/activate

python -m pip install --upgrade pip
pip install psycopg2-binary

mkdir -p secrets

printf '%s\n' 'e8fsA1MvaOEkktaZ' > secrets/.psql.pass
printf '%s\n' 'e8fsA1MvaOEkktaZ' > secrets/.hive.pass

chmod 600 secrets/.psql.pass secrets/.hive.pass

ls -la secrets
```

---

System Requirements

**UC-1 (Просмотр карточки товара и рекомендованных продуктов)**  
- **RQ-UC1-01 (User):** Пользователь видит на странице товара название, изображение, цену, рейтинг и описание.  
- **RQ-UC1-02 (User):** Под карточкой товара отображается список рекомендуемых продуктов в виде кликабельных карточек.  
- **RQ-UC1-03 (User):** При отсутствии подходящих рекомендаций выводится сообщение «Нет рекомендаций».  
- **RQ-UC1-04 (System):** При загрузке страницы система запрашивает данные товара (название, изображение, цена, рейтинг, описание) из центральной базы.  
- **RQ-UC1-05 (System):** Система формирует список рекомендаций на основе категорий и поведения пользователя, отбирая товары с рейтингом ≥ 4.  
- **RQ-UC1-06 (System):** Карточка товара не показывается, если остаток на складе равен нулю.

**UC-2 (Добавление рекомендованных продуктов в корзину)**  
- **RQ-UC2-01 (User):** Пользователь может добавить любой доступный товар в корзину.  
- **RQ-UC2-02 (User):** После добавления пользователь мгновенно получает визуальное подтверждение (например, всплывающее уведомление).  
- **RQ-UC2-03 (User):** Неаутентифицированный пользователь при попытке добавить товар перенаправляется на страницу входа.  
- **RQ-UC2-04 (User):** Счётчик товаров в иконке корзины (в шапке) увеличивается на единицу (или обновляется количество) после успешного добавления.  
- **RQ-UC2-05 (System):** Перед добавлением система проверяет статус аутентификации пользователя и доступность товара на складе.  
- **RQ-UC2-06 (System):** Товары корзины хранятся в сессии и связываются с учётной записью после входа.  
- **RQ-UC2-07 (System):** При повторном добавлении того же товара его количество в корзине увеличивается, а не создаётся дублирующая позиция.  
- **RQ-UC2-08 (System):** Установлен лимит — не более 10 единиц одного товара; при превышении выводится ошибка.

**UC-3 (Оформление и размещение заказа)**  
- **RQ-UC3-01 (User):** Кнопка «Перейти к оформлению» доступна, когда в корзине есть хотя бы один товар.  
- **RQ-UC3-02 (User):** Перед подтверждением покупки пользователь проверяет состав заказа, адрес и способ оплаты.  
- **RQ-UC3-03 (User):** После успешного заказа отображается страница подтверждения с номером заказа и предполагаемой датой доставки.  
- **RQ-UC3-04 (System):** Если не выбрана доставка на дом, система проверяет, что указанный адрес соответствует реальному пункту самовывоза.  
- **RQ-UC3-05 (System):** Итоговая цена вычисляется по формуле: сумма (цена × количество) + стоимость доставки; при самовывозе плата за доставку = 0.  
- **RQ-UC3-06 (System):** Система инициирует платёж через Платёжный шлюз и при успешной транзакции создаёт запись заказа в Системе управления заказами.