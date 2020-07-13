
# HelloController Test code작성법

     @RestController
    public class HelloController {

        @GetMapping("/hello")
        public String hello(){
            return "hello";
        }
    }
## 단위테스트 코드
    @RunWith(SpringRunner.class)
    @WebMvcTest(controllers = HelloController.class)
    public class HelloControllerTest {
        @Autowired
        private MockMvc mvc;

        @Test
        public void hello가_리턴된다() throws Exception{
            String hello="hello";
            mvc.perform(get("/hello"))
                    .andExpect(status().isOk())
                    .andExpect(content().string(hello));
        }
    }
### `@RunWith(SpringRunner.class)`
- 테스트를 진행할때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킴
- 스프링부트 테스트와 JUnit사이에 연결자역할
### `@WebMvcTest`
- Web(Spring MVC)에 집중할수있는 애노테이션
- 선언할경우 @Controller,@ControllerAdvice 등을 사용할수있음
- 단, @Service,@Component, @Repository 등은 사용할수없음.
### `private MockMvc mvc`
- 웹 API를 테스트할때 사용
- 스프링MVC테스트의 시작점
- 이클래스를 통해 HTTP GET,POST등에 대한 API테스트를 할수잇음
### `mvc.perform(get("/hello"))`
- MockMvc 를 통해 /hello 주소로 HTTP GET요청을함.
- 체이닝이 지원되어 아래와 같이 여러 검증기능을 이어서 선언할수있음.
### `.andExpect(status().isOk())`
- mvc.perform의 결과를 검증
- HTTP header의 status를검증
### `.andExpect(content().string(hello))`
- mvc.perform의 결과를 검증
- 응답 본문의 내용을검증
- Controller에서 "hello"를 리턴하기에 이값이맞는지검증하기위한. 


----
# 롬복을 사용한 코드
#### 앞서 테스트 코드가 기존작성코드를 지켜주기에 편하게 사용할수있당.
    @Getter
    @RequiredArgsConstructor
    public class HelloResponseDto {
        private final String name;
        private final int mount;
    }
### @Getter
- 선언된 모든 필드의 get메소드를 생성해 줍니다.
### @RequiredArgsConstructor
- 선언된 모든 final필드가 포함된 생성자를 생성해줍니다.
- final이없는 필드는 생성자에 포함하지않는다.
