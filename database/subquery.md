# subquery

## 정의

SELECT, INSERT, UPDATE, DELETE에 포함된 query

## 특징

() 안에 기술

## 사례 1

**ID가 14인 임직원보다 생일이 빠른 임직원의 ID, 이름, 생일 조회**

1.ID가 14인 임직원의 생일을 찾기

```sql
SELECT birth_date FROM employee WHERE id=14;
```

2. ID가 14인 직원의 생일보다 생일이 빠른 임직원

```sql
SELECET id, name, birth_date FROM employee
WHERE birth_date < '1992-08-04';
```

3. 두 개의 쿼리를 합치기

```sql
SELECET id, name, birth_date FROM employee
WHERE birth_date < (
  SELECT birth_date FROM employee WHERE id=14
);
```

### subquery(nested query, inner query)

```sql
SELECT birth_date FROM employee WHERE id=14
```

### outer query(main query)

subquery를 포함하는 쿼리

```sql
SELECET id, name, birth_date FROM employee
WHERE birth_date
```

## 사례 2

**ID가 1인 임직원과 같은 부서, 같은 성별인 임직원들의 ID, 이름, 직군 조회**

```sql
SELECT id, name, position
FROM employee
WHERE (dept_id, gender) = (
  SELECT dept_id, gender
  FROM employee
  WHERE id=1
);
```

## 사례 3

**ID가 5인 임직원과 같은 프로젝트에 참여한 임직원들의 ID 조회**

1. ID가 5인 임직원이 참여한 프로젝트 조회

```sql
SELECT proj_id FROM works_on WHERE empl_id=5;
```

2. ID가 5인 임직원을 제외하고, 2001번 프로젝트 또는 2002번 프로젝트(ID가 5인 입직원이 참여한 프로젝트)에 참여한 임직원의 ID

```sql
SELECT DISTINCT empl_id FROM works_on
WHERE empl_id!=5 AND (proj_id=2001 OR proj_id=2002)
```

> 💡 아래 두 쿼리는 같은 의미를 나타낸다.
>
> ```sql
> proj_id=2001 OR proj_id=2002
>
> proj_id IN (2001, 2002)
> ```

> 💡 `DISTINCT`를 사용하는 이유는 어떤 임직원이 2001, 2002 두 개의 프로젝트에 모두 참여했을 수도 있기 때문에 중복을 제거하기 위함이다.

3. 두 개의 쿼리 합치기

```sql
SELECT DISTINCT empl_id FROM works_on
WHERE empl_id!=5 AND proj_id IN (
  SELECT proj_id FROM works_on WHERE empl_id=5
)
```

> 💡 unqalified attribute가 참조하는 table은 해당 attribute가 사용된 query를 포함하여 그 query의 바깥쪽으로 존재하는 뭐든 queries 중, 해당 attribute 이름을 가지는 가장 가까운 table을 참조한다.
> subquery에 있는 proj_id와 empl_id는 outer query에 있는 works_on과 subquery에 있는 works_on 중, subquery에 있는 works_on을 참조한다.

### IN

#### v IN (v1, v2, v3, ...)

- v가 (v1, v2, v3, ...) 중에 하나와 같다면 true를 반환한다.
- (v1, v2, v3, ...)는 명시적인 값들의 집합일 수도 있고, subquery의 결과일 수도 있다.

### NOT IN

#### v NOT IN (v1, v2, v3, ...)

- v가 (V1, v2, v3, ...)의 모든 값과 다르면 true를 반환한다.

## 사례 3

**ID가 5인 임직원과 같은 프로젝트에 참여한 임직원들의 ID와 이름 조회**

- `works_on`테이블에는 `proj_id`, `empl_id`만 있기 때문에 `name`은 알 수 없다.
- `name`은 `employee`테이블에 있다.

```sql
SELECT id, name
FROM employee
WHERE id IN (
  SELECT DISTINCT empl_id FROM works_on
  WHERE empl_id!=5 AND proj_id IN (
    SELECT proj_id FROM works_on WHERE empl_id=5
  )
)
```

```sql
SELECT id, name
FROM employee,
(
  SELECT DISTINCT empl_id
  FROM employee
  WHERE empl_id!=5 AND proj_id IN (
    SELECT proj_id
    FROM works_on
    WHERE empl_id=5
  )
) AS DISTINC_E
WHERE id=DISTINC_E.empl_id
```

subquery의 결과로 가상의 테이블(`DISTINCT_E`)을 만들어 employee와 가상의 테이블을 FROM에 명시한다.

## 사례 4

**ID가 7이거나 12인 임직원이 참여한 프로젝트의 ID와 이름**

```sql
SELECT P.id, P.name
FROM project P
WHERE EXISTS (
  SELECT *
  FROM works_on W
  WHERE W.proj_id=P.id AND W.empl_id IN (7, 12)
)
```

```sql
SELECT P.id, P.name
FROM project P
WEHRE IN (
  SELECT W.proj_id
  FROM wors_on W
  WHERE W.empl_id IN (7, 12)
)
```

> 💡 `EXISTS`와 `IN`은 같은 결과를 위해 사용할 수 있다.

### EXISTS

subquery의 결과로 하나의 row라도 있다면 true를 반환한다.

### NOT EXISTS

subquery의 결과로 단 하나의 row도 없다면 true를 반환한다.

### correlated query

subquery가 outer query의 attribute를 참조할 때(subquery가 `P.id`를 참조), 이 subquery를 correlated query라고 한다.

## 사례 5

**2000년대생이 없는 부서의 ID와 이름 조회**

```sql
SELECT D.id, D.name
FROM department D
WHERE NOT EXISTS (
  SELECT *
  FROM employee E
  WHERE E.dept_id=D.id AND E.birth_date>='2000-01-01'
)
```

- `E.dept_id=D.id`를 통해 해당 부서에 속해있는 임직원을 찾는다.
- `E.birth_date>='2000-01-01'`를 통해 2000년대생 임직원을 찾는다.
- `NOT EXISTS`의 조건에 해당하는 튜플이 없다면 2000년대생이 없다는 것을 의미하므로 해당 department를 선택한다.

```sql
SELECT D.id, D.name
FROM department D
WHERE D.id NOT IN (
  SELECT E.dept_id
  FROM employee E
  WHERE E.birth_date >= '2000-01-01'
);
```

## 사례 6

**리더보다 높은 연봉을 받는 부서원을 가진 리더의 ID, 이름, 연봉 조회**

```sql
SELECT E.id, E.name, E.salary
FROM department D, employee E
WHERE D.leader_id=E.id AND E.salary < ANY (
  SELECT salary
  FROM employee
  WEHRE id <> D.leader_id AND dept_id = E.dept_id
);
```

- subquery에서 현재 부서(`E.dept_id`)에 소속된 직원 중 리더가 아닌 직원의 연봉을 모두 가져온다.
- 부서 내에서 리더가 아닌 직원의 연봉이 리더보다 연봉이 적은 경우를 필터링한다.
  - `ANY`는 subquery에서 반환된 여러 값 중 하나만 만족하면 조건이 참이되도록 하는 비교 연산자이다.

### ANY

#### v comparison_operator ANY (subquery)

subquery가 반환한 결과 중에 단 하라도 v와의 비교 연산이 true라면(하나라도 조건이 만족하면) true를 반환한다.

## 사례 7

**리더보다 높은 연봉을 받는 부서원을 가진 리더의 ID, 이름, 연봉과 해당 부서의 최고연봉 조회**

```sql
SELECT E.id, E.salary,
(
  SELECT max(salary)
  FROM employee
  WHERE dept_id = E.dept_id
) AS dept_max_salary
FROM department D, employee E
WHERE D.leader_id = E.id AND E.salary < ANY (
  SELECT salary
  FROM employee
  WHERE id <> D.leader_id AND dept_id = E.dept_id
);
```

## 사례 8

**ID가 13인 임직원과 한 번도 같은 프로젝트에 참여하지 못한 임직원들의 ID, 이름, 직군 조회**

```sql
SELECT E.id, E.name, E.posiiton
FROM employee E, works_on W
WHERE E.id = W.empl_id and W.proj_id <> ALL (
  SELECT proj_id
  FROM works_on
  WHERE empl_id = 13
);
```

### ALL

#### v comparison_operator ALL (subquery)

subquery가 반환한 결과들과 v와의 비교 연산이 모두 true면 true를 반환한다.

🔗 [**(2부) SQL로 데이터 조회하기**](https://www.youtube.com/watch?v=lwmwlA2WhFc) 를 통해 공부하였습니다.
