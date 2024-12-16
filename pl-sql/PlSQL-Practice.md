# 전체 구조 잡기 (w/커서 For Loop, 변수선언 rowtype)

`%type`으로 변수를 정할 수도 있고, rowtype을 

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

