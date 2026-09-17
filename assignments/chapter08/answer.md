# Chapter 08 확장 실습 답안 템플릿

> **과제:** JOIN과 집계로 서비스 질문에 답하기  
> **사용 방법:** 이 파일을 내려받아 본인의 GitHub 저장소에 `chapter08_answer.md`라는 이름으로 저장한 뒤 실습하면서 바로 작성합니다.  
> **제출 방법:** LMS에는 파일을 직접 업로드하지 않고, **본인 GitHub 저장소의 `chapter08_answer.md` 파일 URL**을 제출합니다.

---

## 제출 전 주의

이 파일과 캡처 화면에는 실제 비밀번호, 전체 DB 접속 URL, API Key, 개인정보를 기록하지 않습니다.

```text
GitHub 계정 또는 별칭:wjs0951467-wq
과제 작성일:2026-09-17
사용한 AI 도구:chat gpt
```

---

# 1. Chapter 07 기준 상태 확인

다음을 실행합니다.

```text
code/chapter08/00_check_course_project.sql
```

## 1-1. 사전 검사 결과

```text
검증 메시지:Chapter 08 prerequisite check passed

students 행 수:3
instructors 행 수:2
courses 행 수:3
enrollments 행 수:5

전체 신청 건수:5
전체 recorded_amount:590000
활성 신청 건수:3
활성 recorded_amount:340000
취소 제외 신청 건수:4
취소 제외 recorded_amount:440000
```

기준값:

```text
students = 3
instructors = 2
courses = 3
enrollments = 5

전체 = 5 / 590000
활성 = 3 / 340000
취소 제외 = 4 / 440000
```

### 기준값이 다르면 그대로 진행하면 안 되는 이유

```text
Chapter 08은 Chapter 07에서 만든 course_project 데이터를 기준으로
JOIN과 집계를 진행하기 때문에 기준 데이터가 다르면 이후 결과도 달라질 수 있다.
따라서 기준값이 다르면 그대로 진행하지 않고 Chapter 07 데이터가 정상 상태인지 먼저 확인한 뒤 다시 검증해야 한다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter08/images/step01_prerequisite.png
```

`여기에 사전 검사 통과 화면을 삽입하세요.`
![01](images/step01_prerequisite.png)
---

# 2. 업무 질문을 SQL보다 먼저 정의하기

다음 세 질문을 각각 SQL 작성 전에 먼저 정의합니다.

## 질문 A

```text
업무 질문:신청 한 건마다 학생 이름과 강의 제목을 조회한다.
결과 한 행의 의미:수강신청 한 건
포함 상태:모든 신청 상태
제외 상태:없음
JOIN할 테이블:enrollments, students, courses
JOIN 경로:
enrollments.student_id = students.id
enrollments.course_id = courses.id
INNER JOIN / LEFT JOIN 선택:INNER JOIN
그 이유:신청 한 건을 기준으로 실제 연결된 학생과 강의 정보를 조회하는 것이 목적이기 때문이다.
예상 행 수:5행
```

## 질문 B

```text
업무 질문:강의별로 취소되지 않은 신청 수와 recorded_amount 합계를 구한다.
결과 한 행의 의미:강의 한 개
포함 상태:신청, 수강중, 완료
제외 상태:취소
JOIN할 테이블:courses, enrollments
JOIN 경로:courses.id = enrollments.course_id
집계 대상:취소 제외 신청 건수와 recorded_amount 합계
예상 결과:
강의 301 = 2건 / 200000
강의 302 = 2건 / 240000
강의 303 = 0건 / 0
```

## 질문 C

```text
업무 질문:취소되지 않은 신청이 없는 학생도 포함하여 학생별 신청 상태를 확인한다.
결과 한 행의 의미:학생 한 명
포함 상태:신청, 수강중, 완료
제외 상태:취소
0건인 부모도 보여야 하는가:예
NULL을 어떻게 해석할 것인가:
취소되지 않은 신청이 연결되지 않은 학생으로 해석한다.
예상 결과:
학생 3명 모두 결과에 남고,
박서연은 취소 제외 신청이 없어 NULL 또는 0건으로 확인된다.:
```

---

# 3. INNER JOIN과 다중 JOIN

## 3-1. 신청 한 건마다 학생 이름과 강의 제목 조회

실행 전 예상:

```text
결과 한 행 = 수강신청 한 건
예상 행 수 = 5행
JOIN 경로 =
enrollments.student_id = students.id
enrollments.course_id = courses.id
```

내가 실행한 SQL:

```sql
SELECT
    e.id AS enrollment_id,
    s.name AS student_name,
    c.title AS course_title,
    e.status
FROM course_project.enrollments AS e
JOIN course_project.students AS s
    ON e.student_id = s.id
JOIN course_project.courses AS c
    ON e.course_id = c.id
ORDER BY e.id;
```

실제 결과:

```text
실제 행 수:5행
예상과 일치 여부:일치 
```

### 학생 이름이 여러 번 보이는 것이 중복 오류가 아닐 수 있는 이유

```text
학생 한 명이 여러 강의에 신청할 수 있기 때문에
수강신청 한 건을 기준으로 조회하면 같은 학생 이름이 여러 번 나올 수 있다.
이 경우는 중복 오류가 아니라 1:N 관계에서 자연스럽게 발생하는 결과이다.
```

## 3-2. 학생·강의·강사까지 연결

```text
결과 한 행 = 수강신청 한 건
강사까지 가는 JOIN 경로 =
enrollments.course_id = courses.id
courses.instructor_id = instructors.id
```

```sql
SELECT
    e.id AS enrollment_id,
    s.name AS student_name,
    c.title AS course_title,
    i.name AS instructor_name,
    e.status
FROM course_project.enrollments AS e
JOIN course_project.students AS s
    ON e.student_id = s.id
JOIN course_project.courses AS c
    ON e.course_id = c.id
JOIN course_project.instructors AS i
    ON c.instructor_id = i.id
ORDER BY e.id;
```

실제 행 수:

```text
5행
```

### 증거 화면

권장 경로:

```text
assignments/chapter08/images/step03_inner_join.png
```

`여기에 다중 JOIN 결과 화면을 삽입하세요.`
![03](images/step03_inner_join.png)
---

# 4. LEFT JOIN과 0건 표현

## 4-1. 강의별 취소 제외 신청 수

신청이 없는 강의도 결과에 남도록 작성합니다.

실행 전:

```text
결과 한 행 = 강의 한 개
강의 303의 예상 실제 신청 수 = 0건
강의 303의 예상 고유 학생 수 = 0명
강의 303의 예상 recorded_amount = 0
```

내 SQL:

```sql
SELECT
    c.id AS course_id,
    c.title AS course_title,
    COUNT(e.id) AS enrollment_count,
    COUNT(DISTINCT e.student_id) AS student_count,
    COALESCE(SUM(e.recorded_amount), 0) AS recorded_amount
FROM course_project.courses AS c
LEFT JOIN course_project.enrollments AS e
    ON c.id = e.course_id
   AND e.status <> '취소'
GROUP BY
    c.id,
    c.title
ORDER BY c.id;
```

실제 결과:

```text
강의 301: 2건 / 2명 / 200000
강의 302: 2건 / 2명 / 240000
강의 303: 0건 / 0명 / 0
```

## 4-2. `COUNT(*)`와 `COUNT(e.id)` 비교

강의 303을 기준으로 작성합니다.

```text
COUNT(*) 결과:1
COUNT(e.id) 결과:0
COUNT(DISTINCT e.student_id) 결과:0
```

### 왜 `COUNT(*) = 1`인데 실제 신청 수는 0일 수 있나요?

```text
LEFT JOIN은 신청이 없어도 강의 행은 남기 때문에 COUNT(*)는 1이 될 수 있다.
하지만 실제 신청 id는 없어서 COUNT(e.id)는 0이 된다.
```

### 자식 사건 수를 셀 때 `COUNT(child.id)`가 더 적절한 이유

```text
COUNT(child.id)는 실제 연결된 자식 데이터만 세기 때문에 신청 건수를 셀 때 더 정확하다.
```

---

# 5. `LEFT JOIN`에서 `ON`과 `WHERE` 조건 비교

취소 제외 신청만 연결한다고 가정합니다.

## 5-1. 조건을 `ON`에 둔 경우

```sql
SELECT
    s.id,
    s.name,
    e.status
FROM course_project.students AS s
LEFT JOIN course_project.enrollments AS e
    ON s.id = e.student_id
   AND e.status <> '취소'
ORDER BY s.id;
```

```text
결과 학생 수:3명
박서연 포함 여부:포함
```

## 5-2. 조건을 `WHERE`에 둔 경우

```sql
SELECT
    s.id,
    s.name,
    e.status
FROM course_project.students AS s
LEFT JOIN course_project.enrollments AS e
    ON s.id = e.student_id
WHERE e.status <> '취소'
ORDER BY s.id;
```

```text
결과 학생 수:2명
박서연 포함 여부:미포함
```

## 5-3. 차이 설명

```text
ON 조건이 LEFT JOIN의 오른쪽 연결 대상을 제한하는 방식:
취소가 아닌 신청만 연결하고, 학생 행 자체는 유지한다.
WHERE 조건이 JOIN 이후 결과 행을 제거하는 방식:
JOIN이 끝난 뒤 취소가 아닌 행만 남기기 때문에 NULL인 행도 제거된다.
이번 사례에서 ON = 3명, WHERE = 2명이 되는 이유:
박서연은 취소 제외 신청이 없어서 ON에서는 남지만, WHERE에서는 NULL 행이 제거되기 때문이다.
```

---

# 6. 신청이 없는 학생 찾기 — 두 방법 비교

## 방법 1. `LEFT JOIN ... IS NULL`

```sql
SELECT
    s.id,
    s.name
FROM course_project.students AS s
LEFT JOIN course_project.enrollments AS e
    ON s.id = e.student_id
   AND e.status <> '취소'
WHERE e.id IS NULL;
```

## 방법 2. `NOT EXISTS`

```sql
SELECT
    s.id,
    s.name
FROM course_project.students AS s
WHERE NOT EXISTS (
    SELECT 1
    FROM course_project.enrollments AS e
    WHERE e.student_id = s.id
      AND e.status <> '취소'
);
```

```text
방법 1 결과:1명
방법 2 결과:1명
두 결과가 같은가:같음
찾아진 학생:박서연
```

### 두 방식의 공통 의미를 자신의 말로 설명

```text
둘 다 취소를 제외한 신청이 존재하지 않는 학생을 찾는 방법이다.
```

---

# 7. 기본 집계 검산

다음 결과를 직접 확인합니다.

| 분석 범위 | 예상 건수 | 실제 건수 | 예상 금액 | 실제 금액 | 일치? |
| --- | ---: | ---: | ---: | ---: | --- |
| 전체 신청 | 5 | 5 | 590000 | 590000 | 일치 |
| 활성 신청 | 3 | 3 | 340000 | 340000 | 일치 |
| 취소 제외 | 4 | 4 | 440000 | 440000 | 일치 |
| 취소 | 1 | 1 | 150000 | 150000 | 일치 |

## 7-1. 전체 평균 `recorded_amount`

```text
예상 평균: 118000.00
실제 평균: 118000.00
```

## 7-2. 취소 제외 평균

```text
예상 평균: 110000.00
실제 평균: 110000.00
```

### `recorded_amount`를 실제 회계 매출이라고 부르면 안 되는 이유

```text
recorded_amount는 신청 당시 기록된 금액일 뿐,
실제 결제·환불이 반영된 회계 매출을 뜻하지 않기 때문이다.
```

---

# 8. `GROUP BY`, `HAVING`, `FILTER`

## 8-1. 상태별 신청 건수

```sql
SELECT
    status,
    COUNT(*) AS enrollment_count
FROM course_project.enrollments
GROUP BY status
ORDER BY status;
```

결과:

```text
신청:2
수강중:1
완료:1
취소:1
상태별 합계:5
```

### 상태별 건수 합이 전체 신청 5건과 맞는지 검산

```text
2 + 1 + 1 + 1 = 5로 전체 신청 건수와 일치한다.
```

## 8-2. 강의별 취소 제외 신청 수와 금액

```sql
SELECT
    c.id AS course_id,
    COUNT(e.id) AS enrollment_count,
    COALESCE(SUM(e.recorded_amount), 0) AS recorded_amount
FROM course_project.courses AS c
LEFT JOIN course_project.enrollments AS e
    ON c.id = e.course_id
   AND e.status <> '취소'
GROUP BY c.id
ORDER BY c.id;
```

```text
강의 301: 2건 / 200000
강의 302: 2건 / 240000
강의 303: 0건 / 0
강의별 합계를 다시 더한 값: 440000
전체 취소 제외 기준 440000과 일치 여부: 일치
```

## 8-3. `HAVING` 사용

취소 제외 신청이 2건 이상인 강의를 조회합니다.

```sql
SELECT
    c.id AS course_id,
    COUNT(e.id) AS enrollment_count
FROM course_project.courses AS c
LEFT JOIN course_project.enrollments AS e
    ON c.id = e.course_id
   AND e.status <> '취소'
GROUP BY c.id
HAVING COUNT(e.id) >= 2
ORDER BY c.id;
```

```text
예상 강의 수:2개
실제 강의 수:2개
```

---

# 9. 과대 집계 오류 직접 관찰

강사 201의 강의 가격 합계를 구한다고 가정합니다.

## 9-1. 신청까지 JOIN해서 잘못 집계한 결과

```sql
SELECT
    c.instructor_id,
    SUM(c.price) AS wrong_total_price
FROM course_project.courses AS c
JOIN course_project.enrollments AS e
    ON c.id = e.course_id
WHERE c.instructor_id = 201
GROUP BY c.instructor_id;
```

```text
강사 201 잘못된 가격 합계:440000
```

본문 기준:

```text
440000
```

## 9-2. 강의 수준에서 올바르게 집계

```sql
SELECT
    instructor_id,
    SUM(price) AS correct_total_price
FROM course_project.courses
WHERE instructor_id = 201
GROUP BY instructor_id;
```

```text
강사 201 올바른 가격 합계:220000
```

본문 기준:

```text
220000
```

## 9-3. 왜 두 결과가 달라졌나요?

```text
JOIN 전 강의 행 수:2행
JOIN 후 강의가 반복된 이유:강의마다 여러 신청 행이 연결되었기 때문이다.
SUM이 무엇을 반복해서 더했는가:같은 강의의 price를 신청 건수만큼 반복해서 더했다.
```

### `SUM(DISTINCT c.price)`를 일반적인 해결책으로 사용하면 안 되는 이유

```text
서로 다른 강의가 같은 가격일 경우 하나로 합쳐질 수 있기 때문이다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter08/images/step09_over_aggregation.png
```

`여기에 잘못된 합계와 올바른 합계를 비교한 화면을 삽입하세요.`
![09](images/step09_over_aggregation.png)
---

# 10. 상세 결과 ↔ 집계 결과 교차 검산

강의 하나를 선택합니다.

```text
선택한 course_id: 301
강의 제목: 데이터베이스입문
```

## 10-1. 상세 신청 행 조회

```sql
SELECT
    id,
    student_id,
    course_id,
    status,
    recorded_amount
FROM course_project.enrollments
WHERE course_id = 301
  AND status <> '취소'
ORDER BY id;
```

```text
상세 행 수:2
상세 recorded_amount를 직접 더한 값:200000
```

## 10-2. 집계 SQL

```sql
SELECT
    COUNT(*) AS enrollment_count,
    SUM(recorded_amount) AS total_recorded_amount
FROM course_project.enrollments
WHERE course_id = 301
  AND status <> '취소';
```

```text
집계 건수:2
집계 금액:200000
```

## 10-3. 비교

```text
상세 행 수와 COUNT 결과 일치 여부: 일치
상세 금액 합과 SUM 결과 일치 여부: 일치
다르다면 원인: 없음
```

---

# 11. 자동 완료 게이트

다음을 실행합니다.

```text
code/chapter08/03_join_aggregation_validation.sql
```

```text
최종 검증 메시지:Chapter 08 join and aggregation validation passed
```

기대 메시지:Chapter 08 join and aggregation validation passed

```text
Chapter 08 join and aggregation validation passed
```

### 자동 검증이 통과했어도 사람이 SQL 의미를 설명해야 하는 이유

```text
검증이 통과해도 SQL이 업무 질문의 의미와 범위를 정확히 반영했는지는
사람이 직접 확인해야 하기 때문이다.
```

---

# 12. 개인 프로젝트 업무 질문 3개 만들기

Chapter 07에서 작성한 개인 프로젝트를 사용합니다.

| 질문 ID | 업무 질문 | 결과 한 행 | 포함/제외 범위 | JOIN 경로 | 집계 대상 | 검산 방법 |
| --- | --- | --- | --- | --- | --- | --- |
| P08-Q01 | 국가별 등록된 와인은 몇 개인가? | 국가 한 개 | 등록된 전체 와인 | 국가 → 와인 | 와인 수 | 국가별 와인 상세 목록과 COUNT 비교 |
| P08-Q02 | 포도 품종별 연결된 와인은 몇 개인가? | 품종 한 개 | 등록된 전체 와인 | 품종 → 와인-품종 연결 → 와인 | 와인 수 | 연결 상세 행과 COUNT 비교 |
| P08-Q03 | 와인별 작성된 리뷰는 몇 개인가? | 와인 한 개 | 등록된 전체 리뷰 | 와인 → 리뷰 | 리뷰 수 | 리뷰 상세 행과 COUNT 비교 |
## 12-1. 질문 1 SQL

```sql
-- SQL 초안 / 미실행
SELECT
    c.id,
    c.name,
    COUNT(w.id) AS wine_count
FROM countries AS c
LEFT JOIN wines AS w
    ON c.id = w.country_id
GROUP BY c.id, c.name
ORDER BY c.id;
```

```text
예상 결과: 실제 데이터가 없어 수치 예측 불가
실제 결과: 미실행
검산 결과: PostgreSQL 구현 및 데이터 입력 후 상세 행과 집계 결과를 비교할 예정
```

## 12-2. 질문 2 SQL

```sql
-- SQL 초안 / 미실행
SELECT
    g.id,
    g.name,
    COUNT(wg.wine_id) AS wine_count
FROM grape_varieties AS g
LEFT JOIN wine_grape_varieties AS wg
    ON g.id = wg.grape_variety_id
GROUP BY g.id, g.name
ORDER BY g.id;
```

```text
예상 결과: 실제 데이터가 없어 수치 예측 불가
실제 결과: 미실행
검산 결과: 품종별 연결된 와인 상세 행 수와 COUNT 결과를 비교할 예정
```

## 12-3. 질문 3 SQL

```sql
-- SQL 초안 / 미실행
SELECT
    w.id,
    w.name,
    COUNT(r.id) AS review_count
FROM wines AS w
LEFT JOIN reviews AS r
    ON w.id = r.wine_id
GROUP BY w.id, w.name
ORDER BY w.id;
```

```text
예상 결과: 실제 데이터가 없어 수치 예측 불가
실제 결과: 미실행
검산 결과: 와인별 리뷰 상세 행 수와 COUNT 결과를 비교할 예정
```

> 아직 개인 프로젝트 테이블을 PostgreSQL로 완성하지 않았다면 SQL 초안과 예상 검산 방법까지만 작성하고 `미실행`이라고 명시합니다.

---

# 13. AI를 JOIN·집계 리뷰어로 활용

## 13-1. 내가 AI에게 전달한 질문

```text
와인 검색 개인 프로젝트의 JOIN과 집계 SQL을 검토해 주세요.

결과 한 행의 의미, JOIN 경로, LEFT JOIN 사용 여부,
COUNT 대상, 0건 처리, 과대 집계 위험을 확인해 주세요.

아직 실제 데이터는 입력하지 않아 SQL은 미실행 상태입니다.
```

## 13-2. 내 SQL과 AI SQL 비교

| 검토 항목 | 내 판단/SQL | AI 제안 | 최종 선택 | 이유 |
| --- | --- | --- | --- | --- |
| 결과 한 행 | 국가/품종/와인 한 개 | 기준을 먼저 명확히 정의 | 수용 | 집계 기준이 명확해야 함 |
| 상태 범위 | 실제 데이터가 없어 별도 상태 조건 없음 | 포함/제외 범위를 먼저 정의 | 수용 | 범위에 따라 결과가 달라질 수 있음 |
| JOIN 경로 | PK/FK 관계를 따라 연결 | 임의 컬럼이 아닌 PK/FK 기준으로 JOIN | 수용 | 잘못된 연결을 막기 위해 |
| INNER/LEFT 선택 | 0건도 보여주기 위해 LEFT JOIN 사용 | 0건 부모를 유지하려면 LEFT JOIN 사용 | 수용 | 연결 데이터가 없어도 결과에 남기기 위해 |
| COUNT 대상 | COUNT(*) 대신 자식 테이블 id 사용 | 실제 연결된 자식 id를 COUNT | 수용 | 0건을 정확히 표현하기 위해 |
| 과대 집계 위험 | 여러 1:N JOIN 시 중복 가능성 확인 필요 | 상세 행을 먼저 확인한 뒤 집계 | 수용 | 중복으로 COUNT나 SUM이 커질 수 있음 |
| 상세 검산 방법 | 상세 행 수와 집계 결과 비교 | 상세 조회와 COUNT/SUM 결과를 교차 검산 | 수용 | 집계 오류를 확인하기 위해 |

### AI가 만든 SQL에서 발견한 위험 또는 확인한 점

```text
실제 데이터가 없는 상태에서는 예상 수치를 임의로 작성하면 안 된다.
JOIN 전에 결과 한 행의 기준과 집계 대상을 먼저 정해야 한다.
```

### AI SQL이 실행 성공했다고 바로 정답이라고 할 수 없는 이유

```text
SQL이 실행되어도 JOIN 경로나 집계 기준이 잘못되면
결과가 틀릴 수 있기 때문이다.
```

---

# 14. 최종 성찰

아래 문장은 본인의 말로 작성합니다.

```text
1. JOIN SQL을 작성하기 전에 가장 먼저 정해야 하는 것은
   결과 한 행이 무엇을 의미하는지와 어떤 데이터를 포함할지 정하는 것이다.

2. LEFT JOIN에서 COUNT(*) 대신 COUNT(child.id)를 검토해야 하는 이유는
   COUNT(*)는 자식 데이터가 없어도 부모 행을 셀 수 있지만,
   COUNT(child.id)는 실제 연결된 자식 데이터만 셀 수 있기 때문이다.

3. ON과 WHERE 조건 위치가 중요한 이유는
   ON은 JOIN할 데이터를 제한하고,
   WHERE는 JOIN이 끝난 뒤 결과 행을 제거하기 때문이다.

4. 여러 1:N 관계를 JOIN한 뒤 바로 SUM하면 위험한 이유는
   같은 행이 여러 번 반복되어 실제보다 큰 값으로 집계될 수 있기 때문이다.

5. 집계 결과를 신뢰하기 전에 가장 좋은 검산 방법 중 하나는
   상세 데이터를 직접 조회한 뒤 COUNT나 SUM 결과와 비교하는 것이다.
```

---

# 15. 제출 체크리스트

- [x] `chapter08_answer.md`를 본인 저장소에 만들었다.
- [x] `00_check_course_project.sql`이 통과했다.
- [x] 업무 질문마다 결과 한 행을 먼저 정의했다.
- [x] INNER JOIN과 다중 JOIN을 실행했다.
- [x] LEFT JOIN에서 0건 부모를 확인했다.
- [x] `COUNT(*)`와 `COUNT(child.id)` 차이를 설명했다.
- [x] ON과 WHERE 조건 위치 차이를 직접 비교했다.
- [x] `LEFT JOIN ... IS NULL`과 `NOT EXISTS`를 비교했다.
- [x] 전체/활성/취소 제외 기준값을 직접 검산했다.
- [x] `GROUP BY`, `HAVING`을 사용했다.
- [x] 과대 집계 오류와 수정 결과를 비교했다.
- [x] 상세 결과와 집계 결과를 교차 검산했다.
- [x] `03_join_aggregation_validation.sql`이 통과했다.
- [x] 개인 프로젝트 업무 질문 3개를 작성했다.
- [x] AI SQL을 실행 성공 여부가 아니라 의미와 검산 결과로 평가했다.
- [x] 핵심 캡처는 3~4장 정도만 사용했다.
- [x] 비밀번호·개인정보·비밀정보가 없다.
- [x] GitHub 웹에서 Markdown과 이미지가 정상적으로 보인다.
- [x] 최종 답안을 commit/push했다.

---

# 16. LMS 제출 URL

아래 형식의 **본인 GitHub 파일 URL**을 LMS에 제출합니다.

```text
https://github.com/<본인-GitHub-ID>/<본인-저장소>/blob/main/assignments/chapter08/chapter08_answer.md
```

내 제출 URL:

```text

```

> 저장소 메인 URL, 교수자 템플릿 URL, Raw URL이 아니라 **작성 완료된 본인 `chapter08_answer.md` 파일 화면 URL**을 제출합니다.