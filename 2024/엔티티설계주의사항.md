# 실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발

### 섹션 2. 도메인 분석 설계

### 설계시 지향 point

- 단방향 설계 지향
- 테이블은 Many to Many 구현X(실제로 실무에서 엔티티 설계도  Many to Many XX)
- 외래키(FK)가 있는 곳을 연관관계의 주인으로 설정
- FK를 걸어야해 말아야해? 시스템마다 다름
    - 실시간 트래픽 중요, 잘 운영이 되는 게 중요 → 인덱스를 잘 잡기
    - 돈과 관련되는 등 데이터의 정확성이 맞아야하는 경우 → FK거는 게 더 중요

### 엔티티 설계시 주의점

- 엔티티에는 가급적 Setter 지양
    - Setter가 열려있으면 변경 포인트가 너무 많아 유지보수에 어렵다
    - 엔티티가 어디서 수정되었는지 파악 어려움
    - 나중에 고쳐야 할 때 어디서 set 요청했는지 다 뒤져서 찾아야 함
- 모든 연관관계는 지연 로딩(Lazy)으로 설정(⭐️)
- 모든 연관관계가 기본값인 (Eager)으로 설정되는 경우
    - 즉시로딩: member를 조회할 때 연관된 order도 한번에 모두 조회하는 것(oreder와 관련있는 orderitem도 조회… orderitem과 관련있는 item도 조회…NeverEnding)
    - OneToOne, ManyToOne (@XToOne)관계는 기본이 즉시로딩이므로 직접 지연로딩으로 설정
    - n+1문제
        
        ```jsx
        @Entity
        @Table(name = "orders")
        @Getter @Setter
        public class Order {
        
            @Id @GeneratedValue
            @Column(name = "order_id")
            private Long id;
        
            @ManyToOne(fetch = FetchType.EAGER)
            @JoinColumn(name = "member_id")
            private Member member;
        
        }
        ```
        
        - 단건조회에서는 문제X
        - (JPQL) select o From order o; → (SQL) select * from order
        - order가 100개면 order에 따른 member를 조회하기 위해서 쿼리 100개 날림 → n + 1(첫번째 날린 쿼리가 n개)
- 값타입은 변경 불가능하게 설계해야함
    - @Setter 제거
    - 생성자에서 값을 초기화해서 변경 불가능한 클래스를 만들어야 한다
- 임베디드 타입: 자바 기본 생성자를 protected로 설정하는 걸 권장
- B. JPA가 기본 생성자 필수와 같은 제약을 두는 이유는 JPA 구현 라이브러리가 객체 생성할 때 reflection과 같은 기술을 사용할 수 있도록 지원해야하므로

- `cascade = CascadeType.*ALL*`

- 연관관계 메서드(연관관계 편의 메서드) - 양방향일 때 쓰면 좋다