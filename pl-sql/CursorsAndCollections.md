# Cursors & Collections 프롤로그

## Cursors Overview

A cursor is a pointer to a private SQL area that stores information about processing a specific SELECT or DML statement.

## Collection Types

PL/SQL has three collection types - associates array, VARRAY (variable-size array), and nested table.

# Implicit Cursor 묵시적 커서

## Implicit Cursors

> An Implicit cursor is a session cursor that is constructed and managed by PL/SQL.
> PL/SQL opens an implicit cursor every time you run a SELECT or DML statement.
> You cannot control an implicit cursor, but you can get imformation from its attributes.

> 묵시적 커서는 PL/SQL에 의해 생성되고 관리되는 세션 커서입니다.
> PL/SQL은 SELECT 또는 DML 문을 실행할 때마다 묵시적 커서를 엽니다.
> 묵시적 커서를 제어할 수는 없지만 해당 속성에서 정보를 얻을 수 있습니다.

- `SQL%ISOPEN` Attribute : Is the Cursor Open?
- `SQL%FOUND` Attribute : Were Any Rows Affected?
- `SQL%NOTFOUND` Attribute : Were No Rows Affected
- `SQL%ROWCOUNT` Attribute : How Many Rows Were Affected?


### 실습

커서를 사용하면 추가/삭제/수정을 했을 때 됐는지 안 됐는지 체크된다.

```sql
declare
  real_order_count number;
begin
  update real_order
  set register_day = sysdate
  where customer_id = 'C003'
  ;
  
  if sql%found then
    dbms_outut.put_line('Update 건수가 있음' || sql%rowcount);
  else
    --
  end if;
  
end;
```

`sql%found`를 하면 수행된 쿼리가 있는지에 대해서만 확인할 수 있다.
`sql%rowcount`는 적용된 행(커서)의 개수를 파악한다.
