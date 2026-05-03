df columns:
```
interaction_id
user_id
session_id
track_id
course_id
learning_unit_id
agent_decision_id
event_timestamp
event_index
event_type

topic_id
competency_id
primary_ksa_id

name_of_course
session_mode

action_type
action_source
content_id
content_type
content_format
content_version_hash
question_type
difficulty
fragment_id
document_section_id

answer_text
response_type
result_label
score_float
is_correct
time_spent_seconds
attempt_no
hint_count
appeal_used

bkt_before
bkt_after
mastery_before_topic
mastery_after_topic
mastery_before_ksa
mastery_after_ksa

is_terminal_event
terminal_event_type
```

df_example:

| interaction_id | user_id | session_id | track_id | course_id     | learning_unit_id | agent_decision_id | event_timestamp     | event_index | event_type          | topic_id     | competency_id | primary_ksa_id       | name_of_course | session_mode | action_type | action_source  | content_id | content_type         | content_format | question_type | difficulty | answer_text                   | response_type         | result_label | score_float | is_correct | time_spent_seconds | attempt_no | hint_count | bkt_before | bkt_after | mastery_before_topic | mastery_after_topic | mastery_before_ksa | mastery_after_ksa | is_terminal_event | terminal_event_type |
| -------------- | ------- | ---------- | -------- | ------------- | ---------------- | ----------------- | ------------------- | ----------: | ------------------- | ------------ | ------------- | -------------------- | -------------- | ------------ | ----------- | -------------- | ---------- | -------------------- | -------------- | ------------- | ---------: | ----------------------------- | --------------------- | ------------ | ----------: | ---------- | -----------------: | ---------: | ---------: | ---------: | --------: | -------------------: | ------------------: | -----------------: | ----------------: | ----------------- | ------------------- |
| int_001        | user_17 | sess_501   | track_21 | course_math_1 | lu_1001          | dec_9001          | 2026-03-11 10:00:05 |           1 | pedagogical_action  | topic_lin_eq | comp_solve_eq | ksa_move_term        | math           | lesson       | ПП          | rule_reactive  | cont_7001  | explanation          | text           | null          |       0.35 | null                          | system_action         | null         |        null | null       |                  0 |          0 |          0 |       0.42 |      0.42 |                 0.42 |                0.38 |               0.38 |             false | null              |                     |
| int_002        | user_17 | sess_501   | track_21 | course_math_1 | lu_1001          | dec_9001          | 2026-03-11 10:01:40 |           2 | student_response    | topic_lin_eq | comp_solve_eq | ksa_move_term        | math           | lesson       | ПП          | rule_reactive  | cont_7001  | explanation_check    | open           | open          |       0.35 | "x=7"                         | answer                | partial      |        0.50 | false      |                 95 |          1 |          0 |       0.42 |      0.58 |                 0.42 |                0.55 |               0.38 |              0.52 | false             | null                |
| int_003        | user_17 | sess_501   | track_21 | course_math_1 | lu_1002          | dec_9002          | 2026-03-11 10:01:45 |           3 | pedagogical_action  | topic_lin_eq | comp_solve_eq | ksa_isolate_variable | math           | lesson       | ПМ          | rule_reactive  | cont_7002  | hint                 | text           | null          |       0.30 | null                          | hint_request_resolved | null         |        null | null       |                  0 |          1 |          1 |       0.58 |      0.58 |                 0.55 |                0.55 |               0.52 |              0.52 | false             | null                |
| int_004        | user_17 | sess_501   | track_21 | course_math_1 | lu_1003          | dec_9003          | 2026-03-11 10:03:10 |           4 | student_response    | topic_lin_eq | comp_solve_eq | ksa_isolate_variable | math           | lesson       | ПТ          | rule_proactive | cont_7003  | question             | mcq            | mcq           |       0.45 | "option_2"                    | answer                | correct      |        1.00 | true       |                 62 |          1 |          1 |       0.58 |      0.81 |                 0.55 |                0.79 |               0.52 |              0.80 | false             | null                |
| int_005        | user_17 | sess_501   | track_21 | course_math_1 | lu_1004          | dec_9004          | 2026-03-11 10:04:20 |           5 | explanation_request | topic_lin_eq | comp_solve_eq | ksa_isolate_variable | math           | lesson       | КИ          | user_triggered | cont_7003  | fragment_explanation | text           | null          |       0.40 | "почему переносим со знаком?" | ask_explanation       | null         |        null | null       |                 18 |          1 |          1 |       0.81 |      0.81 |                 0.79 |                0.79 |               0.80 |              0.80 | false             | null                |
| int_006        | user_17 | sess_501   | track_21 | course_math_1 | lu_1005          | dec_9005          | 2026-03-11 10:06:00 |           6 | session_end         | topic_lin_eq | comp_solve_eq | ksa_isolate_variable | math           | lesson       | РФ          | rule_proactive | cont_7004  | reflection           | text           | open          |       0.40 | "теперь понял"                | answer                | correct      |        1.00 | true       |                 44 |          1 |          0 |       0.81 |      0.95 |                 0.79 |                0.95 |               0.80 |              0.94 | true              | completed           |

