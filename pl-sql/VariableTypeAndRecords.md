# 일반 변수 / ROWTYPE

> 학습목표
>
> Procedure or Function에서 일반변수 / ROWTYPE을 파라미터 / 리턴 값으로 사용할 수 있다.


> 실습
>
> 1. 일반변수를 파라미터와 리턴 값으로 사용한다.
> 2. ROWTYPE 변수를 파라미터와 리턴 값으로 사용한다.
> 3. 위 둘을 Function & Procedure에서 모두 사용해본다.
> 4. Function <-> Procedure를 변환해본다.

## SAMPLE 일반변수 & ROWTYPE 사용

### SP_SECTION_SIX_ONE 프로시저

```sql
CREATE OR REPLACE PROCEDURE SP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;

begin
    p_customer_id := 'C003';

    -- 일반변수 이용
    select name
      into v_name
    from customer_info
    where customer_id = p_customer_id
    ;

    dbms_output.put_line('v_name > ' || v_name);

    -- rowtype 이용
    select *
      into r_customer_info
    from customer_info
    where customer_id = p_customer_id
    ;

    dbms_output.put_line('r_customer_info > ' || r_customer_info.name);
END SP_SECTION_SIX_ONE;
```

파라미터가 없는 프로시저는 바로 사용할 수 있다.

```sql
begin
  sp_section_six_one;
end;
```

### 팡숀 사용 f_get_name1

위 프로시저에 팡숀(`f_get_name1`) 하나 넣어서 사용해보겠다.

이 팡숀은 고객의 아이디를 파라미터로 받아 그의 이름을 조회한 후 반환하는 팡숀이다.

```sql
create or replace function f_get_name1 (
  p_customer_id in customer_info.customer_id%type
) return varchar2
as
  v_name customer_info.name%type;
begin
  select name
    into v_name
  from customer_info
  where customer_id = p_customer_id;
  
  return v_name;
end f_get_name1;
```

### 팡숀 사용 f_get_name2

이번에는 테이블의 컬럼을 직접 참조한 타입이 아닌 rowtype 변수를 만들어볼 것이다.

```sql
CREATE OR REPLACE PROCEDURE SP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;

begin
    p_customer_id := 'C003';

    -- 일반변수 이용
    v_name := f_get_name1(p_customer_id);

    dbms_output.put_line('v_name > ' || v_name);

    -- rowtype 이용
    r_customer_info := f_get_name2(p_customer_info);

    dbms_output.put_line('r_customer_info > ' || r_customer_info.name);
END SP_SECTION_SIX_ONE;
```

프로시저는 이렇게 바꿀 수 있고

```sql
begin
  sp_section_six_one;
end;
```

이렇게 실행하면

```log
오류 보고 -
ORA-01403: no data found
ORA-06512: at "INFLEARN.F_GET_NAME2", line 7
ORA-06512: at "INFLEARN.SP_SECTION_SIX_ONE", line 19
ORA-06512: at line 2
01403. 00000 -  "no data found"
*Cause:    No data was found from the objects.
*Action:   There was no data from the objects which may be due to end of fetch.
```

파라미터로 넣은 p_customer_id 값을 `C003`으로 했는데 해당하는 적당한 값이 없는 듯하다.

p_customer_id를 `C002`로 바꾸고 p_customer_info의 customer_id에다가 `C004`값을 넣고 다시 실행해본다.

```sql
create or replace PROCEDURE SP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;

begin
    p_customer_id := 'C002';

    -- 일반변수 이용
    v_name := f_get_name1(p_customer_id);

    dbms_output.put_line('v_name > ' || v_name);

    -- rowtype 이용
    p_customer_info.customer_id := 'C004';
    r_customer_info := f_get_name2(p_customer_info);

    dbms_output.put_line('r_customer_info > ' || r_customer_info.name);
END SP_SECTION_SIX_ONE;
```

추가한 함수는 아래와 같다.

```sql
create or replace FUNCTION f_get_name2 (
  p_customer_info in customer_info%rowtype
) return customer_info%rowtype
as
r_customer_info customer_info%rowtype;
begin
  select *
    into r_customer_info
  from customer_info
  where customer_id = p_customer_info.customer_id
  ;
  return r_customer_info;
end f_get_name2;
```

### 프로시저 사용 sp_get_name1

```sql
create or replace procedure sp_get_name1 (
    p_customer_id in customer_info.customer_id%type
  , v_name out customer_info.name%type
)
as

begin
  select name
    into v_name
  from customer_info
  where customer_id = p_customer_id;
end sp_get_name1;
```

```sql
create or replace PROCEDURE SP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;

begin
    p_customer_id := 'C002';

    -- 일반변수 이용
    v_name := f_get_name1(p_customer_id);

    dbms_output.put_line('v_name > ' || v_name);

    -- rowtype 이용
    p_customer_info.customer_id := 'C004';
    r_customer_info := f_get_name2(p_customer_info);

    dbms_output.put_line('r_customer_info > ' || r_customer_info.name);
END SP_SECTION_SIX_ONE;
```
