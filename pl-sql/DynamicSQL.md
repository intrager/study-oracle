# Dynamic SQL 개요

> **Dynamic SQL** is a programming methodology for generating and running SQL statements at run time.

> 동적 SQL은 런타임 시 SQL문을 생성하고 실행하기 위한 프로그래밍 방법론이다.

## Static SQL vs Dynamic SQL

> If it can be done in static SQL, do it in static SQL.

1. Static SQL provides compile time checking. Dynamic SQL does not.
2. Static SQL creates schema object dependencies. Dynamic SQL does not.
3. Dynamic SQL comes with greater security risks.
4. Static SQL usually performs better than dynamic SQL
5. Static SQL is easier to read and maintain than Dynamic SQL.

## dynamic SQL 개념 맛보기 (vs Static SQL)

### Static SQL

```sql
-- 기본 구문
declare
    v_birth customer_info.birth%type;
begin
    -- 생년 가져오기
    select birth
      into v_birth
    from customer_info
    where customer_id = 'C001'
    ;
    dbms_output.put_line('v_birth : ' || v_birth);
end;
```

### Dynamic SQL

```sql
-- 기본 구문
declare
    v_birth customer_info.birth%type;
begin
    -- 생년 가져오기
    execute immediate
       'select birth'
    || 'from customer_info'
    || 'where customer_id = "C001"'
    into v_birth
    ;
    dbms_output.put_line('v_birth : ' || v_birth);
end;
```

변수를 따로 만들어서 그 안에 쿼리문을 집어넣어도 된다. JDBC를 생각하면 쉽다. 다만 아래와 같이 `||`로 바인딩하는 방법은 자주 사용하지는 않는다.

```sql
declare
    v_birth customer_info.birth%type;
    v_query varchar2(1000);
    p_customer_id customer_info.customer_id%type := 'C001';
begin
    v_query := ' select birth from customer_info where customer_id='''||p_customer_id||'''';
    -- 생년 가져오기
    execute immediate v_query
    into v_birth
    ;
end;
```

대신에 `:변수명`을 통해서 변수를 바인딩할 수 있다.

```sql
declare
    v_birth customer_info.birth%type;
    v_query varchar2(1000);
    p_customer_id customer_info.customer_id%type := 'C001';
begin
    -- v_query := ' select birth from customer_info where customer_id='''||p_customer_id||'''';
    v_query := ' select birth from customer_info where customer_id = :cst';

    -- 생년 가져오기
    execute immediate v_query
    into v_birth
    using p_customer_id
    ;
end;
```

`:변수명`으로 할 때는 아래 execute immediate문에서 using을 통해 p_customer_id를 집어넣는다.

#### 변수 두 개 이상을 돌려보고 싶을 때

정적 쿼리로는 변수 한 번 세팅, 쿼리 한 번 세팅, 변수 한 번 세팅, 쿼리 한 번 세팅 총 2쌍씩 구문을 만들어야 한다.

이 구문이 너무 많으면 수정이 필요할 때 어려워서 재활용하고자 동적 쿼리를 응용할 수 있다.

```sql
declare
    v_birth customer_info.birth%type;
    v_query varchar2(1000);
    p_customer_id customer_info.customer_id%type := 'C001';
begin
    -- v_query := 'select birth from customer_info where customer_id='''||p_customer_id||'''';
    v_query := 'select birth from customer_info where customer_id = :cst';

    
    -- 생년 가져오기
    execute immediate v_query into v_birth using p_customer_id;
    dbms_output.put_line(p_customer_id || ' : ' || v_birth);
    
    p_customer_id := 'C002';
    execute immediate v_query into v_birth using p_customer_id;
    dbms_output.put_line(p_customer_id || ' : ' || v_birth);
end;
```

주의할 점은 동적 쿼리의 컬럼, 테이블에는 `:변수`로 세팅하면 안 된다.

```sql
declare
    v_birth customer_info.birth%type;
    v_query varchar2(1000);
    p_customer_id customer_info.customer_id%type := 'C001';
    v_col varchar2(100);
begin
    v_col := 'birth';
    v_query := 'select :v_col from customer_info where customer_id = :cst';
    
    -- 생년 가져오기
    execute immediate v_query into v_birth using v_col, p_customer_id; -- ORA-01722: invalid number
    dbms_output.put_line(p_customer_id || ' : ' || v_birth);
end;
```

런타임 시에 실행한다고 해도, 이 SQL문은 SQL엔진에 가서 실행된다. 이 바인딩은 where 조건에 있는 바인딩은 바인딩이 가능하지만, 컬럼과 테이블, 오라클 DB 내부에서 사용하는 것들은 바인딩할 수 없다.
`||`로 변수를 넣어서 세팅해야 한다.

