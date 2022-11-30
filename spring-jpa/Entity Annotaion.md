> Entity Annotation


# Entity Class

---

```java
import lombok.*;
import javax.persistence.*;
import java.util.*;

@Getter
@ToString(callSuper = true)
@Table(indexes = {
        @Index(columnList = "title"),
})
@EntityListeners(AuditingEntityListener.class)
@Entity
public class Article {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Setter
    @ManyToOne(optional = false)
    @JoinColumn(name = "user_id")
    private User user;  // 유저 정보(ID)

    @Setter
    @Column(nullable = false)
    private String title;  // 제목

    @Setter
    @Column(nullable = false, columnDefinition = "TEXT")
    private String content;  // 본문

    @Setter
    private String hashtag;  // 해시태그

    // mappedBy 를 걸지 않으면 두 entity 이름을 합쳐서 테이블을 하나 만듬
    @ToString.Exclude
    @OrderBy("createdAt DESC")
    @OneToMany(mappedBy = "article", cascade = CascadeType.ALL)
    private final Set<ArticleComment> articleComments = new LinkedHashSet<>();

    @CreateDate @Column(nullable = false) private LocalTime createdTime;

    @CreateBy @Column(nullable = false, length = 100) private String createdBy;

    @LastModifiedDate @Column(nullable = false) private LocalTime updatedTime;

    @LastModifiedBy @Column(nullable = false, length = 100) private String updatedBy;


    // public, protected no-arg constructor -> entity
    protected Article() {
    }

    private Article(User user, String title, String content, String hashtag) {
        this.user = user;
        this.title = title;
        this.content = content;
        this.hashtag = hashtag;
    }

    // factory method
    public static Article of(User user, String title, String content, String hashtag) {
        return new Article(user, title, content, hashtag);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        // Pattern Matching for instanceof in Java 14
        if (!(o instanceof Article article)) return false;
        return id != null && id.equals(article.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }

}
```

## ToStirng

---

- `@ToString` : effective java item 12. toString을 항상 재정의하라
  - 재정의 했다면, `System.out.println(article)`와 같이 작성하면 내부 내용을 볼 수 있으므로 디버깅하기 쉽다.
  - Map 과 같은 경우, 내부에 많은 값을 가지고 있는 `toString()`을 재정의하면 모든 값을 알아내기에 매우 좋다.
  - `callSuper = true (default: false)`: 출력에 슈퍼클래스의 toString 구현 결과를 포함합니다.
  - `@ToString.Exclude`: 연관 관계에 있는 필드에는 exclude annotation을 통해 toString을 제외시킨다. 이는 순환 toString이 발생하면 무한한 메서드 콜이 발생한다.

## ManyToOne

---

## OneToMany

---

```java
// mappedBy 를 걸지 않으면 두 entity 이름을 합쳐서 테이블을 하나 만듬
@ToString.Exclude
@OrderBy("createdAt DESC")
@OneToMany(mappedBy = "article", cascade = CascadeType.ALL)
private final Set<ArticleComment> articleComments = new LinkedHashSet<>();
```

- `@OneToMany(mappedBy = "article", cascade = CascadeType.ALL)`
  - 이는, cascade를 연결할 수도 연결을 안 할수도 있다. 이는 팀 정책마다 달라진다.
  - 또한 FK 제약 때문에 연관관계를 안 걸수도 있다.
- `OrderBy("createdAt desc")`: Collection의 조회 순서를 정할 수 있다. 이는 JPA 쿼리를 작성할 때 유용하다.

### OrderBy 예시

---

```java
Example 1:

     @Entity
     public class Course {
        ...
        @ManyToMany
        @OrderBy("lastname ASC")
        public List<Student> getStudents() {...};
        ...
     }

     Example 2:

     @Entity
     public class Student {
        ...
        @ManyToMany(mappedBy="students")
        @OrderBy // ordering by primary key is assumed
        public List<Course> getCourses() {...};
        ...
     }

     Example 3:

     @Entity
     public class Person {
          ...
        @ElementCollection
        @OrderBy("zipcode.zip, zipcode.plusFour")
        public Set<Address> getResidences() {...};
        ...
     }

     @Embeddable
     public class Address {
        protected String street;
        protected String city;
        protected String state;
        @Embedded protected Zipcode zipcode;
     }

     @Embeddable
     public class Zipcode {
        protected String zip;
        protected String plusFour;
     }
```

## Entity 기본 생성자

---

```java
// public, protected no-arg constructor -> entity
protected Article() {
}
```

- 하이버네이트 구현체 일 때: entity는 항상 protected, public, default 기본 생성자를 열어줘야한다. (필자는 필드 값을 주입하기 위한 것 같다.)

## Entity Equals & HashCode

---

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    // Pattern Matching for instanceof in Java 14
    if (!(o instanceof Article article)) return false;
    return id != null && id.equals(article.id);
}

@Override
public int hashCode() {
    return Objects.hash(id);
}
```

- equals and hashcode
  - 최대한 작성을 안하는 것이 좋지만, 만약 비교하는 상황이 나온다면 idea의 도움을 받아서 작성하는 것이 좋다.
  - 또한, id 필드가 아닌 값을 통해 equals and hashCode를 작성하면, 주의해야 할 점은 JPA Lazy Loading으로 인해 hiberante proxy 값일 때 비교가 일어날 수 있기에 주의해야 한다.
  - [hibernateProxy](https://www.google.com/search?q=hibernate+proxy&oq=hibernate+prox&aqs=chrome.0.0i512j69i57j0i512l4j69i60l2.5143j0j7&sourceid=chrome&ie=UTF-8)
  - [lazy loading jpa](https://www.google.com/search?q=lazy+loading+jpa&sxsrf=ALiCzsZ-X3m5x7qS8wCboqG6IgBeuZZXTw%3A1669776328852&ei=yMOGY7aeMs7v-QbjtZD4DA&ved=0ahUKEwi2orzH8dT7AhXOd94KHeMaBM8Q4dUDCA8&uact=5&oq=lazy+loading+jpa&gs_lcp=Cgxnd3Mtd2l6LXNlcnAQAzIFCAAQgAQyCAgAEIAEEMsBMggIABCABBDLATIICAAQgAQQywEyBggAEAUQHjIGCAAQBRAeMgYIABAIEB4yBggAEAgQHjIGCAAQCBAeMgYIABAIEB46BAgjECc6BAgAEEM6CwguEIAEEMcBENEDOgsIABCABBCxAxCDAToKCAAQsQMQgwEQQzoKCAAQgAQQhwIQFEoECEEYAEoECEYYAFAAWKQkYIclaAJwAHgAgAGGAYgB2AySAQQxLjEzmAEAoAEBwAEB&sclient=gws-wiz-serp)

## Entity 메타데이터

### 1 엔티티 : 1 테이블

```java
@EnableJpaAuditing
@SpringBootApplication
public class BoardToyProjectApplication {
    public static void main(String[] args) {
        SpringApplication.run(BoardToyProjectApplication.class, args);
    }

}


@EntityListeners(AuditingEntityListener.class)
@Entity
public class Article {

    @CreateDate @Column(nullable = false) private LocalTime createdTime;
    @CreateBy @Column(nullable = false, length = 100) private String createdBy;
    @LastModifiedDate @Column(nullable = false) private LocalTime updatedTime;
    @LastModifiedBy @Column(nullable = false, length = 100) private String updatedBy;
}
```

- 코드의 중복을 신경 쓰지 않고 1 엔티티 : 1 테이블로 매핑을 할 때 메타데이터 값을 엔티티 하단에 적는 것 또한 좋은 방법이다.
  - 개별적 대응
  - 코드의 depth가 줄어든다.

### 추출

---

```java
@Getter
@ToString
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public abstract class AuditingFields {
    // 상식적으로 Jpa Auditing 은 not null
    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME)
    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;  // 생성일시

    @CreatedBy
    @Column(nullable = false, updatable = false, length = 100)
    private String createdBy;  // 생성자

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME)
    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime modifiedAt;  // 수정일시

    @LastModifiedBy
    @Column(nullable = false)
    private String modifiedBy;  // 수정자
}


@Getter
@ToString(callSuper = true)
@Table(indexes = {
        @Index(columnList = "title"),
        @Index(columnList = "hashtag"),
        @Index(columnList = "createdAt"),
        @Index(columnList = "createdBy")
})
@Entity
public class Article extends AuditingFields {
    ...
}
```
