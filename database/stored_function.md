# stored function

## 개념

- 사용자가 정의한 함수
- DBMS에 저장되고 사용되는 함수
- SQL의 select, insert, update, delete statement에서 사용 가능

## 사례 1.

**임직원의 ID를 열 자리 정수로 랜덤하게 발급하고 싶다**

```sql
delimeter $$
CREATE FUNCTION id_generator()
RETURNS int
NO SQL
BEGIN
  RETURN (1000000000 + floor(rand() * 10000000000));
END
$$

delimeter ;
```

> 💡 **NO SQL**
>
> 해당 function이 SQL을 포함하고 있지 있음을 나타낸다.

> 💡 default delimter인 `;`을 그대로 사용하면, create function을 하다가 중간에 `;`를 만났을 때 create function의 구현이 끝났다고 생각할 수 있다. 이를 방지하기 위해 create function 전에 delimeter를 `$$`로 변경하고 시작한다.

**`id_generator()`를 사용해서 임직원 테이블에 정보 추가**

- 임직원 테이블에는 ID, 이름, 생년월일, 성별, 직무, 연봉, 부서의 정보가 필요

```sql
INSERT INTO employee
VALUES(id_generator(), 'JEHN', '1991-08-04', 'F', 'PO', 100000000, 1005);
```

## 사례 2.

**부서 ID를 파라미터로 받으면 해당 부서의 평균 연봉을 알려주는 함수를 작성한다**

```sql
delimeter $$
CREATE FUNCTION dept_avg_salary(d_id int)
RETURNS int
READS SQL DATA
BEGIN
  DECLARE avg_sal int;
  select avg(salar) into avg_sal
                    from employee
                    where dept_id = d_id;
  RETURN avg_sal
END
$$

delimeter ;
```

> 💡 **READS SQL DATA**
>
> 데이터를 읽는 명령문(예: SELECT)이 포함되어 있찌만, 데이터를 쓰는 명령문은 포함되어 있찌 않음을 나타낸다.

```sql
CREATE FUNCTION dept_avg_salary(d_id int)
RETURNS int
READS SQL DATA
BEGIN
  select avg(salar) into @avg_sal
                    from employee
                    where dept_id = d_id;
  RETURN @avg_sal
END
$$
```

> 💡 `DECLARE` 키워드로 변수를 선언하는 것 대신에, `@`를 사용할 수 있다.

**부서 정보와 부서 평균 영봉을 함께 조회**

```sql
SELECT *, dept_avg_salary(id)
FROM department
```

## 사례 3.

**졸업 요건 중 하나인 토익 800점 이상 충족 여부를 알려주는 함수를 작성한다**

```sql
delimeter $$
CREATE FUNCTION toeic_pass_fail(toeic_score int)
RETURNS char(4)
NO SQL
BEGIN
  DECLARE pass_fail char(4);
  IF toeic_scroe is null    THEN SET pass_fail = 'fail';
  ELSEIF toiec_score < 800  THEN SET pass_fail = 'fail';
  ELSE                      THEN SET pass_fail = 'pass';
  END IF;
  RETURN pass_fail;
END
$$

delimeter ;
```

```sql
delimeter $$
CREATE FUNCTION toeic_pass_fail(toeic_score int)
RETURNS char(4)
NO SQL
BEGIN
  IF toeic_scroe is null    THEN SET @pass_fail = 'fail';
  ELSEIF toiec_score < 800  THEN SET @pass_fail = 'fail';
  ELSE                      THEN SET @pass_fail = 'pass';
  END IF;
  RETURN @pass_fail;
END
$$

delimeter ;
```

**학생 정보와 토익 800 이상 통과 여부를 함께 조회**

```sql
SELECT *, toeic_pass_fail(toeic)
FROM student;
```

## stored function 삭제

**DROP FUNCTION stored_function_name;**

## stored function 저장

따로 DB를 명시하지 않은 경우에는 현재 활성화되어 있는 DB에 저장이 된다.

만약 특정 DB에 저장하고 싶다면 create function에서 DB명을 지정한다.

```sql
CREATE FUNCTION db_name.toeic_pass_fail(toeic_score int)
...
```

## stored function 조회

**SHOW CREATE FUNCTION stored_function_name**

**SHOW FUNCTION STATUS where DB = db_name**

🔗 [**SQL에서 stored function을 아시나요?**](https://youtu.be/I1jjR58Rzic?si=djYO-YweQ0N8avlm) 를 통해 공부하였습니다.
