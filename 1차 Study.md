## 1. JPA 가 나오게 된 배경
- 애플리케이션 언어 : java, .net, c++... 객체 지향 언어
- 데이터베이스 : MS-SQL, ORACLE, MySQL... 관계형 DB
- 실질적으로 객체를 관계형 DB에 관리 : 언어는 객체 지향 언어이지만 관계형 DB를 관리하기 위해서는 결국 SQL을 사용해야 함.
- SQL 의존적인 개발
- 객체를 테이블에 맞추어서 모델링 -> 객체에 SQL을 통해서 관계형 DB 데이터를 객체에 저장함
- 가져오려는 데이터가 실제로 객체에 들어가 있는지 확실치 않음 => 엔티티 신뢰 문제 발생
- 패러다임의 불일치 : 객체 vs 관계형 데이터 베이스
- 해결책으로 JPA 탄생
  * Java Persistence API (자바 진영의 ORM 기술 표준)
  * ORM, Object-relational mapping( 객체 관계 매핑 )
  * 객체는 객체대로, 관계형 DB는 관계형 DB대로 설계
  * 객체와 관계형 DB 중간에서 매핑
  * 패러다임의 불일치 해결
<br>

## 2. JPA 는 왜 써야 하는가?
- SQL 중심적인 개발에서 객체 중심으로 개발
- 생산성이 좋다
- 유지보수의 장점 (예: 필드 추가)
- 패러다임의 불일치 해결  
- 성능 : 러닝커브가 높은 단점이 있어서 잘 사용하면 성능적 이점, 잘못 사용하면 오히려 마이너스?
- 데이터 접근 추상화와 벤더 독립성, 표준 => 데이터베이스 방언,  https://goodgid.github.io/What-is-Dialect/ 
<br>

## 3. 영속성 컨텍스트란 무엇인가?
- JPA를 이해하는데 가장 중요한 용어.
- 엔티티를 영구 저장하는 환경이라는 뜻. 
- https://velog.io/@seongwon97/Spring-Boot-영속성-컨텍스트Persistence-Context
- 어플리케이션과 DB사이에서 객체를 보관하는 가상의 DB같은 역할을 한다
- 서비스별로 하나의 EntityManager Factory가 존재하며 Entity Manager Factory에서 디비에 접근하는 트랜잭션이 생길 때 마다 쓰레드 별로 Entity Manager를 생성하여 영속성 컨텍스트에 접근
- DB에 객체를 저장한다고 생각할 수 있지만, 실제로는 DB에 저장하는 것이 아니라, 영속성 컨텍스트를 통해서 엔티티를 영속화 한다는 뜻이다. 정확히 말하면 persist() 시점에는 영속성 컨텍스트에 엔티티를 저장한다. DB 저장은 이후이다.

- 비영속
  * 영속성 컨텍스트와 전혀 관계가 없는 상태
  * 엔티티 객체를 생성하였지만 아직 영속성 컨텍스트에 저장하지 않은 상태
- 영속
  * 영속성 컨텍스트에 저장된 상태
  * 엔티티가 영속성 컨텍스트에 의해 관리
  * 영속 상태가 되었다고 바로 DB에 값이 저장되지 않고 트렌젝션의 커밋 시점에 영속성 컨텍스트에 있는 정보들을 DB에 쿼리로 날림
  * entityManager.persist(entity);
- 준영속
  * 영속성 컨텍스트에 저장되었다가 분리된 상태
  * entityManager.detach();
- 삭제
  * 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제
  * entityManager.remove(entity);
```bash
// 객체를 생성한 상태 ( 비영속 )
Member member = new Member();
member.setId( "member1" );
member.setUsername("회원 1");

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

// 객체를 저장한 상태 ( 영속 )
em.persist(member);
``` 
<br>

## 4. 영속성 컨텍스트의 이점
- https://ict-nroo.tistory.com/130

## 5. 플러쉬

