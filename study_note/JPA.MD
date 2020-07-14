## 객체를 관계형 데이터베이스에서 관리하는것이 무엇보다 중요해지고 있는추세임. :bee:
#### `하지만 관계형 데이터베이스로 객체지향을 표현하기는 쉽지않다.`
##### 관계형 데이터베이스와 객체지향 프로그래밍 언어의 패러다임이 서로 다른데 객체를 데이터베이스에 저장하려하니 여러문제가발생
### 이를 `패러다임 불일치`라고함. :man:
#### 객체지향 프로그래밍에서 부모가되는 객체를 가져오려면 어떻게해야할까?
    User user = findUser();
    Group group=user.getGroup();
##### 이를 보면 User와 Group은 부모-자식관계임을 알수있다. 하지만 여기에 데이터베이스가 추가된다면?
    User user=userDao.findUser();
    Group group=groupDao.findGroup(user.getGroupId());
#### 이처럼 User따로 Group따로 조회하게되어 User와 Group이 어떤관계인지 알수없다.
### 이렇게 웹어플리케이션 개발이 데이터베이스모델링에만 집중하베괴다보니 `JPA`가 문제점해결위해 등장        
## `JPA가 서로 지향하는 바가 다른 2개영역을 중간에서 패러다임 일치를시켜줌.`

### 즉 개발자는 객체지향적 프로그래밍을하고 JPA가 이를관계형 데이터베이스에 맞게 SQL을대신생성실행.
### 더는 SQL에 종속적인 개발을 하지않아도되게끔 된다.

- JPA <- Hibernate <- Spring Data JPA
#### Hibernamte를 쓰는것과 SpringDataJPA를 쓰는것사이 큰차이가 없지만 등장한 이유는 다음과같다.
- 구현체 교체의 용이성
- 저장소 교체의 용이성
##### 먼저 '구현체 교체의 용이성' 이란 Hibernate 외에 다른 구현체로 쉽게 교체하기위함이다. Spring Data JPA내부에서 구현체 매핑을 지원해주기때문.
##### 다음으로 '저장소 교체의 용이성'이란 관계형데이터베이스 외에 다른저장소로 쉽게 교체하기 위함. db를 교체한다면
##### Spring DATA MongoDB로 의존성만 교체하면된다. (이는 Spring Data의 하위프로젝트들은 기본적인 CRUD인터페이스가 같기때문)

#### 프로젝트에 Spring Data Jpa를 적용하려면 아래와같이한다
    //build.gradle에 추가
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('com.h2database:h2')
### spring-boot-starter-data-jpa
- 스프링 부트용 Spring Data Jpa 추상화 라이브러리입니다.
- 스프링 부트 버전에 맞춰 자동으로 JPA관련 라이브러리들의 버전을 관리해 줍니다.
### h2
- 인메모리 관계형 데이터베이스
- 별도의 설치가 필요없이 프로젝트 의존성만으로 관리할수있음
- 메모리에서 실행되기때문에 어플리케이션을 재시작할때마다 초기화한다는점을 이용해 테스트용도로 사용

### Post 클래스
        @Getter
    @NoArgsConstructor
    @Entity
    public class Posts {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        @Column(length = 500, nullable = false)
        private String title;

        @Column(columnDefinition = "TEXT", nullable = false)
        private String content;

        private String author;

        @Builder
        public Posts(String title,String content, String author)
        {
            this.title=title;
            this.content=content;
            this.author=author;
        }
    }


    @Entity는 JPA의 애노테이션이며 @Getter와 @NoArgsConstructor는 롬복의 애노테이션

    Post 클래스는 실제 DB의 테이블과 매칭될 클래스이며 Entity클래스라고한다.
    JPA를 사용하시면 DB데이터에작업할 겨우 실제 쿼리를 날리기보다는 이 Entity클래스의 수정을통해
    작업한다.

## `JPA에서 제공해주는 애노테이션`
### @Entity
- 테이블과 링크될 클래스임을 나타냄.
- 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍(_)으로 테이블 이름을 매칭합니다.
- ex) SalesManager.java -> sales_manager table
### @id
- 해당 테이블의 PK필드를 나타냄
### @GeneratedValue
- PK의 생성규칙을 나타냄
- 스프링부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment가됨.
### @Column
- 테이블의 칼럼을 나타내며 굳이 선언하지않아도 해당클래스의 필드는 모두칼럼이됨
- 사용하는 이유는 , 기본값외에 추가로 변경이 필요한 옵션이 있으면 사용
- 문자열의 경우 VARCHAR(255)가 기본인데 사이즈를 50으로늘리고싶다거나 TEXT로변경하고싶다거나 경우에사용
## `롬복 라이브러리의 애노테이션`
### @NoArgsConstructor
- 기본 생성자 자동 추가
- public Post(){}와 같은 효과
### @Getter
- 클래스 내 모든 필드의 Getter 메소드를 자동생성
### @Builder
- 해당 클래스의 빌더 패턴 클래스를 생성
- 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함
##### 위의 Post클래스에는 Setter메소드가 없다. 자바 빈 규약을 생각하면서 getter/setter를 무작정 생성하는 경우가 있는데
##### 해당 클래스의 인스턴스값들이 언제 어디서 변해야 하는지 코드상으로 명확하게 구분할수없어 차후 기능변경시 복잡해진다.
##### 그래서 Entity클래스에는 절대 Setter메소드를 만들지않는다. 대신 해당 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼수있는 
##### 메소드 추가해야함. 예를들어 주문취소 메소드를 만든다고 가정하고 아래 비교를해보자
    잘못된 사용예시
    public class Oreder{
        public void setStatus(boolean status){
            this.status=status;
        }
    }
    public void 주문서비스_취소이벤트(){
        order.setStatus(false);
    }
---
    올바른 사용 예시
    public class Oreder{
        public void cancelOrder(){
            this.status=false;
        }
        public void 주문서비스의_취소이벤트(){
            order.cancelOrder();
        }
    }
---
##### 여기서 의문이 남는다. Setter가 없는 상황에서 어떻게 값을 채워 DB에 삽입해야할까??
##### 기본적인 구조는 생성자를 통해 최종값을 채운후 DB에 삽입하는 것이며 값변경이 필요한 경우 해당이벤트에 맞는 public메소드를 호출하여
##### 변경하는것을 전제로한다.
##### 생성자 대신에 @Builder를 통해 제공되는 빌더 클래스를 쓰자. 생상저나 빌더나 생성시점에 값을 채워주는 역할은 똑같지만
##### 생성자의 경우 지금 채워야 할 필드가 무엇인지 명확히 지정할수없다.
#### 예를들어 다음생성자가있다면 a와 b위치를 변경해도 코드를 실행하기전까지는 문제찾을수가없다.
    public Example(String a,String b){
        this.a=a;
        this.b=b;
    }
#### `하지만 빌더를 사용한다면 어느필드에 어떤값을 채워야할지 명확하게 인지가능`
    Example.builder()
    .a(a)
    .b(b)
    .build();
#### 이제 Post클래스로 Database를 접근하게 해줄 JpaRepository를 생성해보자
    public interface PostRepository extends JpaRepository<Posts,Long> {
    }

    보통 ibatis나 MyBatis드에서 Dao라고 불리는 DB Layer 접근자임.
    JPA에선 Repository라고 부르며 인터페이스로 생성한다. 단순히 인터페이스를 생성한 후
    JpaRepository<Entity 클래스,PK타입>를 상속하면 기본적인 CRUD메소드가 자동으로 생성됨.
    @Repository를 추가할 필요도없다. 하지만 주의할점은 Entity클래스와 기본 Entity Repository는
    함께 위치해야한다는점이다. 둘은 아주 밀접한 관계이고, Entity 클래스는 기본 Repository없이는
    제대로 역할을 할수가없다.
## 테스트코드 
     @RunWith(SpringRunner.class)
    @SpringBootTest
    public class PostRepositoryTest {
        @Autowired
        PostRepository postRepository;

        @After
        public void cleanup(){
            postRepository.deleteAll();
        }
        @Test
        public void 게시글저장_불러오기(){
            String title="테스트 게시글";
            String content="테스트 본문";
            postRepository.save(Posts.builder()
            .title(title)
            .content(content)
            .author("anima94@naver.com")
            .build());
            List<Posts> postsList=postRepository.findAll();

            Posts posts=postsList.get(0);
            assertThat(posts.getTitle()).isEqualTo(title);
            assertThat(posts.getContent()).isEqualTo(content);
        }
    }
### @After
- Junit에서 단위테스트가 긑날때마다 수행되는 메소드를 지정
- 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기위해 사용
- 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로남아있어 다음테스트 실행시 테스트가 실패할수잇음.
### postsRepository.save
- 테이블 posts에 insert/update 쿼리를 실행한다
- id값이 있다면 update가, 없다면 insert쿼리가 실행됨.
### postsRepository.findAll
- 테이블 posts에 있는 모든 데이터를 조회해오는 메소드입니다.
##### 별다른 설정없이 @SpirngBootTest를 사용할경우 H2데이터베이스를 자동으로 실행해줍니다. 

# 등록/수정/조회 API만들기
#### api를만들기위해 총 3개의 클래스가 필요하다
- Request 데이터를 받을 Dto
- API 요청을 받을 Controller
- 트랜잭션,도메인 기능간의 순서를 보장하는 Service
##### 많은 사람들이 오해하고 있는건 Service에서 비지니스 로직을 처리해야한다는것. `하지만 그렇지않다` Service는 트랜잭션,
##### 도메인 간 순서보장의 역할만한다.
![image](https://user-images.githubusercontent.com/40031858/87391646-1a24f100-c5e6-11ea-9f07-6b7b230342be.png)

```
WebLayer
- 흔히 사용하는 컨트롤러와 JSP/freemarker 등의 뷰 템플릿 영역입니다.
- 이외에도 필터(@Filter),인터셉터,컨트롤러 어드바이스(@ControllerAdvice) 등 외부 요청과
응답에 대한 전반적인 영역을 이야기함.

Service Layer
- @Service에 사용되는 서비스 영역
- 일반적으로 Controller와 Dao의 중간영역에서 사용됩니다.
- @Transactional이 사용되어야 하는 영역이기도함.

Repository Layer
- Database와 같이 데이터 저장소에 접근하는 영역
- Dao(Data Access Object)영역으로 이해하면 쉬움

Dtos
- Dto(Data Transfer Object)는 계층 간에 데이터 교환을 위한 객체를 이야기하며 Dtos는 이들의 영역을
 이야기함.
- 예를 들어 뷰 템프릿엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등이 이들을
이야기함

Domain Model
- 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록
단순화시킨것을 도메인 모델이라고함
- 예를들어 택시앱이라고 하면 배차,탑승,요금등이 모든도메인이될수있음.
- @Entity를 사용한사람이라면 @Entity가 사용된 영역도 도메인모델이라고 이해하면됨.
- 다만 무조건 데이터베이스의 테이블과 관계가 있어야만 하는건아님
- vo처럼 값 객체들도 이영역에 해당하기때문.
```
##### Web,Service,Repository,Dto,Domain 5가지 레이어에서 비지니스 처리를 담당해야하는 곳은 `Domain`이다
#### 기존 서비스로 처리하던 방식을 트랜잭션 스크립트라고 하며 주문 취소 로직을 작성한다면 다음과같다.
    @Transactional
    public Order cancelOrder(int orderId){
        OrdersDto order=ordersDao.selectOrders(orderId);
        BillingDto billing = billingDao.selectBilling(orderId);
        DeliveryDto delivery=deliveryDao.selectDelivery(orderId);

        String deliveryStatus=delivery.getStatus();

        if("IN_PROGRESS".equals(deliveryStatus)){
            delivery.setStatus("CANCEL");
            deliveryDao.update(delivery);
        }

        order.setStatus("CANCEL");
        ordersDao.update(order);

        billing.setStatus("CANCEL");
        deliveryDao.update(billing);

        return order;
    }

##### 모든로직이 서비스 클래스 내부에서 처리된다. 그러다보니 서비스 계층이 무의미하며 객체란 단순히 데이터 덩어리 역할만 하게됨.
##### 반면 도메인모델에서 처리할경우 다음과같은 코드가될수있다.
    @Transactional
    public Order cancelOrder(int orderId){
        Orders order=orderRepository.findById(orderId);
        Billing billing = billingRepository.findByOrderId(orderId);
        Delivery delivery=deliveryRepository.findByOrderId(orderId);

        delivery.cancel();

        order.cancel();
        billing.cancel();
        return order;
    }

    order,billing,delivery가 각자 본인의 취소 이벤트 처리를 하며 서비스 메소드는 트랜잭션과
    도메인 간의 순서만 보장해줌.

#### 이제 등록, 수정, 삭제를 만들어보장.
    @RequiredArgsConstructor
    @RestController
    public class PostsApiController {
        private final PostsService postsService;
        @PostMapping("/api/v1/posts")
        public Long save(@RequestBody PostsSaveRequestDto requestDto){
            return postsService.save(requestDto);
        }
    }
    //////////////////////////////

    @RequiredArgsConstructor
    @Service
    public class PostsService {
        private final PostRepository postRepository;
        @Transactional
        public Long save(PostsSaveRequestDto requestDto){
            return postRepository.save(requestDto.toEntity()).getId();
        }
    }
    //////////////////////////

    @Getter
    @NoArgsConstructor
    public class PostsSaveRequestDto {
        private String title;
        private String content;
        private String author;
        @Builder
        public PostsSaveRequestDto(String title,String content,String author){
            this.title=title;
            this.content=content;
            this.author=author;
        }
        public Posts toEntity(){
            return Posts.builder()
                    .title(title)
                    .content(content)
                    .author(author)
                    .build();
        }
    }
    이후는 프로젝트 코드를보장
