# Chapter 08 확장 실습 답안 템플릿

> **과제:** JOIN과 집계로 서비스 질문에 답하기  
> **사용 방법:** 이 파일을 내려받아 본인의 GitHub 저장소에 `chapter08_answer.md`라는 이름으로 저장한 뒤 실습하면서 바로 작성합니다.  
> **제출 방법:** LMS에는 파일을 직접 업로드하지 않고, **본인 GitHub 저장소의 `chapter08_answer.md` 파일 URL**을 제출합니다.

---

## 제출 전 주의

이 파일과 캡처 화면에는 실제 비밀번호, 전체 DB 접속 URL, API Key, 개인정보를 기록하지 않습니다.

```text
GitHub 계정 또는 별칭: gosaengchung
과제 작성일: 2026-11-03
사용한 AI 도구: GPT 6 Sol
```

---

# 1. Chapter 07 기준 상태 확인

다음을 실행합니다.

```text
code/chapter08/00_check_course_project.sql
```

## 1-1. 사전 검사 결과

```text
검증 메시지: Chapter 08 prerequisite check passed

students 행 수: 3
instructors 행 수: 2
courses 행 수: 3
enrollments 행 수: 5

전체 신청 건수: 5
전체 recorded_amount: 590000
활성 신청 건수: 3
활성 recorded_amount: 340000
취소 제외 신청 건수: 4
취소 제외 recorded_amount: 440000
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
확인 쿼리 예상값과 다르다는 것은 (쿼리에 문제가 없다는 가정하에) 데이터베이스의 구조가 의도대로 설계되지 않았을 가능성이 높으므로 검증을 해야 함
```

### 증거 화면

권장 경로:

```text
assignments/chapter08/images/step01_prerequisite.png
```

`여기에 사전 검사 통과 화면을 삽입하세요.`
![검사 통과 화면](.images/step01_prerequisite.png)
---

# 2. 업무 질문을 SQL보다 먼저 정의하기

다음 세 질문을 각각 SQL 작성 전에 먼저 정의합니다.

## 질문 A

```text
업무 질문: 현재 수강신청을 학생·강의·담당 강사 정보와 함께 확인할 수 있는가?
결과 한 행의 의미: 수강신청 한 건
포함 상태: 신청, 수강중, 완료, 취소 전체
제외 상태: 없음
JOIN할 테이블: enrollments, students, courses, instructors
JOIN 경로: enrollments → students / enrollments → courses → instructors
INNER JOIN / LEFT JOIN 선택: INNER JOIN
그 이유: 각 신청에 학생·강의·담당 강사가 모두 연결된 정상 데이터만 조회한다.
예상 행 수: 5
```

## 질문 B

```text
업무 질문: 수강 취소 건수와 취소 금액 합계는 얼마인가?
결과 한 행의 의미: 취소 이력 건수와 취소 금액 합계
포함 상태: 취소 전체
제외 상태: 신청, 수강중, 완료
JOIN할 테이블: students, enrollments
JOIN 경로: students → enrollments
집계 대상: 학생별 신청 건수, recorded_amount 합계
예상 결과: 1 / 150000
```

## 질문 C

```text
업무 질문: 강의별 수강신청 건수와 신청 기록 금액 합계는 얼마인가?
결과 한 행의 의미: 강의 한 개의 신청 현황
포함 상태: 신청, 수강중, 완료, 취소 전체
제외 상태: 없음
0건인 부모도 보여야 하는가: 예. 신청이 아직 없는 강의도 결과에 포함한다.
NULL을 어떻게 해석할 것인가: 신청이 없어 집계값이 NULL이면 건수와 금액을 0으로 표시한다.
예상 결과:
데이터베이스 입문: 2건 / 200,000원
정규화 실습: 2건 / 240,000원
파이썬 데이터 분석: 1건 / 150,000원
```

---

# 3. INNER JOIN과 다중 JOIN

## 3-1. 신청 한 건마다 학생 이름과 강의 제목 조회

실행 전 예상:

```text
결과 한 행 = id(등록 id), course_id, s_name, c_title 로 구성
예상 행 수 = 5
JOIN 경로 = enrollment <- students, courses
```

내가 실행한 SQL:

```sql
select e.id, e.course_id, s.name as s_name, c.title as c_title
from course_project.enrollments as e
left join course_project.students as s on (e.student_id = s.id)
left join course_project.courses as c on (e.course_id = c.id);
```

실제 결과:

```text
실제 행 수: 5
예상과 일치 여부: 일치함
```

### 학생 이름이 여러 번 보이는 것이 중복 오류가 아닐 수 있는 이유

```text

```

## 3-2. 학생·강의·강사까지 연결

```text
결과 한 행 = id(등록 id), course_id, student_name, course_title, instructor_id 로 구성
강사까지 가는 JOIN 경로 = enrollment <- students, courses / courses <- instructors (각각 id로 연결)
```

```sql
select e.id, e.course_id, s.name as s_name, c.title as c_title, i.name as i_name
from course_project.enrollments as e
left join course_project.students as s on (e.student_id = s.id)
left join course_project.courses as c on (e.course_id = c.id)
left join course_project.instructors as i on (c.instructor_id = i.id)
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

![다중 JOIN 결과 화면](.images/step03_inner_join.png)
---

# 4. LEFT JOIN과 0건 표현

## 4-1. 강의별 취소 제외 신청 수

신청이 없는 강의도 결과에 남도록 작성합니다.

실행 전:

```text
결과 한 행 = 강의 한 개
강의 303의 예상 실제 신청 수 = 1
강의 303의 예상 고유 학생 수 = 1
강의 303의 예상 recorded_amount = 150000
```

내 SQL:

```sql
select course_id, count(*) as enrollment_count, sum(e.recorded_amount) as recorded_amount
from course_project.courses as c
left join course_project.enrollments as e on (e.course_id = c.id)
group by course_id
order by course_id
```

실제 결과:

```text
301	2	200000
302	2	240000
303	1	150000
```

## 4-2. `COUNT(*)`와 `COUNT(e.id)` 비교

강의 303을 기준으로 작성합니다.

```text
COUNT(*) 결과: 2 2 1
COUNT(e.id) 결과: 2 2 1
COUNT(DISTINCT e.student_id) 결과: 2 2 1
```

### 왜 `COUNT(*) = 1`인데 실제 신청 수는 0일 수 있나요?

```text
LEFT JOIN은 신청이 없는 강의도 결과에 남긴다.
이때 강의 행은 결과에 있지만 연결된 신청 열(e.id 등)은 NULL이다.
COUNT(*)는 NULL로 채워진 행도 세므로 1이 나오고, COUNT(e.id)는 NULL을 세지 않으므로 실제 신청 수인 0이 나온다.
```

### 자식 사건 수를 셀 때 `COUNT(child.id)`가 더 적절한 이유

```text
LEFT JOIN 결과에는 자식 행이 없는 부모도 한 행으로 남을 수 있다.
COUNT(*)는 그 부모 행까지 세지만 COUNT(child.id)는 실제로 연결된 자식의 ID만 센다.
따라서 신청 건수처럼 자식 사건 수를 계산할 때는 COUNT(e.id)가 더 적절하다.
```

---

# 5. `LEFT JOIN`에서 `ON`과 `WHERE` 조건 비교

취소 제외 신청만 연결한다고 가정합니다.

## 5-1. 조건을 `ON`에 둔 경우

```sql
SELECT
    s.id,
    s.name,
    e.id AS enrollment_id,
    e.status
FROM course_project.students AS s
LEFT JOIN course_project.enrollments AS e
    ON e.student_id = s.id
   AND e.status <> '취소'
ORDER BY s.id, e.id;
```

```text
결과 학생 수: 3명 (결과 행은 5행)
박서연 포함 여부: 포함됨. 신청 열은 NULL로 표시됨.
```

## 5-2. 조건을 `WHERE`에 둔 경우

```sql
SELECT
    s.id,
    s.name,
    e.id AS enrollment_id,
    e.status
FROM course_project.students AS s
LEFT JOIN course_project.enrollments AS e
    ON e.student_id = s.id
WHERE e.status <> '취소'
ORDER BY s.id, e.id;
```

```text
결과 학생 수: 2명 (결과 행은 4행)
박서연 포함 여부: 제외됨.
```

## 5-3. 차이 설명

```text
ON 조건이 LEFT JOIN의 오른쪽 연결 대상을 제한하는 방식:
학생과 연결할 신청 중 취소 상태를 제외한다. 조건에 맞는 신청이 없어도 왼쪽 테이블인 학생 행은 결과에 남는다.

WHERE 조건이 JOIN 이후 결과 행을 제거하는 방식:
JOIN 결과에서 취소 상태나 NULL 상태인 행을 제거한다. 연결된 신청이 없는 학생도 결과에서 빠진다.

이번 사례에서 ON = 3명, WHERE = 2명이 되는 이유:
박서연에게는 취소된 신청만 있다. ON에 취소 제외 조건을 두면 연결되는 신청은 없어도 LEFT JOIN이 박서연을 남긴다. WHERE에 두면 연결된 신청 상태가 NULL이라 조건을 만족하지 않아 박서연 행이 제거된다.
```

---

# 6. 신청이 없는 학생 찾기 — 두 방법 비교

## 방법 1. `LEFT JOIN ... IS NULL`

```sql
SELECT s.id, s.name
FROM course_project.students AS s
LEFT JOIN course_project.enrollments AS e
    ON (e.student_id = s.id AND e.status <> '취소')
WHERE e.id IS NULL
ORDER BY s.id;
```

## 방법 2. `NOT EXISTS`

```sql
SELECT s.id, s.name
FROM course_project.students AS s
WHERE NOT EXISTS (
    SELECT 1
    FROM course_project.enrollments AS e
    WHERE e.student_id = s.id
      AND e.status <> '취소'
)
ORDER BY s.id;
```

```text
방법 1 결과: 1행
방법 2 결과: 1행
두 결과가 같은가: 예
찾아진 학생: 박서연
```

### 두 방식의 공통 의미를 자신의 말로 설명

```text
두 방식 모두 취소되지 않은 신청이 하나도 없는 학생을 찾는다.
방법 1은 신청과 연결되지 않은 학생 행을 NULL 여부로 확인하고, 
방법 2는 해당 학생의 취소되지 않은 신청이 존재하지 않는지 확인한다.
```

---

# 7. 기본 집계 검산

다음 결과를 직접 확인합니다.

| 분석 범위 | 예상 건수 | 실제 건수 | 예상 금액 | 실제 금액 | 일치? |
|---|---:|---:|---:|---:|---|
| 전체 신청 | 5 | 5 | 590000 | 590000 | 예 |
| 활성 신청 | 3 | 3 | 340000 | 340000 | 예 |
| 취소 제외 | 4 | 4 | 440000 | 440000 | 예 |
| 취소 | 1 | 1 | 150000 | 150000 | 예 |

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
recorded_amount는 신청이 생성될 때 신청 행에 저장한 금액이다.
이 값만으로 실제 결제 승인 여부나 환불 여부를 알 수 없고, 회계상 매출 인식 기준도 반영하지 않는다.
따라서 신청 기록 금액으로 해석해야 한다.
```

---

# 8. `GROUP BY`, `HAVING`, `FILTER`

## 8-1. 상태별 신청 건수

```sql
SELECT status, COUNT(*) AS enrollment_count
FROM course_project.enrollments
GROUP BY status
ORDER BY status
```

결과:

```text
신청: 2건
수강중: 1건
완료: 1건
취소: 1건
상태별 합계: 5건
```

### 상태별 건수 합이 전체 신청 5건과 맞는지 검산

```text
일치한다.
```

## 8-2. 강의별 취소 제외 신청 수와 금액

```sql
SELECT
    c.id AS course_id,
    c.title,
    COUNT(e.id) AS non_cancelled_count,
    COALESCE(SUM(e.recorded_amount), 0) AS non_cancelled_amount
FROM course_project.courses AS c
LEFT JOIN course_project.enrollments AS e
    ON (e.course_id = c.id AND e.status <> '취소')
GROUP BY c.id, c.title
ORDER BY c.id;
```

```text
강의 301: 2건 / 200000원
강의 302: 2건 / 240000원
강의 303: 0건 / 0원
강의별 합계를 다시 더한 값: 4건 / 440000원
전체 취소 제외 기준 440000과 일치 여부: 예
```

## 8-3. `HAVING` 사용

취소 제외 신청이 2건 이상인 강의를 조회합니다.

```sql
SELECT
    c.id AS course_id,
    c.title,
    COUNT(e.id) AS non_cancelled_count
FROM course_project.courses AS c
LEFT JOIN course_project.enrollments AS e
    ON (e.course_id = c.id AND e.status <> '취소')
GROUP BY c.id, c.title
HAVING COUNT(e.id) >= 2
ORDER BY c.id;
```

```text
예상 강의 수: 2개
실제 강의 수: 2개
```

---

# 9. 과대 집계 오류 직접 관찰

강사 201의 강의 가격 합계를 구한다고 가정합니다.

## 9-1. 신청까지 JOIN해서 잘못 집계한 결과

```sql
SELECT SUM(c.price) AS incorrect_price_sum
FROM course_project.courses AS c
JOIN course_project.enrollments AS e ON (e.course_id = c.id)
WHERE c.instructor_id = 201;
```

```text
강사 201 잘못된 가격 합계: 440000
```

본문 기준:

```text
440000
```

## 9-2. 강의 수준에서 올바르게 집계

```sql
SELECT SUM(c.price) AS correct_price_sum
FROM course_project.courses AS c
WHERE c.instructor_id = 201;
```

```text
강사 201 올바른 가격 합계: 220000
```

본문 기준:

```text
220000
```

## 9-3. 왜 두 결과가 달라졌나요?

```text
JOIN 전 강의 행 수: 2행
JOIN 후 강의가 반복된 이유: 강의 301과 302에 각각 신청이 2건씩 있어, 각 강의 행이 신청 건수만큼 반복됐다.
SUM이 무엇을 반복해서 더했는가: 강의 301의 가격 100000원과 강의 302의 가격 120000원을 각각 두 번씩 더했다.
```

### `SUM(DISTINCT c.price)`를 일반적인 해결책으로 사용하면 안 되는 이유

```text
DISTINCT는 강의별로 가격을 한 번씩 세는 것이 아니라, 서로 다른 가격값만 한 번씩 더한다.
예를 들어 강의 두 개의 가격이 모두 100000원이라면 실제 합계는 200000원이지만 SUM(DISTINCT c.price)는 100000원만 반환한다.
따라서 가격값의 중복 제거에 기대기보다, 합산하려는 대상의 수준인 강의 테이블에서 먼저 집계해야 한다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter08/images/step09_over_aggregation.png
```

![잘못된 합계](.images/step09_over_aggregation_1.png)
![올바른 합계](.images/step09_over_aggregation_2.png)
---

# 10. 상세 결과 ↔ 집계 결과 교차 검산

강의 하나를 선택합니다.

```text
선택한 course_id: 302
강의 제목: 정규화 실습
```

## 10-1. 상세 신청 행 조회

```sql
SELECT
    id AS enrollment_id,
    student_id,
    course_id,
    status,
    recorded_amount
FROM course_project.enrollments
WHERE course_id = 302
ORDER BY id;
```

```text
상세 행 수: 2
상세 recorded_amount를 직접 더한 값: 240000
```

## 10-2. 집계 SQL

```sql
SELECT
    COUNT(*) AS enrollment_count,
    SUM(recorded_amount) AS total_recorded_amount
FROM course_project.enrollments
WHERE course_id = 302;
```

```text
집계 건수: 2건
집계 금액: 240000
```

## 10-3. 비교

```text
상세 행 수와 COUNT 결과 일치 여부: 예
상세 금액 합과 SUM 결과 일치 여부: 예
다르다면 원인: 일치함
```

---

# 11. 자동 완료 게이트

다음을 실행합니다.

```text
code/chapter08/03_join_aggregation_validation.sql
```

```text
최종 검증 메시지:
Chapter 08 join and aggregation validation passed
```

기대 메시지:

```text
Chapter 08 join and aggregation validation passed
```

### 자동 검증이 통과했어도 사람이 SQL 의미를 설명해야 하는 이유

```text
해당 검증 쿼리가 잘 구조화되어 있는지 및 실제 업무에서 원하는 결과를 보여주고 있는 것인지를 보장해주지 않기 때문이다.
```

---

# 12. 개인 프로젝트 업무 질문 3개 만들기

Chapter 07에서 작성한 개인 프로젝트를 사용합니다.

| 질문 ID | 업무 질문 | 결과 한 행 | 포함/제외 범위 | JOIN 경로 | 집계 대상 | 검산 방법 |
| --- | --- | --- | --- | --- | --- | --- |
| P08-Q01 |  |  |  |  |  |  |
| P08-Q02 |  |  |  |  |  |  |
| P08-Q03 |  |  |  |  |  |  |

## 12-1. 질문 1 SQL

```sql
SELECT
    m.member_id,
    m.member_name,
    COUNT(pick.pick_record_id) AS pick_record_count
FROM members AS m
LEFT JOIN pick_records AS pick
    ON m.member_id = pick.member_id
GROUP BY
    m.member_id,
    m.member_name
ORDER BY
    m.member_id;
```

```text
예상 결과:
회원별로 한 행씩 나오고, 투자 기록이 없는 회원의 기록 수는 0으로 표시된다.

실제 결과:
미실행

검산 결과:
회원별 pick_records 행을 직접 조회해 센 값과 pick_record_count를 비교한다.
투자 기록이 없는 회원도 결과에 포함되는지 확인한다.
```

## 12-2. 질문 2 SQL

```sql
WITH pick_counts AS (
    SELECT
        stock_id,
        COUNT(*) AS pick_record_count
    FROM pick_records
    GROUP BY stock_id
),
review_counts AS (
    SELECT
        p.stock_id,
        COUNT(r.review_id) AS review_count
    FROM pick_records AS p
    LEFT JOIN reviews AS r
        ON p.pick_record_id = r.pick_record_id
    GROUP BY p.stock_id
)
SELECT
    s.stock_id,
    s.stock_name,
    COALESCE(pc.pick_record_count, 0) AS pick_record_count,
    COALESCE(rc.review_count, 0) AS review_count
FROM stocks AS s
LEFT JOIN pick_counts AS pc
    ON s.stock_id = pc.stock_id
LEFT JOIN review_counts AS rc
    ON s.stock_id = rc.stock_id
ORDER BY
    s.stock_id;
```

```text
예상 결과:
종목별로 한 행씩 나오며, 해당 종목의 투자 기록 수와 복기 기록 수가 각각 표시된다.
기록이 없는 종목의 수는 0으로 표시된다.

실제 결과:
미실행

검산 결과:
종목별 pick_records와 reviews를 각각 따로 조회해 센 값과 비교한다.
투자 기록 수와 복기 기록 수를 한 번에 JOIN해 집계할 때 발생할 수 있는 중복 계산도 확인한다.
```

## 12-3. 질문 3 SQL

```sql
WITH review_counts AS (
    SELECT
        p.member_id,
        COUNT(r.review_id) AS review_count
    FROM pick_records AS p
    LEFT JOIN reviews AS r
        ON p.pick_record_id = r.pick_record_id
    GROUP BY p.member_id
),
pick_counts AS (
    SELECT
        member_id,
        COUNT(*) AS pick_record_count
    FROM pick_records
    GROUP BY member_id
)
SELECT
    m.member_id,
    m.member_name,
    COALESCE(pc.pick_record_count, 0) AS pick_record_count,
    COALESCE(rc.review_count, 0) AS review_count
FROM members AS m
LEFT JOIN pick_counts AS pc
    ON m.member_id = pc.member_id
LEFT JOIN review_counts AS rc
    ON m.member_id = rc.member_id
ORDER BY
    m.member_id;
```

```text
예상 결과:
회원별로 한 행씩 나오며, 투자 기록 수와 복기 기록 수가 표시된다.
투자 기록이나 복기 기록이 없는 회원은 해당 값이 0으로 표시된다.

실제 결과:
미실행

검산 결과:
회원별 pick_records 행 수와 reviews 행 수를 각각 별도로 세어 비교한다.
기록이 없는 회원이 누락되지 않는지 확인한다.
```

> 아직 개인 프로젝트 테이블을 PostgreSQL로 완성하지 않았다면 SQL 초안과 예상 검산 방법까지만 작성하고 `미실행`이라고 명시합니다.

---

# 13. AI를 JOIN·집계 리뷰어로 활용

## 13-1. 내가 AI에게 전달한 질문

```text
픽로그 프로젝트의 회원별 투자 기록 수와 복기 기록 수를 구하는 SQL을 검토해줘.

가정한 테이블은 members(member_id, member_name),
pick_records(pick_record_id, member_id, stock_id),
reviews(review_id, pick_record_id)이야.

각 회원이 결과에 한 행씩 나오고, 투자 기록이나 복기 기록이 없는 회원도 0으로 표시되어야 해.
JOIN으로 기록 수가 중복 집계될 위험이 있는지, COUNT 대상과 LEFT JOIN 사용이 적절한지 설명하고,
상세 행을 이용한 검산 방법도 제안해줘.
```

## 13-2. 내 SQL과 AI SQL 비교

| 검토 항목 | 내 판단/SQL | AI 제안 | 최종 선택 | 이유 |
| --- | --- | --- | --- | --- |
| 결과 한 행 | 회원별 한 행 | 회원을 기준으로 집계 | 회원별 한 행 | 회원 단위 비교가 목적이기 때문 |
| 상태 범위 | 모든 회원 포함 | `members`에서 시작하고 `LEFT JOIN` | 모든 회원 포함 | 기록이 없는 회원도 보여야 함 |
| JOIN 경로 | 회원 → 투자 기록 → 복기 | `members → pick_records → reviews` | 집계 결과를 따로 만든 뒤 회원에 연결 | 여러 관계가 늘어날 때 중복 집계 위험을 줄임 |
| INNER/LEFT 선택 | `LEFT JOIN` | 기준 테이블은 `members`, 하위 데이터는 `LEFT JOIN` | `LEFT JOIN` | 하위 기록이 없어도 회원 행을 보존 |
| COUNT 대상 | 자식 테이블의 ID | `COUNT(review_id)` 또는 사전 집계 | 각 테이블에서 따로 집계 | `COUNT(*)`는 LEFT JOIN에서 빈 자식 행도 셀 수 있음 |
| 과대 집계 위험 | 기록과 복기를 동시에 JOIN하면 확인 필요 | 1:N 관계를 한 번에 JOIN할 때 중복 가능 | 집계 CTE를 분리 | 상세 행이 늘어나면서 다른 값까지 반복될 수 있음 |
| 상세 검산 방법 | 회원 하나를 골라 상세 조회 | 기록과 복기를 따로 조회 | 각 테이블의 행 수를 따로 세어 비교 | 집계 결과가 원본 행과 일치하는지 확인 가능 |

### AI가 만든 SQL에서 발견한 위험 또는 확인한 점

```text
pick_records와 reviews를 한 번에 JOIN하면 한 투자 기록에 복기 기록이 여러 개 있을 때 투자 기록 행이 복기 수만큼 반복될 수 있다.
따라서 투자 기록 수와 복기 기록 수를 각각 따로 집계한 뒤 회원 테이블에 연결하는 편이 안전하다.
LEFT JOIN 결과에서 자식 행 수를 셀 때는 COUNT(*) 대신 자식 테이블의 기본 키를 COUNT해야 한다.
```

### AI SQL이 실행 성공했다고 바로 정답이라고 할 수 없는 이유

```text
SQL이 실행되더라도 JOIN 조건이 잘못되었거나, 결과 행이 중복되었거나, 집계 기준이 질문과 다를 수 있다.
실제 테이블의 기본 키와 외래 키를 확인하고, 일부 회원의 상세 데이터를 직접 조회해 집계 결과와 비교해야 한다.
```

---

# 14. 최종 성찰

아래 문장은 본인의 말로 작성합니다.

```text
1. JOIN SQL을 작성하기 전에 가장 먼저 정해야 하는 것은
   결과를 어떤 기준으로 한 행씩 보여줄지이다.

2. LEFT JOIN에서 COUNT(*) 대신 COUNT(child.id)를 검토해야 하는 이유는
   자식 행이 없어도 만들어지는 부모 행을 COUNT(*)가 셀 수 있기 때문이다.

3. ON과 WHERE 조건 위치가 중요한 이유는
   LEFT JOIN에서 WHERE에 자식 테이블 조건을 두면 자식 행이 없는 부모가 제외될 수 있기 때문이다.

4. 여러 1:N 관계를 JOIN한 뒤 바로 SUM하면 위험한 이유는
   한쪽의 행이 다른 쪽의 여러 행과 결합해 같은 값이 반복 합산될 수 있기 때문이다.

5. 집계 결과를 신뢰하기 전에 가장 좋은 검산 방법 중 하나는
   일부 데이터를 상세 조회해 직접 센 행 수와 더한 금액을 집계 결과와 비교하는 것이다.
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
https://github.com/gosaengchung/ai-database-study/blob/main/assignments/chapter08/chapter08_answer.md
```

> 저장소 메인 URL, 교수자 템플릿 URL, Raw URL이 아니라 **작성 완료된 본인 `chapter08_answer.md` 파일 화면 URL**을 제출합니다.
