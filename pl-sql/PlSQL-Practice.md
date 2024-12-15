# 전체 구조 잡기 (w/커서 For Loop, 변수선언 rowtype)

`%type`으로 변수를 정할 수도 있고, rowtype을 활용할 수도 있다.


```sql
DECLARE
    -- 변수선언, 서브 프로그램
    -- v_customer_id varchar2(30) := 'C001';
     v_customer_id customer_info.customer_id%type := 'C001';
--    v_customer_id temp_order.customer_id%type;
    v_menu_id temp_order.menu_id%type;
    v_menu_size temp_order.menu_size%type;
    
    -- 주문서 변수
    v_price real_order.total_price%type:=100;
    v_total_price real_order.total_price%type:=2000;
    v_point_add real_order.point_add%type:=0;
    
    r_real_order real_order%rowtype;
BEGIN
    -- 실행구문
    -- 1. 장바구니의 자료를 가져온다.
    -- 2. 장바구니의 개수만큼 Loop 진행을 한다.
    for fc in (select * from temp_order where customer_id = v_customer_id)
    loop
        v_menu_size := fc.menu_size;
        -- 3. 주문서에 필요한 기본 정보를 가져온다.
        -- 3-a. 맥주일 경우 미성년자 체크를 한다.
        -- 4. 옵션가를 가져온다.
        -- 5. 주문금액을 가져온다. (수량 * (기준단가 + 옵션가))
        r_real_order.price := 200;
        r_real_order.total_price := 9200;
        -- 6. 포인트를 1% 추가한다.
        r_real_order.point_add := 2000;
        -- 7. 주문서를 생성한다.
        insert into real_order (
            order_sequence
          , customer_id
          , menu_id
          , menu_size
          , menu_ice
          , quantity
          , price
          , total_price
          , point_use
          , point_add
        ) values (
            1
          , fc.customer_id
          , fc.menu_id
          , fc.menu_size
          , fc.menu_ice
          , fc.quantity
          , r_real_order.price
          , r_real_order.total_price
          , fc.point_use
          , r_real_order.point_add
        );
    end loop;
    -- 8. 개인정보에 포인트를 넣어준다.
END;
```

# 메뉴 가격 가져오기 (옵션가 체크) / Function 사용

## Function

팡숀은 없을 수도 있는 어떤 입력 값에 대해 처리한 후 Return 값을 반환한다.

```sql
DECLARE
    -- 변수 선언, 서브 프로그램
BEGIN
    -- 실행 구문
    1. 장바구니의 자료를 가져온다.
    2. 장바구니의 개수만큼 loop 진행을 한다.
    3. 주문서에 필요한 기본 정보를 가져온다.
    3-1. 맥주일 경우 미성년자인지 체크한다.
    4. 옵션가를 가져온다.
    5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
    6. 포인트를 1% 추가한다.
    7. 주문서를 생성한다.
    8. 개인정보에 포인트를 넣어준다.
EXCEPTION
    -- 예외처리
END;
```

옵션에 따른 가격을 얻는 팡숀을 만들어보자.

```sql
CREATE OR REPLACE FUNCTION F_GET_OPTION_PRICE 
(
  P_MENU_ID IN VARCHAR2 
, P_MENU_OPTION IN VARCHAR2 
) RETURN NUMBER AS 
-- 변수 선언
v_menu_type menu.menu_type%type;
v_option_price menu_option.option_price%type;
BEGIN

    -- 메뉴 타입 가져오기
    select menu_type
    into v_menu_type
    from menu
    where menu_id = p_menu_id;
    
    -- 옵션가 가져오기
    select option_price
    into v_option_price
    from menu_option
    where menu_type = v_menu_type
      and menu_option = p_menu_option;
  RETURN v_option_price;
END F_GET_OPTION_PRICE;
```

AS-IS 예제는 다음과 같다.

```sql
declare
    v_customer_id customer_info.customer_id%type := 'C001';
    v_menu_id temp_order.menu_id%type;
    v_menu_size temp_order.menu_size%type;
    
    -- 주문서 변수
    v_price real_order.total_price%type := 100;
    v_total_price real_order.total_price%type := 2000;
    v_point_add real_order.point_add%type := 0;
    
    r_real_order real_order%rowtype;
begin
    -- 1. 장바구니의 자료를 가져온다.
    -- 2. 장바구니의 개수만큼 loop 진행한다.
    for fc in (select * from temp_order where customer_id = v_customer_id)
    loop
        v_menu_size := fc.menu_size;
        dbms_output.put_line(fc.customer_id);
        
        -- 3. 주문서에 필요한 기본정보를 가져온다.
        -- 3-1. 맥주일 경우 미성년자인지 체크한다.
        -- 4. 옵션가를 가져온다.
        -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
        
        -- 기준금액
        select x.menu_price
        into r_Real_order.price
        from menu x
        where x.menu_id = fc.menu_id;
        
        -- 사이즈 옵션 가격
        select menu_option.option_price
        into 
        from menu
        inner join menu_option
                on menu.menu_type = menu_option.menu_type
               and menu_option.menu_option = fc.menu_size
        where menu.menu_id = fc.menu_id;
        
        -- 아이스 옵션가격
        select menu_option.option_price
        into 
        from menu
        inner join menu_option
                on menu.menu_type = menu_option.menu_type
               and menu.menu_option = fc.menu_ice
        where menu.menu_id = fc.menu_id;
        
        r_real_order.total_price := 9200;
        -- 5. 포인트를 1% 추가한다.
        r_real_order.point_add := 2000;
        -- 6. 주문서를 생성한다.
        insert into real_order (
            order_sequence
          , customer_id
          , menu_id
          , menu_size
          , menu_ice
          , quantity
          , price
          , total_price
          , point_use
          , point_add
        ) values (
            1
          , fc.customer_id
          , fc.menu_id
          , fc.menu_size
          , fc.menu_ice
          , fc.quantity
          , r_real_order.price
          , r_real_order.total_price
          , fc.point_use
          , r_real_order.point_add
        );
    end loop;
end;
```

이를 함수로 만든 f_get_option_price를 활용하면 다음과 같다.

## f_get_option_price

### identifier 'MENU_ID' must be declared

```sql
declare
    v_customer_id customer_info.customer_id%type := 'C001';
    v_menu_id temp_order.menu_id%type;
    v_menu_size temp_order.menu_size%type;
    
    -- 주문서 변수
    v_price real_order.total_price%type := 100;
    v_total_price real_order.total_price%type := 2000;
    v_point_add real_order.point_add%type := 0;
    
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;
begin
    -- 1. 장바구니의 자료를 가져온다.
    -- 2. 장바구니의 개수만큼 loop 진행한다.
    for fc in (select * from temp_order where customer_id = v_customer_id)
    loop
        v_menu_size := fc.menu_size;
        dbms_output.put_line(fc.customer_id);
        
        -- 3. 주문서에 필요한 기본정보를 가져온다.
        -- 3-1. 맥주일 경우 미성년자인지 체크한다.
        
        -- 기준금액
        select x.menu_price
        into r_real_order.price
        from menu x
        where x.menu_id = fc.menu_id;
        
        -- 4. 옵션가를 가져온다.
        -- 사이즈 옵션 가격
        v_price_size := f_get_option_price(menu_id, menu_size);
        -- 아이스 옵션가격
        v_price_ice := f_get_option_price(menu_id, menu_ice);
        
        -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
        -- 주문 단가
        r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
        
        r_real_order.total_price := r_real_order.price * fc.quantity;
        -- 5. 포인트를 1% 추가한다.
        r_real_order.point_add := 2000;
        -- 6. 주문서를 생성한다.
        insert into real_order (
            order_sequence
          , customer_id
          , menu_id
          , menu_size
          , menu_ice
          , quantity
          , price
          , total_price
          , point_use
          , point_add
        ) values (
            1
          , fc.customer_id
          , fc.menu_id
          , fc.menu_size
          , fc.menu_ice
          , fc.quantity
          , r_real_order.price
          , r_real_order.total_price
          , fc.point_use
          , r_real_order.point_add
        );
    end loop;
end;
```

### no data found

```sql
declare
    v_customer_id customer_info.customer_id%type := 'C001';
    v_menu_id temp_order.menu_id%type;
    v_menu_size temp_order.menu_size%type;
    
    -- 주문서 변수
    v_price real_order.total_price%type := 100;
    v_total_price real_order.total_price%type := 2000;
    v_point_add real_order.point_add%type := 0;
    
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;
begin
    -- 1. 장바구니의 자료를 가져온다.
    -- 2. 장바구니의 개수만큼 loop 진행한다.
    for fc in (select * from temp_order where customer_id = v_customer_id)
    loop
        v_menu_size := fc.menu_size;
        dbms_output.put_line(fc.customer_id);
        
        -- 3. 주문서에 필요한 기본정보를 가져온다.
        -- 3-1. 맥주일 경우 미성년자인지 체크한다.
        
        -- 기준금액
        select x.menu_price
        into r_real_order.price
        from menu x
        where x.menu_id = fc.menu_id;
        
        -- 4. 옵션가를 가져온다.
        -- 사이즈 옵션 가격
        v_price_size := f_get_option_price(fc.menu_id, fc.menu_size);
        -- 아이스 옵션가격
        v_price_ice := f_get_option_price(fc.menu_id, fc.menu_ice);
        
        -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
        -- 주문 단가
        r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
        
        r_real_order.total_price := r_real_order.price * fc.quantity;
        -- 5. 포인트를 1% 추가한다.
        r_real_order.point_add := 2000;
        -- 6. 주문서를 생성한다.
        insert into real_order (
            order_sequence
          , customer_id
          , menu_id
          , menu_size
          , menu_ice
          , quantity
          , price
          , total_price
          , point_use
          , point_add
        ) values (
            1
          , fc.customer_id
          , fc.menu_id
          , fc.menu_size
          , fc.menu_ice
          , fc.quantity
          , r_real_order.price
          , r_real_order.total_price
          , fc.point_use
          , r_real_order.point_add
        );
    end loop;
end;
```

이 select 구문에서는 오류가 안 났지만

```sql
select temp_order.*
  , f_get_option_price(menu_id, menu_size) as option_size_price
  , f_get_option_price(menu_id, menu_ice) as option_ice_price
from temp_order
;
```

PL/SQL 구문에서 팡숀을 통해서 값을 가져올 때는
- select된 데이터가 없던지
- select된 데이터가 두 개 이상이면 오류가 발생한다.

지금은 EXCEPTION으로 처리한다.

```sql
EXCEPTION when others then
  return 0;
```

팡숀을 활용한 TO-BE는 다음과 같다.

```sql
declare
    v_customer_id customer_info.customer_id%type := 'C001';
    v_menu_id temp_order.menu_id%type;
    v_menu_size temp_order.menu_size%type;
    
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;
begin
    -- 1. 장바구니의 자료를 가져온다.
    -- 2. 장바구니의 개수만큼 loop 진행한다.
    for fc in (select * from temp_order)
    loop
        v_menu_size := fc.menu_size;
        dbms_output.put_line(fc.customer_id);
        
        -- 3. 주문서에 필요한 기본정보를 가져온다.
        -- 3-1. 맥주일 경우 미성년자인지 체크한다.
        
        -- 기준금액
        select x.menu_price
        into r_real_order.price
        from menu x
        where x.menu_id = fc.menu_id;
        
        -- 4. 옵션가를 가져온다.
        -- 사이즈 옵션 가격
        v_price_size := f_get_option_price(fc.menu_id, fc.menu_size);
        -- 아이스 옵션가격
        v_price_ice := f_get_option_price(fc.menu_id, fc.menu_ice);
        
        -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
        -- 주문 단가
        r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
        r_real_order.total_price := r_real_order.price * fc.quantity;
        -- 5. 포인트를 10% 추가한다.
        r_real_order.point_add := 2000;
        -- 6. 주문서를 생성한다.
        insert into real_order (
            order_sequence
          , customer_id
          , menu_id
          , menu_size
          , menu_ice
          , quantity
          , price
          , total_price
          , point_use
          , point_add
        ) values (
            1
          , fc.customer_id
          , fc.menu_id
          , fc.menu_size
          , fc.menu_ice
          , fc.quantity
          , r_real_order.price
          , r_real_order.total_price
          , fc.point_use
          , r_real_order.point_add
        );
        -- 7. 개인정보에 포인트를 넣어준다.
    end loop;
end;
```


# Function 생성 - Point 계산하기

## Function

```sql
CREATE OR REPLACE FUNCTION F_GET_ADDPOINT 
(
  P_PRICE IN NUMBER 
) RETURN VARCHAR2 IS -- IS가 아닌 AS로 해도 된다.
BEGIN
  RETURN round(p_price * 0.1);
END F_GET_ADDPOINT;
```

## Point 계산 및 적립

```sql
declare
    -- real_order 테이블 컬럼 값을 변수로 사용
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;
begin
    -- 1. 장바구니의 자료를 가져온다.
    -- 2. 장바구니의 개수만큼 loop 진행한다.
    for fc in (select * from temp_order)
    loop
        -- 3. 주문서에 필요한 기본정보를 가져온다.
        -- 3-1. 맥주일 경우 미성년자인지 체크한다.
        
        -- 기준금액
        select x.menu_price
        into r_real_order.price
        from menu x
        where x.menu_id = fc.menu_id;
        
        -- 4. 옵션가를 가져온다.
        -- 사이즈 옵션 가격
        v_price_size := f_get_option_price(fc.menu_id, fc.menu_size);
        -- 아이스 옵션가격
        v_price_ice := f_get_option_price(fc.menu_id, fc.menu_ice);
        
        -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
        -- 주문 단가
        r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
        r_real_order.total_price := r_real_order.price * fc.quantity;
        -- 5. 포인트를 10% 추가한다.
        r_real_order.point_add := f_get_addpoint(r_real_order.total_price);
        -- 6. 주문서를 생성한다.
        insert into real_order (
            order_sequence
          , customer_id
          , menu_id
          , menu_size
          , menu_ice
          , quantity
          , price
          , total_price
          , point_use
          , point_add
        ) values (
            1
          , fc.customer_id
          , fc.menu_id
          , fc.menu_size
          , fc.menu_ice
          , fc.quantity
          , r_real_order.price
          , r_real_order.total_price
          , fc.point_use
          , r_real_order.point_add
        );
        -- 7. 개인정보에 포인트를 넣어준다.
        update customer_info
        set point = point + r_real_order.point_add
        where customer_id = fc.customer_id;
    end loop;
end;
```