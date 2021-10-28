## 02 JPA 시작

### JPA 프로젝트 구조

<small style="color: grey;">JPA 주요 라이브러리</small>

자바 프로젝트에서 JPA 사용을 위해 라이브러리가 필요하다. 라이브러리는 빌드 도구(메이븐, 그레이들)을 통해 관리할 수 있다.

1. hibernate-entitymanger: JPA 표준과 하이버네이트를 포함한 라이브러리이다.
2. h2: H2 데이터베이스를 사용하기 위한 라이브러리이다.

<small style="color: grey;">JPA 설정</small>

JPA 설정은 `META-INF/persistence.xml` 파일에서 이루어진다. JPA 표준 설정과 하이버네이트 설정으로 구분할 수 있다.

1. JPA 표준 설정
   1. `javax.persistence.jdbc.driver`
   2. `javax.persistence.jdbc.user`
   3. `javax.persistence.jdbc.password`
   4. `javax.persistence.jdbc.url`
2. 하이버네이트 설정
   1. `hibernate.dialect`
   2. `hibernate.show_sql`
   3. `hibernate.format_sql`
   4. `hibernate.use_sql_comments`
   5. `hibernate.id.new_generator_mappings`

<details>
<summary> META-INF/persistence.xml 파일 살펴보기</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
  <persistence-unit name="jpabook">
    <properties>
      <!-- 필수 속성-->
      <property name="javax.persistence.jdbc.driver" value="org.h2.Driver" />
      <property name="javax.persistence.jdbc.user" value="sa" />
      <property name="javax.persistence.jdbc.password" value="" />
      <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/workspaces/jpabook/h2/data/test" />
      <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />

      <!-- 옵션-->
      <property name="hibernate.show_sql" value="true" />
      <property name="hibernate.format_sql" value="true" />
      <property name="hibernate.use_sql_comments" value="true" />
      <property name="hibernate.id.new_generator_mappings" value="true" />      
    </properties>
  </persistence-unit>
</persistence>
```
</details>

<small style="color: grey;">JPA 애플리케이션 개발</small>

JPA 애플리케이션 개발은 관계형 데이터베이스에 있는 테이블을 **객체(엔티티)로 매핑**하는 과정과 매핑된 **객체(엔티티)를 활용**하는 과정으로 구분할 수 있다.

엔티티 매핑 과정은 `javax.persistence` 패키지의 어노테이션을 사용한다. 대표적으로 사용하는 어노테이션으로는 `@Entity`, `@Table`, `@Id`, `@Column`가 있다.

<details>
<summary> 엔티티 매핑: Member.java 파일 살펴보기</summary>

```java
package jpabook.start;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "MEMBER")
public class Member {
    
    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age;

    // Getter, Setter
    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
}
```
</details>

매핑된 엔티티로 로직을 구현하면, 관계형 데이터베이스에 반영되어야 한다. 이는 `EntityManager`의 트랜잭션을 통해 이루어진다. EntityManager는 `persistence.xml` JPA 설정 정보로 만들어진 `EntityManagerFactory`에서 생성한다.

<details>
<summary> 애플리케이션 로직 개발: JpaMain.java 파일 살펴보기</summary>

```java
package jpabook.start;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JpaMain 
{
    public static void main(String[] args) {

        // [엔티티 메니저 팩토리] - 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");

        // [엔티티 매니저] - 생성
        EntityManager em = emf.createEntityManager();

        // [트랜잭션] - 획득
        EntityTransaction tx = em.getTransaction();

        try {
            tx.begin();     // [트랜잭션] - 시작
            logic(em);      // 비즈니스 로직 실행
            tx.commit();    // [트랜잭션] - 커밋
        } catch (Exception e) {
            tx.rollback();  // [트랜잭션] - 롤백
        } finally {
            em.close();     // [엔티티 매니저] - 종료
        }

        emf.close();        // [엔티티 매니저 팩토리] - 종료
    }

    // 비즈니스 로직
    private static void logic(EntityManager em) {

        String id = "id1";
        Member member = new Member();
        member.setId(id);
        member.setUsername("지한");
        member.setAge(2);

        // 등록
        em.persist(member);

        // 수정
        member.setAge(20);

        // 한 건 조회
        Member findMember = em.find(Member.class, id);
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

        // 목록 조회
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
        System.out.println("members.size=" + members.size());

        // 삭제
        em.remove(member);
    }
}
```
</details>