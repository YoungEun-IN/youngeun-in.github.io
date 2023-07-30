# 연관관계 매핑


## 양방향 연관관계

양방향 매핑은 **단방향 매핑에서 반대 방향으로 객체 조회(객체 그래프 탐색) 기능이 추가**된 매핑 방법이다.

아래와 같은 연관 관계가 있다고 가정하자.

- 예제 시나리오
  - 회원과 팀이 있다
  - 회원은 하나의 팀에만 소속될 수 있다.
  - 회원과 팀은 다대일 관계다.

위의 시나리오를 구현하기 위해서 Member(=회원), Team Entity를 작성해보자.

```java
package hello.jpa;

import javax.persistence.*;

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @ManyToOne	// 추가1! Member <-> Team은 N:1 관계
    @JoinColumn(name = "TEAM_ID") // 추가2! 외래키의 이름
    private Team team;

    // GETTER, SETTER 생략
}
```

```java
package hello.jpa;

import javax.persistence.*;

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team") // 반대편 연관관계를 갖는 엔티티의 필드명
    private List<Member> members = new ArrayList<>();

    // GETTER, SETTER 생략
}
```

JPA에서 외래키를 도맡아서 관리하는 하는 클래스의 필드를 연관관계의 주인이라고 한다. **외래키가 있는 쪽이 주인이다.**

{{< admonition note "컬렉션을 필드에서 초기화해야 하는 이유" true >}}
컬렉션은 필드에서 바로 초기화 하는 것이 안전하다.
- null문제에서 안전하다.
- 하이버네이트는 엔티티를 영속화 할때, 컬렉션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다.

만약 임의의 메서드에서 컬렉션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문제가 발생할 수 있다. 따라서 필드레벨에서 생성하는 것이 가장 안전하고, 코드도 간결하다.
{{< /admonition >}}

## 양방향 매핑 규칙
양방향 매핑에는 다음과 같은 규칙이 있다.
- 두 객체의 참조 변수 중 하나를 연관관계의 주인으로 지정
- 연관관계의 주인만이 외래키를 관리(등록, 수정)
- 주인이 아닌쪽은 읽기만 가능
- 주인은 mappedBy 속성 사용 X
- 참고로 @ManyToOne 은 mappedBy 속성 자체가 없음
- 주인이 아니면 mappedBy 속성으로 주인 지정

## 연관관계 편의 메소드

다대일 관계에서 양쪽에 값을 설정할 수 있도록 하는 메소드를 연관관계 편의 메소드라고 한다.

연관관계 편의 메소드 작성시 알아둘 점은 다음과 같다.

- 연관관계 편의 메소드는 관례적인 setXXX 라는 이름 말고 다른 걸 쓰도록 한다.
  - 단순한 setter가 아니라 어떤 중요한 로직이 있다는 것을 인식할 수 있도록 한다.
- 연관관계 편의 메소드는 둘 중 한 군데에서만 작성한다. 무한 루프를 방지하기 위함이다.
- 실제 개발 시 편의 메소드를 둘 중 어디에 둬야될지 상황에 따라 결정하게 된다.

### 연관관계 편의 메소드의 예

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Member {

    @Id
    @Column(name = "member_id")
    private String id;

    private String username;

    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;

    // 연관관계 편의 메소드
    public void setTeam(Team team) {
    
        if(this.team != null) {
            this.team.getMembers().remove(this);
        }
    
        this.team = team;
        team.getMembers().add(this);
    }
    
    //... 생략 ...    
}
```

## 참고
https://www.inflearn.com/course/ORM-JPA-Basic

