# DDL 실습

## 문제 1: 테이블 생성하기 (CREATE TABLE)

### 1. 중복 데이터 확인

`attendance` 테이블은 중복된 데이터가 쌓이는 구조입니다. 중복된 데이터는 어떤 컬럼인가요?

- **답변:** `id` , `nickname`

### 2. 테이블 설계

`attendance` 테이블에서 중복을 제거하기 위해 `crew` 테이블을 어떻게 구성해 볼 수 있을까요?

```sql
CREATE TABLE crew (
    id INT PRIMARY KEY,
    nickname VARCHAR(255) NOT NULL
);
```

### 3. crew 테이블에 들어가야 할 크루들의 정보는 어떻게 추출할까? (hint: DISTINCT)

```sql
INSERT INTO crew (id, nickname)
SELECT DISTINCT crew_id, nickname FROM attendance;
```

## 문제 2: 테이블 컬럼 삭제하기 (ALTER TABLE)

### 1. `attendance`에서 불필요해지는 컬럼은?

- **답변:** `nickname`

### 2. 컬럼을 삭제하려면 어떻게 해야 하는가?

```sql
ALTER TABLE attendance DROP COLUMN nickname;
```

## 문제 3: 외래키 설정하기 (Foreign Key)

### 2. 외래키 설정 SQL

```sql
ALTER TABLE attendance
ADD CONSTRAINT fk_attendance_crew
FOREIGN KEY (crew_id) REFERENCES crew(id);
```

## 문제 4: 유니크 키 설정 (UNIQUE KEY)

### 1. 유니크 키 설정 SQL

```sql
ALTER TABLE crew
ADD CONSTRAINT uk_nickname UNIQUE (nickname);
```

## 문제 5: 크루 닉네임 검색하기 (LIKE)

### 1. 닉네임 검색 조건

- 닉네임의 첫 글자가 '디'로 시작하는 크루를 찾습니다.
- **SQL 패턴:** `LIKE '디%'`

### 2. 크루 검색 SQL

```sql
SELECT * FROM crew
WHERE nickname LIKE '디%';
```

## 문제 6: 출석 기록 확인하기 (SELECT + WHERE)

### 1. 기록 확인 조건

- **대상:** 닉네임이 '어셔'인 크루
- **날짜:** '2025-03-06'
- **내용:** 해당 날짜에 '어셔'의 출석 데이터가 실제로 누락되었는지 조회하여 확인합니다.

### 2. 출석 기록 확인 SQL

```sql
SELECT a.* FROM attendance a
JOIN crew c ON a.crew_id = c.id
WHERE c.nickname = '어셔'
  AND a.attendance_date = '2025-03-06';
```

## 문제 7: 누락된 출석 기록 추가 (INSERT)

### 1. 출석 기록 추가 조건

- **대상:** 닉네임 '어셔' (ID를 직접 넣거나 서브쿼리로 조회)
- **날짜:** '2025-03-06'
- **등교 시간:** '09:31'
- **하교 시간:** '18:01'

### 2. 출석 기록 추가 SQL

```sql
INSERT INTO attendance (crew_id, attendance_date, start_time, end_time)
VALUES (
    (SELECT id FROM crew WHERE nickname = '어셔'),
    '2025-03-06',
    '09:31',
    '18:01'
);
```

## 문제 8: 잘못된 출석 기록 수정 (UPDATE)

### 1. 출석 기록 수정 조건

- **대상:** 닉네임 '주니' (ID 조회가 필요함)
- **날짜:** '2025-03-12'
- **수정 내용:** 기존 등교 시간(start_time)을 '10:05'에서 '10:00'으로 변경

### 2. 출석 기록 수정 SQL

```sql
UPDATE attendance
SET start_time = '10:00'
WHERE crew_id = (SELECT id FROM crew WHERE nickname = '주니')
  AND attendance_date = '2025-03-12';
```

## 문제 9: 허위 출석 기록 삭제 (DELETE)

### 1. 기록 삭제 조건

- **대상:** 닉네임 '아론' (ID 조회가 필요함)
- **날짜:** '2025-03-12'
- **내용:** 해당 날짜에 잘못 입력된 '아론'의 출석 데이터를 테이블에서 영구 삭제합니다.

### 2. 출석 기록 삭제 SQL

```sql
DELETE FROM attendance
WHERE crew_id = (SELECT id FROM crew WHERE nickname = '아론')
  AND attendance_date = '2025-03-12';
```

## 문제 10: 출석 정보 조회하기 (JOIN)

### 1. 출석 정보 통합 조회 SQL

```sql
SELECT
    c.nickname,
    a.attendance_date,
    a.start_time,
    a.end_time
FROM attendance a
JOIN crew c ON a.crew_id = c.id;
```

## 문제 11: nickname으로 쿼리 처리하기 (서브 쿼리)

### 1. 서브 쿼리를 이용한 조회 SQL

```sql
SELECT * FROM attendance
WHERE crew_id = (SELECT id FROM crew WHERE nickname = '검프');
```

## 문제 12: 가장 늦게 하교한 크루 찾기

### 1. 조회 조건

- **날짜:** '2025-03-05'
- **정렬:** 하교 시각(`end_time`) 기준 내림차순(DESC)
- **제한:** 가장 상위 1건만 조회(`LIMIT 1`)

### 2. 하교 시각 조회 SQL

```sql
SELECT c.nickname, a.end_time
FROM attendance a
JOIN crew c ON a.crew_id = c.id
WHERE a.attendance_date = '2025-03-05'
ORDER BY a.end_time DESC
LIMIT 1;
```

## 문제 13: 크루별로 '기록된' 날짜 수 조회

```sql
SELECT c.nickname, COUNT(*) AS 기록된_날짜_수
FROM attendance a
JOIN crew c ON a.crew_id = c.id
GROUP BY c.nickname;
```

## 문제 14: 크루별로 등교 기록이 있는(start_time IS NOT NULL) 날짜 수 조회

```sql
SELECT c.nickname, COUNT(a.start_time) AS 등교_날짜_수
FROM attendance a
JOIN crew c ON a.crew_id = c.id
GROUP BY c.nickname;
```

## 문제 15: 날짜별로 등교한 크루 수 조회

```sql
SELECT attendance_date, COUNT(crew_id) AS 등교_크루_수
FROM attendance
GROUP BY attendance_date;
```

## 문제 16: 크루별 가장 빠른 등교 시각(MIN)과 가장 늦은 등교 시각(MAX)

```sql
SELECT
    c.nickname,
    MIN(a.start_time) AS 가장_빠른_등교,
    MAX(a.start_time) AS 가장_늦은_등교
FROM attendance a
JOIN crew c ON a.crew_id = c.id
GROUP BY c.nickname;
```
