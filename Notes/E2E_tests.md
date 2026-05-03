SetUp:
```PowerShell
docker compose up -d
```

```PowerShell
docker compose exec django-api python manage.py migrate
```

```PowerShell
docker compose exec django-api python manage.py test courses -v 2
```

```PowerShell
docker compose exec ml-llm python -m pytest tests/ -v
```

```PowerShell
docker compose exec django-api python manage.py loaddata courses/fixtures/universal_template.json
```

```PowerShell
docker compose exec django-api python manage.py loaddata courses/fixtures/system_config.json
```

системные переменные:
```PowerShell
$BASE = "http://localhost:8000"
$METHOD_TOKEN = "dev-methodologist-token"
$COURSE_TOKEN = "dev-course-creator-token"
$FILE = "C:\Users\artem\WorkProjects\Nastavnik\backlog\test_file_prob_and_stat.docx"
```

создание дисциплины:
```PowerShell
$disciplineBody = @{  
name = "Probability Theory"  
description = "Discipline for probability theory and conditional probability course generation"  
} | ConvertTo-Json
```

```PowerShell
$disciplineResponse = Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/disciplines" `
  -Headers @{ Authorization = "Bearer $METHOD_TOKEN" } `
  -ContentType "application/json" `
  -Body $disciplineBody
```

```PowerShell
$disciplineResponse | Format-List  
$DISCIPLINE_ID = $disciplineResponse.id  
$DISCIPLINE_ID
```

создание курса:
```PowerShell
$courseBody = @{
  title = "Conditional Probability and Independence"
  discipline_id = $DISCIPLINE_ID
  description = "Course for E2E graph generation based on a lecture about conditional probability and independence"
  target_audience = "Bachelor students studying introductory probability theory"
  audience_level = 3
  main_topic = "Conditional probability and independence of events"
  goals = @(
    "Understand the meaning of conditional probability",
    "Apply the formula of conditional probability",
    "Distinguish independent and dependent events"
  )
  desired_competencies = @(
    "Interpret conditional probability in simple probabilistic settings",
    "Solve introductory problems on independent events"
  )
} | ConvertTo-Json -Depth 5
```

```PowerShell
$courseResponse = Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/courses" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json" `
  -Body $courseBody
```

```PowerShell
$courseResponse | Format-List
$COURSE_ID = $courseResponse.id
$COURSE_ID
```

загрузка документа:
```PowerShell
$docJson = curl.exe -s -X POST "$BASE/api/v1/courses/$COURSE_ID/documents" `
  -H "Authorization: Bearer $COURSE_TOKEN" `
  -F "title=conditional_probability_lecture" `
  -F "file=@$FILE;type=application/vnd.openxmlformats-officedocument.wordprocessingml.document"
```

```PowerShell
$docResponse = $docJson | ConvertFrom-Json
$docResponse | Format-List
```

```PowerShell
$DOCUMENT_ID = $docResponse.id
$STORAGE_KEY = $docResponse.storage_key

$DOCUMENT_ID
$STORAGE_KEY
```

```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.models import Document; import json; print(json.dumps(list(Document.objects.filter(courses__id='$COURSE_ID').distinct().values('id','title','parse_status','created_at','updated_at')), default=str, ensure_ascii=False, indent=2))"
```
worker logs:
```PowerShell
docker compose logs -f celery-worker
```
contentmap:
```PowerShell
docker compose exec django-api python manage.py shell -c "from courses.contentmap import build_content_map; import json; cm = build_content_map(course_id='$COURSE_ID'); print(json.dumps(cm.to_dict(), ensure_ascii=False, indent=2))"
```
проверка query-embedding endpoint:
```PowerShell
$body = @{
  text = "Conditional probability and independence of events"
} | ConvertTo-Json
```

```PowerShell
Invoke-RestMethod `
  -Method POST `
  -Uri "http://localhost:8002/ml/query-embedding" `
  -ContentType "application/json" `
  -Body $body
```


seed для системного графа:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import (
    Discipline,
    EntitySource,
    SystemTopic,
    SystemTopicDependency,
    SystemKSA,
    SystemTopicKSA,
    SystemCompetency,
    SystemCompetencyKSA,
    SystemDisciplineTopic,
    SystemDisciplineCompetency,
)
from courses.services import call_ml_llm_query_embedding

discipline = Discipline.objects.get(id='$DISCIPLINE_ID')

t1_text = 'Probability space and events. Basic notions of probability space, sample space and events.'
t2_text = 'Conditional probability and independence of events. Definition of conditional probability, dice example, and event independence.'
t3_text = 'Bayes theorem. Bayes formula and updating probability with additional information.'

t1 = SystemTopic.objects.create(
    name='Probability space and events',
    description='Basic notions of probability space, sample space and events.',
    embedding=call_ml_llm_query_embedding(text=t1_text),
    source=EntitySource.MANUAL,
    created_by=None,
)
t2 = SystemTopic.objects.create(
    name='Conditional probability and independence',
    description='Definition of conditional probability, examples and event independence.',
    embedding=call_ml_llm_query_embedding(text=t2_text),
    source=EntitySource.MANUAL,
    created_by=None,
)
t3 = SystemTopic.objects.create(
    name='Bayes theorem',
    description='Bayes formula and updating probability with additional information.',
    embedding=call_ml_llm_query_embedding(text=t3_text),
    source=EntitySource.MANUAL,
    created_by=None,
)

SystemDisciplineTopic.objects.create(discipline=discipline, system_topic=t1)
SystemDisciplineTopic.objects.create(discipline=discipline, system_topic=t2)
SystemDisciplineTopic.objects.create(discipline=discipline, system_topic=t3)

SystemTopicDependency.objects.create(from_topic=t1, to_topic=t2, type='required')
SystemTopicDependency.objects.create(from_topic=t2, to_topic=t3, type='recommended')

k1_text = 'Explain probability space and events. Understand sample space, event and probability measure.'
k2_text = 'Compute conditional probability. Apply the formula P(A|B)=P(A∩B)/P(B) in simple problems.'
k3_text = 'Distinguish independent events. Recognize and explain when events are independent.'
k4_text = 'Apply Bayes theorem. Use Bayes formula to update probability estimates.'

k1 = SystemKSA.objects.create(
    name='Explain probability space and events',
    description='Understand the notions of sample space, event and probability measure.',
    type='knowledge',
    order_index=1,
    complexity=1,
    is_critical=False,
    source=EntitySource.MANUAL,
    embedding=call_ml_llm_query_embedding(text=k1_text),
)
k2 = SystemKSA.objects.create(
    name='Compute conditional probability',
    description='Apply the formula P(A|B)=P(A∩B)/P(B) in simple problems.',
    type='skill',
    order_index=1,
    complexity=2,
    is_critical=True,
    source=EntitySource.MANUAL,
    embedding=call_ml_llm_query_embedding(text=k2_text),
)
k3 = SystemKSA.objects.create(
    name='Distinguish independent events',
    description='Recognize and explain when events are independent.',
    type='knowledge',
    order_index=2,
    complexity=2,
    is_critical=False,
    source=EntitySource.MANUAL,
    embedding=call_ml_llm_query_embedding(text=k3_text),
)
k4 = SystemKSA.objects.create(
    name='Apply Bayes theorem',
    description='Use Bayes formula to update probability estimates.',
    type='skill',
    order_index=1,
    complexity=3,
    is_critical=False,
    source=EntitySource.MANUAL,
    embedding=call_ml_llm_query_embedding(text=k4_text),
)

SystemTopicKSA.objects.create(system_topic=t1, system_ksa=k1)
SystemTopicKSA.objects.create(system_topic=t2, system_ksa=k2)
SystemTopicKSA.objects.create(system_topic=t2, system_ksa=k3)
SystemTopicKSA.objects.create(system_topic=t3, system_ksa=k4)

c1 = SystemCompetency.objects.create(
    name='Reason about conditional probability',
    description='Interpret and solve elementary tasks on conditional probability.',
    source=EntitySource.MANUAL,
    created_by=None,
)
c2 = SystemCompetency.objects.create(
    name='Reason about independence and Bayesian update',
    description='Use event independence and Bayes theorem in introductory tasks.',
    source=EntitySource.MANUAL,
    created_by=None,
)

SystemDisciplineCompetency.objects.create(discipline=discipline, system_competency=c1)
SystemDisciplineCompetency.objects.create(discipline=discipline, system_competency=c2)

SystemCompetencyKSA.objects.create(system_competency=c1, system_ksa=k2)
SystemCompetencyKSA.objects.create(system_competency=c1, system_ksa=k3)
SystemCompetencyKSA.objects.create(system_competency=c2, system_ksa=k3)
SystemCompetencyKSA.objects.create(system_competency=c2, system_ksa=k4)

print('system graph seeded with real embeddings')
"
```

проверка ближайших SystemTopic для main_topic:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import Course, SystemTopic
from courses.services import call_ml_llm_query_embedding
from pgvector.django import CosineDistance
import json

course = Course.objects.get(id='$COURSE_ID')
query_embedding = call_ml_llm_query_embedding(text=course.main_topic)

rows = list(
    SystemTopic.objects.annotate(distance=CosineDistance('embedding', query_embedding))
    .order_by('distance', 'created_at', 'id')
    .values('name', 'distance')[:5]
)

for row in rows:
    row['similarity'] = 1.0 - float(row['distance'])

print(json.dumps({
    'main_topic': course.main_topic,
    'top_candidates': rows,
}, ensure_ascii=False, indent=2, default=str))
"
```

discipline_template:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import Course
from courses.services import build_discipline_template
import json

course = Course.objects.get(id='$COURSE_ID')
template = build_discipline_template(course=course)
print(json.dumps(template, ensure_ascii=False, indent=2, default=str))
"
```

source_structure = auto_generation:
```PowerShell
$body = @{
  source_structure = "auto_generation"
  source_course_id = $null
} | ConvertTo-Json
```
```PowerShell
Invoke-RestMethod `
  -Method PATCH `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/source-structure" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json" `
  -Body $body
```

генерация графа (можно перезапустить в случае ошибки):
```PowerShell
$body = @{  
min_topics = 8  
max_topics = 20  
} | ConvertTo-Json
```
```PowerShell
Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/generate-graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json" `
  -Body $body
```

статус курса и что реально создалось:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import Course, Topic, TopicKSA, LearningOutcome, LearningTask
import json

course = Course.objects.get(id='$COURSE_ID')

print(json.dumps({
  'course_id': str(course.id),
  'title': course.title,
  'graph_gen_status': course.graph_gen_status,
  'topics_count': Topic.objects.filter(course=course).count(),
  'topic_ksa_count': TopicKSA.objects.filter(topic__course=course).count(),
  'learning_outcomes_count': LearningOutcome.objects.filter(ksa__topic__course=course).count(),
  'learning_tasks_count': LearningTask.objects.filter(outcome__ksa__topic__course=course).count(),
}, ensure_ascii=False, indent=2))
"
```

получить текущий граф курса и проверить resolved `content_source` по темам:
```PowerShell
$graph = Invoke-RestMethod `
  -Method GET `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }

$graph.topics | Select-Object id,title,topic_content_mode,content_source | Format-Table -AutoSize
```

выбрать тему для проверки `TopicContentChunk` pipeline:
```PowerShell
$TOPIC_ID = $graph.topics[0].id
$TOPIC_ID
```

подготовить длинный manual `TopicContent`, чтобы получить несколько чанков:
```PowerShell
$block1 = ("Контекст и цели темы. " * 80).Trim()
$block2 = ("Концептуальное объяснение условной вероятности и независимости событий. " * 80).Trim()
$block3 = ("Пошаговый алгоритм решения задач на условную вероятность. " * 80).Trim()
$block4 = ("Разобранные примеры и применение формулы Байеса. " * 80).Trim()

$manualTopicContentBody = @{
  content_text = "$block1`n`n$block2`n`n$block3`n`n$block4"
  status = "draft"
} | ConvertTo-Json
```

создать manual `TopicContent` и запустить pipeline chunking:
```PowerShell
$topicContentCreateResponse = Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/topics/$TOPIC_ID/content" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json" `
  -Body $manualTopicContentBody
```

посмотреть ответ создания `TopicContent` и сохранить `TOPIC_CONTENT_ID`:
```PowerShell
$topicContentCreateResponse = Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/topics/$TOPIC_ID/content" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json" `
  -Body $manualTopicContentBody
```

посмотреть worker logs после создания `TopicContent`:
```PowerShell
$topicContentCreateResponse | Format-List
$TOPIC_CONTENT_ID = $topicContentCreateResponse.id
$TOPIC_CONTENT_ID
```

проверить через Django shell, что для current `TopicContent` создались `TopicContentChunk` и embeddings:
```PowerShell
docker compose logs celery-worker --since 5m --tail 200
```

проверить retrieval по embeddings чанков через similarity search в `pgvector`:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import TopicContent, TopicContentChunk
import json

topic_content = TopicContent.objects.filter(topic_id='$TOPIC_ID', is_current=True).order_by('-created_at').first()
chunks = list(
    TopicContentChunk.objects.filter(topic_content=topic_content)
    .order_by('chunk_index')
)

print(json.dumps({
    'topic_content_id': None if topic_content is None else str(topic_content.id),
    'topic_status': None if topic_content is None else topic_content.status,
    'chunks_count': len(chunks),
    'chunks': [
        {
            'id': str(chunk.id),
            'chunk_index': chunk.chunk_index,
            'section_type': chunk.section_type,
            'embedding_dim': 0 if chunk.embedding is None else len(chunk.embedding),
            'first_embedding_value': None if chunk.embedding is None else float(chunk.embedding[0]),
        }
        for chunk in chunks
    ],
}, ensure_ascii=False, indent=2, default=str))
"
```

подготовить обновлённый manual `TopicContent` для проверки пересборки чанков:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import TopicContentChunk
from courses.services import call_ml_llm_query_embedding
from pgvector.django import CosineDistance
import json

query = 'пошаговое решение задач на условную вероятность'
query_embedding = call_ml_llm_query_embedding(text=query)

rows = list(
    TopicContentChunk.objects.annotate(distance=CosineDistance('embedding', query_embedding))
    .order_by('distance', 'created_at')
    .values('id', 'topic_content_id', 'chunk_index', 'section_type', 'distance')[:5]
)

for row in rows:
    row['similarity'] = 1.0 - float(row['distance'])

print(json.dumps({
    'query': query,
    'top_chunks': rows,
}, ensure_ascii=False, indent=2, default=str))
"
```

обновить `TopicContent` и запустить повторный chunking pipeline:
```PowerShell
$topicContentUpdateResponse = Invoke-RestMethod `
  -Method PATCH `
  -Uri "$BASE/api/v1/topics/$TOPIC_ID/content" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json" `
  -Body $manualTopicContentUpdateBody
```

посмотреть ответ обновления `TopicContent` и сохранить `UPDATED_TOPIC_CONTENT_ID`:
```PowerShell
$topicContentUpdateResponse | Format-List
$UPDATED_TOPIC_CONTENT_ID = $topicContentUpdateResponse.id
$UPDATED_TOPIC_CONTENT_ID
```

посмотреть worker logs после обновления `TopicContent`:
```PowerShell
docker compose logs celery-worker --since 5m --tail 200
```

проверить через Django shell, что старая версия снята с `is_current`, а для новой current-версии пересобраны чанки и embeddings:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import TopicContent, TopicContentChunk
import json

topic_contents = list(
    TopicContent.objects.filter(topic_id='$TOPIC_ID')
    .order_by('-created_at')
    .values('id', 'status', 'is_current', 'created_at', 'updated_at')
)

current_topic_content = TopicContent.objects.filter(topic_id='$TOPIC_ID', is_current=True).order_by('-created_at').first()
current_chunks = list(
    TopicContentChunk.objects.filter(topic_content=current_topic_content)
    .order_by('chunk_index')
)

print(json.dumps({
    'topic_contents': topic_contents,
    'current_topic_content_id': None if current_topic_content is None else str(current_topic_content.id),
    'current_chunks_count': len(current_chunks),
    'current_chunks': [
        {
            'id': str(chunk.id),
            'chunk_index': chunk.chunk_index,
            'section_type': chunk.section_type,
            'embedding_dim': 0 if chunk.embedding is None else len(chunk.embedding),
            'first_embedding_value': None if chunk.embedding is None else float(chunk.embedding[0]),
        }
        for chunk in current_chunks
    ],
}, ensure_ascii=False, indent=2, default=str))
"
```

повторно проверить retrieval по embeddings после ручного редактирования `TopicContent`:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import TopicContentChunk
from courses.services import call_ml_llm_query_embedding
from pgvector.django import CosineDistance
import json

query = 'обновлённый процедурный разбор задач на условную вероятность'
query_embedding = call_ml_llm_query_embedding(text=query)

rows = list(
    TopicContentChunk.objects.annotate(distance=CosineDistance('embedding', query_embedding))
    .order_by('distance', 'created_at')
    .values('id', 'topic_content_id', 'chunk_index', 'section_type', 'distance')[:5]
)

for row in rows:
    row['similarity'] = 1.0 - float(row['distance'])

print(json.dumps({
    'query': query,
    'top_chunks': rows,
}, ensure_ascii=False, indent=2, default=str))
"
```

выбрать тему для ручной работы с `TopicContent`
```PowerShell
$TOPIC_ID = $graph.topics[0].id
$TOPIC_ID
```

подготовить payload для ручного создания `TopicContent`:
```PowerShell
$manualTopicContentBody = @{
  content_text = @"
Это авторский longread по теме.

В этом тексте я вручную задаю содержание темы вместо полностью автоматической генерации.
Здесь можно описать контекст, ключевые понятия, пошаговое объяснение и примеры.
"@
  status = "draft"
} | ConvertTo-Json
```

создать manual `TopicContent` для выбранной темы:
```PowerShell
$topicContentCreateResponse = Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/topics/$TOPIC_ID/content" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json" `
  -Body $manualTopicContentBody
```

посмотреть ответ создания `TopicContent` и сохранить `TOPIC_CONTENT_ID`:
```PowerShell
$topicContentCreateResponse | Format-List
$TOPIC_CONTENT_ID = $topicContentCreateResponse.id
$TOPIC_CONTENT_ID
```

получить текущий `TopicContent` темы через `GET`:
```PowerShell
$topicContentGetResponse = Invoke-RestMethod `
  -Method GET `
  -Uri "$BASE/api/v1/topics/$TOPIC_ID/content" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }
```

вывести текущий `TopicContent` темы в JSON:
```PowerShell
$topicContentGetResponse | ConvertTo-Json -Depth 10
```

подготовить payload для ручного обновления `TopicContent`:
```PowerShell
$manualTopicContentUpdateBody = @{
  content_text = @"
Это обновлённый авторский longread по теме.

Я вручную изменил содержание темы.
Теперь здесь уточнены формулировки, добавлены пояснения и обновлён статус.
"@
  status = "verified"
} | ConvertTo-Json
```

обновить manual `TopicContent`:
```PowerShell
$topicContentUpdateResponse = Invoke-RestMethod `
  -Method PATCH `
  -Uri "$BASE/api/v1/topics/$TOPIC_ID/content" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json" `
  -Body $manualTopicContentUpdateBody
```

посмотреть ответ обновления `TopicContent`:
```PowerShell
$topicContentUpdateResponse | Format-List
```

посмотреть worker logs для фоновой пересборки `TopicContentChunk`:
```PowerShell
docker compose logs -f celery-worker
```

проверить через Django shell, что current `TopicContent` и его чанки реально создались:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import TopicContent, TopicContentChunk
import json

topic_content = TopicContent.objects.filter(topic_id='$TOPIC_ID', is_current=True).order_by('-created_at').first()

payload = {
    'topic_content': None if topic_content is None else {
        'id': str(topic_content.id),
        'topic_id': str(topic_content.topic_id),
        'source': topic_content.source,
        'context_level': topic_content.context_level,
        'status': topic_content.status,
        'is_current': topic_content.is_current,
        'chunks_count': TopicContentChunk.objects.filter(topic_content=topic_content).count(),
    },
    'chunks': [] if topic_content is None else list(
        TopicContentChunk.objects.filter(topic_content=topic_content)
        .order_by('chunk_index')
        .values('id', 'chunk_index', 'section_type', 'created_at')
    ),
}

print(json.dumps(payload, ensure_ascii=False, indent=2, default=str))
"
```

прикрепить документ напрямую к теме перед повторной генерацией:
```PowerShell
$topicDocJson = curl.exe -s -X POST "$BASE/api/v1/courses/$COURSE_ID/documents" `
  -H "Authorization: Bearer $COURSE_TOKEN" `
  -F "title=topic_specific_material" `
  -F "topic_id=$TOPIC_ID" `
  -F "file=@$FILE;type=application/vnd.openxmlformats-officedocument.wordprocessingml.document"
```

посмотреть ответ topic-level загрузки документа и сохранить `TOPIC_DOCUMENT_ID`:
```PowerShell
$topicDocResponse = $topicDocJson | ConvertFrom-Json
$topicDocResponse | Format-List
$TOPIC_DOCUMENT_ID = $topicDocResponse.id
$TOPIC_DOCUMENT_ID
```

проверить через Django shell, что документ действительно привязан к теме
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import Document
import json

print(json.dumps(
    list(
        Document.objects.filter(topic_id='$TOPIC_ID')
        .values('id', 'title', 'topic_id', 'parse_status', 'created_at', 'updated_at')
    ),
    ensure_ascii=False,
    indent=2,
    default=str,
))
"
```

повторно получить граф курса и проверить приоритет manual `TopicContent` над `topic_docs`
```PowerShell
$graphAfterTopicDoc = Invoke-RestMethod `
  -Method GET `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }

$graphAfterTopicDoc.topics | Select-Object id,title,topic_content_mode,content_source | Format-Table -AutoSize
```

удалить manual `TopicContent` темы
```PowerShell
$topicContentDeleteResponse = Invoke-RestMethod `
  -Method DELETE `
  -Uri "$BASE/api/v1/topics/$TOPIC_ID/content" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }
```

посмотреть ответ удаления `TopicContent`
```PowerShell
$topicContentDeleteResponse | Format-List
```

проверить через Django shell, что `TopicContent` и связанные `TopicContentChunk` удалены
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import TopicContent, TopicContentChunk
import json

topic_contents = list(
    TopicContent.objects.filter(topic_id='$TOPIC_ID')
    .values('id', 'topic_id', 'source', 'status', 'is_current', 'created_at', 'updated_at')
)
chunk_count = TopicContentChunk.objects.filter(topic_content__topic_id='$TOPIC_ID').count()

print(json.dumps({
    'topic_contents': topic_contents,
    'topic_content_chunk_count': chunk_count,
}, ensure_ascii=False, indent=2, default=str))
"
```

повторно получить граф курса и проверить, что после удаления manual-режима selector переключился на документный источник
```PowerShell
$graphAfterDelete = Invoke-RestMethod `
  -Method GET `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }

$graphAfterDelete.topics | Select-Object id,title,topic_content_mode,content_source | Format-Table -AutoSize
```

получить текущий граф:  
```PowerShell  
Invoke-RestMethod `
  -Method GET `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }
```
  
пример ручного редактирования, например изменить название первой темы и первой задачи:  
  
```PowerShell  
$graph = Invoke-RestMethod `
  -Method GET `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }
```  
  
```PowerShell  
$graph.topics[0].title = "Обновлённая тема вручную: НЕВЕРОЯТНАЯ ТЕОРИЯ ВЕРОЯТНОСТИ И СТАТИСТИКА."
$graph.learning_tasks[0].title = "Обновлённая задача вручную: расскажи когда 1+1=10."

$patchBody = @{
  topics = $graph.topics
  topic_ksa = $graph.topic_ksa
  topic_dependencies = $graph.topic_dependencies
  competencies = $graph.competencies
  competency_ksa = $graph.competency_ksa
  learning_outcomes = $graph.learning_outcomes
  learning_tasks = $graph.learning_tasks
} | ConvertTo-Json -Depth 10

$patchBytes = [System.Text.Encoding]::UTF8.GetBytes($patchBody)

Invoke-RestMethod `
  -Method PATCH `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json; charset=utf-8" `
  -Body $patchBytes
```

подтверждение графа курса:  
```PowerShell  
Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/confirm-graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }
```  

посмотреть worker logs после `confirm-graph`
```PowerShell
docker compose logs -f celery-worker
```

создание UserTrack через Django shell (пока нет endpoint):
```PowerShell
docker compose exec django-api python manage.py shell -c "
import uuid
from django.contrib.auth import get_user_model
from courses.models import Course, UserTrack, UserTrackStatus, UserTrackDiagnosticStatus, UserTrackQuestionnaireStatus

User = get_user_model()
user = User.objects.get(id=2)
course = Course.objects.get(id='e95f7b72-64ae-4f64-85d5-051cb6bc51ce')

track = UserTrack.objects.create(
    user=user,
    course=course,
    status=UserTrackStatus.PENDING,
    questionnaire_status=UserTrackQuestionnaireStatus.NOT_STARTED,
    diagnostic_status=UserTrackDiagnosticStatus.NOT_OFFERED,
    generation_step=None,
    failure_reason=None,
    generation_params={},
)
print('track_id:', track.id)
"
```

initialize_user_track_graph синхронно:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.services import initialize_user_track_graph
initialize_user_track_graph(track_id='$TRACK_ID')
print('OK')
"
```

проверка результата в DB:
```PowerShell
docker compose exec django-api python manage.py shell -c "
import json
from courses.models import UserTrack, UserKnowledgeNode, UserKSANode, UserCompetencyNode

track = UserTrack.objects.order_by('-created_at').first()
print(json.dumps({
    'track_id': str(track.id),
    'status': str(track.status),
    'generation_step': str(track.generation_step),
    'failure_reason': track.failure_reason,
    'knowledge_nodes': UserKnowledgeNode.objects.filter(last_updated_by_track=track).count(),
    'ksa_nodes': UserKSANode.objects.filter(user=track.user).count(),
    'competency_nodes': UserCompetencyNode.objects.filter(user=track.user).count(),
}, ensure_ascii=False, indent=2))
"
```

содержимое `UserKnowledgeNode` для последнего созданного трека:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import UserTrack, UserKnowledgeNode
track = UserTrack.objects.order_by('-created_at').first()
if track:
    nodes = UserKnowledgeNode.objects.filter(last_updated_by_track=track)
    print(f'Track ID: {track.id}')
    print(f'Количество узлов: {nodes.count()}')
    for node in nodes:
        print(f'Тема: {node.topic.title if node.topic else None}, name: {node.name}, bkt_probability: {node.bkt_probability}, status: {node.status}, source: {node.source}')
"
```

---

Draft part:

граф:
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
        'main_topic': course.main_topic,
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
    'learning_tasks': learning_tasks,
}

print(json.dumps(payload, ensure_ascii=False, indent=2, default=str))
"
```

проверка наличия template в prompt generation pipeline:
```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import Course
from courses.services import build_discipline_template, prepare_course_graph_generation_payload
import json

course = Course.objects.get(id='$COURSE_ID')
template = build_discipline_template(course=course)
payload = prepare_course_graph_generation_payload(
    course_id=course.id,
    min_topics=8,
    max_topics=20,
)

print('--- DISCIPLINE TEMPLATE ---')
print(json.dumps(template, ensure_ascii=False, indent=2, default=str))
print('--- SYSTEM PROMPT (FIRST 3000 CHARS) ---')
print(payload['system_prompt'][:3000])
"
```

  
получить текущий граф:  
```PowerShell  
Invoke-RestMethod `
  -Method GET `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }
```

проверить контент по темам:
```PowerShell
$graph.topics | Select-Object id,title,topic_content_mode,content_source | Format-Table -AutoSize
```
  
пример ручного редактирования, например изменить название первой темы и первой задачи:  
  
```PowerShell  
$graph = Invoke-RestMethod `
  -Method GET `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }
```  
  
```PowerShell  
$graph.topics[0].title = "Обновлённая тема вручную: НЕВЕРОЯТНАЯ ТЕОРИЯ ВЕРОЯТНОСТИ И СТАТИСТИКА."
$graph.learning_tasks[0].title = "Обновлённая задача вручную: расскажи когда 1+1=10."

$patchBody = @{
  topics = $graph.topics
  topic_ksa = $graph.topic_ksa
  topic_dependencies = $graph.topic_dependencies
  competencies = $graph.competencies
  competency_ksa = $graph.competency_ksa
  learning_outcomes = $graph.learning_outcomes
  learning_tasks = $graph.learning_tasks
} | ConvertTo-Json -Depth 10

$patchBytes = [System.Text.Encoding]::UTF8.GetBytes($patchBody)

Invoke-RestMethod `
  -Method PATCH `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json; charset=utf-8" `
  -Body $patchBytes
```

подтверждение графа курса:  
```PowerShell  
Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/courses/$COURSE_ID/confirm-graph" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" }
```  

посмотреть prompt:
```PowerShell  
docker compose exec django-api python manage.py shell -c "
from courses.models import Course
from courses.services import build_discipline_template, prepare_course_graph_generation_payload
import json

course = Course.objects.get(id='$COURSE_ID')
template = build_discipline_template(course=course)
payload = prepare_course_graph_generation_payload(
    course_id=course.id,
    min_topics=8,
    max_topics=20,
)

print('--- DISCIPLINE TEMPLATE ---')
print(json.dumps(template, ensure_ascii=False, indent=2, default=str))
print('--- SYSTEM PROMPT (FULL) ---')
print(payload['system_prompt'])
"
```

```
docker compose exec ml-llm sh -lc "ls -lh /tmp/course_graph_debug && sed -n '1,260p' /tmp/course_graph_debug/f6e7dc40-31ad-4c94-a298-b857d2282670_raw_llm_response.txt"
```

```
docker compose exec ml-llm sh -lc "sed -n '1,260p' /tmp/course_graph_debug/f6e7dc40-31ad-4c94-a298-b857d2282670_parsed_graph.json"
```

```
docker compose restart ml-llm celery-worker django-api
```

```
curl.exe -X POST http://localhost:8002/ml/query-embedding `
  -H "Content-Type: application/json" `
  -d "{\"text\":\"test\"}"
```

если ml-llm висит на Waiting for application startup:

1) в docker-compose.yml
```
command: python -X faulthandler -m uvicorn main:app --host 0.0.0.0 --port 8002
```
2) run
```
docker compose up -d --build ml-llm
```

```
docker compose exec django-api python manage.py shell -c "
from courses.tasks import generate_diagnostic_test_task
test_id = '$TEST_ID'
generate_diagnostic_test_task.delay(str(test_id), questions_count=6)
print(f'Generation task queued for test {test_id}')
"
```

```
$answersBody = @"
{
  "answers": [
    { "question_id": "b68a34c1-3c88-4f88-8789-8d1ef993e896", "selected_option": 0 },
    { "question_id": "350a3bda-ac10-4b13-9459-240a73d3c30f", "selected_option": 0 },
    { "question_id": "cc63dc3d-080d-48ab-8bf5-a87cf352ecb1", "selected_option": 0 },
    { "question_id": "ae47d844-7c3f-4e73-b7fb-994d8b3ce4f1", "selected_option": 0 },
    { "question_id": "a125d939-306c-4289-a7a4-89ad268b15c5", "selected_option": 0 },
    { "question_id": "4e413276-c639-4bed-a091-14f5f338ea4f", "selected_option": 0 },
    { "question_id": "641d2433-4c41-469b-a7b2-a68cb771b43d", "selected_option": 0 },
    { "question_id": "0b4ddd23-2b9f-4f32-8df6-e94b55a7bebb", "selected_option": 0 }
  ]
}
"@

Invoke-RestMethod `
  -Method POST `
  -Uri "$BASE/api/v1/diagnostic-attempts/$ATTEMPT_ID/answers" `
  -Headers @{ Authorization = "Bearer $COURSE_TOKEN" } `
  -ContentType "application/json" `
  -Body $answersBody
```



```md
This PR adds bank-first diagnostic attempt preparation for user tracks with runtime LLM fallback for missing question slots, and aligns the API contract with the implemented backend flow.

**Additions**

- `POST /api/v1/tracks/{track_id}/diagnostic-attempt` prepares a personalized diagnostic attempt for a UserTrack.
- The preparation flow builds a per-KSA plan using `UserKSANode`, `TrackSelfAssessment`, default priors, strategy selection, `cap=30`, and `cold_start_cap=20`.
- Question selection now prefers the published course diagnostic bank and excludes questions answered by the same user in the last 7 days.
- Runtime LLM fallback is called only for missing `(primary_ksa_id, bloom_level, count)` slots.
- Runtime fallback questions are persisted separately with `ContentArtifact.source = llm_runtime`.
- `DiagnosticAttemptQuestion` stores ordered attempt questions with `question_source = bank | llm_fallback`.
- Saving answers now marks fallback answers with `DiagnosticAnswer.was_fallback = true`.
- The prepare response redacts correct_index from student-facing diagnostic questions.
- `ml-llm` now accepts and validates `required_bloom_counts` for runtime fallback generation.
- Updated `docs/architecture/endpoints.md` with the new endpoint contract.

**Verification**

- Verified migrations in docker-compose with `makemigrations --check --dry-run`.
- Verified Django system checks and full courses test suite: `225 tests OK`.
- Verified live docker-compose E2E smoke for bank-first selection, cooldown exclusion, runtime fallback persistence, and `was_fallback=true`.
- Verified `ml-llm` runtime Bloom distribution validation accepts correct output and rejects invalid distribution.
```

```PowerShell
git push -u origin HEAD
```

```PowerShell
Invoke-RestMethod -Method Post -Uri "http://localhost:8002/api/v1/user-track/detect-gaps" -ContentType "application/json" -Body '{"course_id":"course-1","user_id":"u-1","mode":"entry","gap_detections":[],"ksa_context":[],"existing_recommended_ksa_ids":[]}'
```

```PowerShell
Invoke-RestMethod -Method Post -Uri "http://localhost:8000/api/v1/decision/next-action" `
  -ContentType "application/json" `
  -Body '{"bkt_probability": 0.97, "bkt_reliable": true, "status": "not_assessed", "last_action": null, "step": 1, "recent_incorrect_streak": 0, "interaction_count": 0, "bkt_delta_last_5": null}'
```

```PowerShell
$result = Invoke-RestMethod -Method Post -Uri "http://localhost:8002/api/v1/runtime-gen/generate-action" `
  -ContentType "application/json" `
  -Body '{
    "action_type": "explain",
    "dialogue_context": {
      "topic_id": "topic-1",
      "topic_title": "Линейные уравнения",
      "ksa_id": "ksa-1",
      "learning_objective": "Решать линейные уравнения.",
      "last_student_response": "Я не понимаю почему переношу 3.",
      "messages": []
    },
    "rag_top_k": 2
  }'

$result.content.body
```

```PowerShell
docker compose exec django-api python manage.py shell -c "
from courses.models import TopicContent, Topic
topics = Topic.objects.filter(course_id='dcf341a4-4f99-41a1-abc6-b42ccf06290b')[:3]
for t in topics:
    print(t.id, t.title)
    tc = TopicContent.objects.filter(topic=t).first()
    print('  has content:', tc is not None)
"
```

```PowerShell
$response = Invoke-WebRequest -Method Post `
  -Uri "http://localhost:8002/api/v1/runtime-gen/generate-action" `
  -ContentType "application/json" `
  -UseBasicParsing `
  -Body '{"action_type":"explain","dialogue_context":{"topic_id":"7253c5a6-f1e6-45bc-acb8-349b0e810ca9","topic_title":"Conditional Probability","ksa_id":"","learning_objective":"Understand conditional probability","last_student_response":"I do not understand why we divide by P(B)","messages":[]},"rag_top_k":4}'
$r = [System.Text.Encoding]::UTF8.GetString($response.RawContentStream.ToArray()) | ConvertFrom-Json
$r.content.body
$r.retrieval_source
```