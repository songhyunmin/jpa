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
- https://willseungh0.tistory.com/65
### 1) 1차 캐시
  - 엔티티를 생성후 영속시킴
  ```bash
  Member member = new Member();
  member.setId("member1");
  em.persist(member); // 를 하면 영속성 컨텍스트의 1차 캐시에 저장 된다.
  Member findMember = entityManager.find(Member.class, "member1"); // 조회하면 1차 캐시에서 조회
  ```  
  ![image](https://user-images.githubusercontent.com/45454552/194304007-9310b324-0f8e-414a-9698-dfaa402cc7d7.png)
  
  ```bash
  Member findMember2 =   em . find (Member.class,   "member2" );
  ```
  - 만약에 1차 캐시에 없을 경우, DB에서 직접 조회하고 1차 캐시에 저장 후 반환.
  ![image](https://user-images.githubusercontent.com/45454552/194305363-0ad141fc-6d36-41a8-b187-f91ba2f73b44.png)
  
  - 요청이 시작되면 영속성 컨텍스트를 생성하고, 끝나면 영속성 컨텍스트를 지움. 이때 1차 캐시도 삭제된다.
  - 따라서 애플리케이션 전체에서 공유하는 것이 아님 (애플리케이션 전체에서 공유하는 캐시는 2차 캐시)
### 2) 동일성 보장 (identify)
  ```bash
  Member a = entityManager.find(Member.class, 100L);
  Member b = entityManager.find(Member.class, 100L);
  
  a == b // true
  ```
  - entityManager.find(Member.class, 100L); 을 반복해서 호출해도 영속성 컨텍스트는 1차 캐시에 있는 같은 인스턴스를 반환한다.
  - 이러한 이유로 영속 엔티티의 동일성 즉 == 비교를 보장할 수 있다.
  - 1차 캐시로 반복 가능한 읽기 (REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공.
### 3) 트랜잭션을 지원하는 쓰기 지연 (Transaction Write-behind)
  ```bash
  em.persist(member1);
  em.persist(member2);
  ```
  - 이때 바로 INSERT 쿼리를 데이터베이스에 보내지 않고. 1차 캐시에 저장하면서 동시에 쓰기 지연 SQL 저장소에 쌓아 둔다.
  ```bash
  transaction.commit(); 
  ```
  - 즉 커밋되는 순간 데이터베이스에 INSERT 쿼리를 보냄. (참고로 flush() 호출 시 커밋과 별도로 저장될 수 있음)
### 4) 변경 감지 (Dirty Checking)
  ```bash
  Member member = entityManager.find(Member.class, 10L);
  member.setName("변경 후");

  transaction.commit(); // Update 쿼리가 나감... 왜?
  ```
  ![image](https://user-images.githubusercontent.com/45454552/194308179-3c3a8f0a-44b8-4807-8322-a1a24979c25a.png)

  - JPA는 변경 감지라는 기능으로 엔티티를 변경할 수 있는 기능을 제공한다.
  - JPA는 트랜잭션 되는 순간 내부적으로 flush()가 호출된다.
  - 그때 엔티티와 1차 캐시 내의 스냅샷(최초 상태)을 비교한다.
  - 비교했을 때, 변경이 있을 때, UPDATE SQL를 쓰기 지연 SQL 저장소에 저장.
  - 커밋이 되면 flush()가 호출돼서 DB에 UPDATE 쿼리가 나감.  
### 5) 지연 로딩 (Lazy Loading)
  - 연관 관계 매핑되어 있는 엔티티 조회 시 프록시를 반환함으로써 쿼리를 진짜 필요할 때 날리는 기능 ?
<br>

## 5. 플러쉬
### 플러시의 흐름
- 변경 감지
- 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
- 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송 (등록, 수정, 삭제 쿼리)
- 쉽게 얘기해서 영속성 컨텍스트의 변경 사항들과 데이터베이스를 싱크하는 작업
### 영속성 컨텍스트를 Flush 하는 방법
- em.flush() 로 직접호출
  ```bash
  // 영속
  Member member = new Member(200L, "A");
  em.persist(member);
  
  em.flush();
  
  System.out.println("플러시 직접 호출하면 쿼리가 커밋 전 플러시 호출 시점에 나감");
  
  transaction.commit();
  ```
- 트랜잭션 커밋시 플러시 자동 호출
- JPQL 쿼리 실행하면 플러시 자동 호출
  * JPQL 쿼리 실행시 플러시가 자동으로 호출되는 이유는 아래와 같이 member1,2,3을 영속화한 상태에서 쿼리는 안날라간 상태이면,
  * JPQL로 SELECT 쿼리를 날리려고 하면 저장되어 있는 값이 없어서 문제가 생길 수 있다
  * .JPA는 이런 상황을 방지하고자 JPQL 실행 전에 무조건 flush()로 DB와의 싱크를 맞춘 다음에 JPQL 쿼리를 날리도록 설정 되어 있다.
  * 그래서 아래의 상황에서는 JPQL로 멤버들을 조회할 수 있다.
  ```bash
  em.persist(memberA);
  em.persist(memberA);
  em.persist(memberA);
  
  // 중간에 JPQL 실행
  query = em.createQuery("select m from Member m", Member.class);
  List<Member> members =  query.getResultList();
  ```
- 플러시가 일어나면 1차 캐시가 삭제될까?
  * 삭제 되지 않는다. 쓰기 지연 SQL 저장소에 있는 쿼리들만 DB에 전송되고 1차 캐시는 남아있다.
- 플러시 모드 옵션
  * FlushModeType.AUTO - 커밋이나 쿼리를 실행할 때 플러시 (기본값)
  * FlushModeType.COMMIT - 커밋할 때만 플러시


