1) "BKT" and "оценка сложности материала при создании статичных учебных программ":
	1. "BKT":
	 - Моделирует забывание с помощью экспоненциальной кривой. -> **хорошо**
	 - Разбиение контента на микромодули и добавление кластеризации делают модель более гибкой, чем стандартные BKT. -> **хорошо**
	 - Всё равно ориентируемся только на правильность/неправильность решения - важен только исход. -> **не исправлено**
	 - Не учитываем шаги решения. -> **не исправлено**
	Note:
	в формуле $$maxF(S) = \sum_j^vw_jk_j-a\sum_vT_i$$ можно варьировать веса $$w_i$$ , к примеру, в зависимости от частоты встречаемости задания.
	1. "оценка сложности материала при создании статичных учебных программ":
	- "прирост" -> $$\sum G_{ij}$$ это сумма вкладов микромодулей.
	- Открыты вопросы по чистоте диагностики и весу каждой задачи.
	- "прирост" - это либо апостериорное обновление $$p(знает|компетенция_i)$$, либо ожидаемый вклад модуля, который ещё нужно оценить.
	- Также никак не учитывает шаги решения конкретной задачи. (для понимания мат модели для построения трека документ хорош)
2) DKT and набор требуемых собираемых данных

	1. DKT работает не с отдельными ответами, а с **последовательностью взаимодействий ученика**:
   $$x_t = \{q_t, a_t\}$$
	   где \($q_t$\) — задание, \($a_t$\) — результат ответа.
	   Значит, минимально нужны:
	- `user_id` — история конкретного ученика.
	- `item_id` / `question_id` / `exercise_id` — с чем было взаимодействие.
	- `outcome` (`is_correct` / `score`) — результат.
	- `timestamp` и/или `sequence_index` — порядок событий.
	
	2. Нам для DKT важна именно **временная структура** данных, потому что модель должна учитывать зависимости между шагами решения.
	Поэтому ещё нужны:
	   - `event_timestamp` — когда произошло взаимодействие.
	   - `step_index` внутри сессии / урока.
	   - `time_spent` (затраченное время на решение)/`time_since_prev_event` — лучше собирать сразу.
	   - `session_id` / `attempt_id` — границы последовательностей.
	
	1. Результат должен быть как можно более явным(результат ответа должен быть представлен **в явном и нормализованном виде**, чтобы его можно было напрямую использовать как target для модели):
	   - `is_correct`
	   - `raw_score`
	   - `normalized_score`
	   - `is_partial`
	   - `result_label = correct / incorrect / partial / timeout / abandoned`
	
	4. Нужно сохранять структурированные данные о попытках (learning trace) (и в прерванных сессиях, думаю, тоже):
	   - `attempt_no` 
	   - `is_final_attempt`
	   - `retry_count`
	   - `session_end_reason` / `abandoned`
	
	5. Для open-ended / code задач нужно собирать более богатые данные (нам для кода и open questions надо больше разных оценочных данных):
	   - `submission_no`
	   - `tests_passed / tests_total`
	   - `checker_type` (`sandbox / llm / sympy / rule-based`)
	   - `checker_confidence`
	   - `error_type` (`compile / runtime / logic`)
	   - `time_to_first_submit`
	   - `time_to_pass`
	
	6. Надо будет собрать какой-то единый `interaction/event log`, из которого можно и будем собирать датасет.
	   Чтобы для записи об ивенте что-то такое было (с доменной модели у нас уже есть модуль так-то):
	   - `event_id`
	   - `user_id`
	   - `session_id`
	   - `event_timestamp`
	   - `item_id`
	   - `topic_id`
	   - `outcome`
	   - `score`
	   - `attempt_no`
	   - `metadata_json`
	7. какой-то такой список пока:
	   - `user_id` 
	   - `item_id/question_id/exercise_id`
	   - `topic_id`
	   - `is_correct` / `score`
	   - `timestamp` / `sequence_index`
	   - `session_id` / `attempt_id`
	   - `time_spent`
	   - `time_since_prev_event`
	   - `attempt_no`
	   - `is_final_attempt`
	   - `abandoned / session_end_reason`
	   - `checker_type`
	   - `error_type`
	   - `normalized outcome`
	   - richer telemetry

course -> KSA
course -> KSADependency

learning -> UserKSANode
learning -> UserTrackTopic.active_ksa

assessment -> DiagnosticQuestion.ksa

interaction Log -> InteractionRecord ()

LearningUnit:

topic_id
ksa_id
response_outcome (enum: {full | partial | skipped})
competency_id
