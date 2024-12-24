# Trigger 개요

> A trigger is like a stroed procedure that oracle database invokes automatically whenever a specified event occurs.

> 트리거는 지정된 이벤트가 발생할 때마다 오라클 데이터베이스가 호출하는 저장 프로시저와 같다.

- DB 운영자 입장에서 App/DB 개발자로부터 DB를 관리하는 DB 관리의 마지막 보루
- DML 뿐만 아니라 DDL Level까지 통제가 가능

![트리거](trigger.png)

트리거가 필요한 경우/예시는 다음과 같다.
- delete를 해서는 안 되는 테이블이 있다면 프로그램에서 막을 수도 있다. 즉, 테이블 만들 때 제약조건을 넣을 수도 있고, 애플리케이션 단에서 막을 수도 있다.
  - 하지만 이것만 해도 믿을 수 없다면 DB단에서 트리거를 통해 최종적으로 대비할 수 있다.

트리거는 스키마 단까지 가능하다.  
운영할 때 truncate를 테이블에 맘대로 수행하면 안 되는 경우나 drop을 하면 안 된다는 등에 대해 통제할 수 있다.  
또한 트리거는 로그도 남길 수 있다.

