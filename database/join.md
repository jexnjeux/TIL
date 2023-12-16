# join

두 개 이상의 table에 있는 데이터를 한 번에 조회하는 것

**ID가 1인 임직원이 속한 부서 이름 조회**

## implicit join

from 절에는 table만 나열하고, where 절에 join condition을 명시하는 방식

### 특징

- old-style join syntax
- where 절에 selection condition과 join condition이 같기 때문에 가독성이 떨어짐

```sql
SELECT D.name
FROM employee E, department D
WHERE E.id = 1 AND E.dept_id = D.id;
```

- 부서 이름 정보는 department 테이블에 있음
- id가 1인 임직원 정보는 employee에 있음

`E.id = 1`인 튜플을 `employee`테이블에서 선택 -> 그 튜플의 `dept_id`와 `department`테이블의 `id`와 연결(join)

## explicit join

from 절에 `JOIN`키워드와 함꼐 joined tables를 명시하는 방식

### 특징

- from절에서 `ON` 뒤에 join condition 명시
- 가독성이 좋음

```sql
SELECT D.name
FROM employee AS E JOIN department AS D ON E.dept_id = D.id
where E.id = 1;
```

## inner join

투 table에서 join condition을 만족하는 tuples로 result table을 만드는 join

### 특징

#### FROM table1 [INNER] JOIN table2 ON join_condition

- join condition에서 사용 가능한 연산자: `=`, `<`, `>`, `!=` 등 비교 연산자 사용 가능
- join condition에서 null 값을 가지는 tuple은 result table에 포함되지 않음

```sql
SELECT *
FROM employee E JOIN department D ON E.dept_id = D.id;
```

## outer join

두 테이블에서 join condition을 만족하지 않는 tuples도 result table에 포함하는 join

#### FROM table1 LEFT [OUTER] JOIN table2 ON join_condition

```sql
SELECT *
FROM employee E LEFT JOIN department D ON E.dept_id = D.id;
```

- left는 `employee`를 의미
- left 테이블에 해당하는 정보를 누락하지 않고 전부 표현

  - join_condition(`E.dept_id = D.id`)을 만족하지 않는 경우 `employee`테이블에 해당하는 attribute는 그대로, `department`에 해당하는 attribute는 null이 됨

#### FROM table1 RIGHT [OUTER] JOIN table2 ON join_condition

```sql
SELECT *
FROM employee E RIGHT JOIN department D ON E.dept_id = D.id;
```

- join_condition에 의해서 매칭되지 않은 right table의 모든 tuple을 표현
  - `department` 테이블에서 매칭되지 않은 모든 tuples도 표현

#### FROM table1 FULL [OUTER] JOIN table2 ON join_condition

```sql
SELECT *
FROM employee E FULL JOIN department D ON E.dept_id = D.id;
```

- `employee`테이블, `department`테이블 둘 다 Join condition에 매칭되지 않는 정보들 모두 표현

> 💡 MySQL은 full join을 지원하지 않는다.

## equi join

join condition에서 `=`(equality comparator)를 사용하는 join

### equi join을 바라보는 시각

1. inner join, outer join 상관없이 `=`를 사용한 join 이라면 `equi join`으로 보는 경우 ⭐️
2. inner join으로 한정해서 `=`를 사용한 경우에 `equi join`으로 보는 경우

```sql
SELECT *
FROM employee E INNER JOIN department D ON E.dept_id = D.dept_id;
```

위 경우에 join하는 attribute의 이름이 같고, 그 attribute에 해당하는 값도 같다.
결과 테이블에서 dept_id가 2번 표시된다(employment의 dept_id, department의 dept_id)

### USING

- 두 테이블이 equi join할 때 join하는 attribute의 이름이 같을 때 작성 가능
- 같은 이름의 attribute는 result table에서 한 번만 표시

```sql
SELECT *
FROM employee E INNER JOIN department D USING(dept_id);
```

## natural join

- 두 table에서 같은 이름을 가지는 모든 attribute pair에 대해 equi join을 수행
- join condition을 따로 명시하지 않음

```sql
SELECT *
FROM employee E NATURAL INNER JOIN department D;
```

employee 테이블과 department 테이블 모두에 dept_id라는 attribute가 있을 때, dept_id를 join condition으로 두 테이블을 join 한다.

> 💡 만약 attribute의 이름은 같으나, 다른 값을 나타내는 경우는 주의해야 한다.
>
> 예를 들어, employee 테이블의 name은 임직원의 이름을 나타내고, department의 name은 부서명을 나타내는 경우, NATURAL JOIN을 사용하게 되면 dept_id뿐만 아니라 name에 대해서도 equi join을 수행하게 된다. 그 결과 아무런 데이터를 조회할 수 없게 된다.

🔗 [**(4부) SQL로 데이터 조회하기**](https://www.youtube.com/watch?v=E-khvKjjVv4&list=PLcXyemr8ZeoREWGhhZi5FZs6cvymjIBVe&index=8) 를 통해 공부하였습니다.
