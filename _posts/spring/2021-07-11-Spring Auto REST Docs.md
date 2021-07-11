---
title : "[Spring] Spring Auto REST Docs"

categories :
- Spring

tags :
- Spring 
- Spring REST Docs
- Spring Auto REST Docs
---

처음 적용해보기 때문에 틀린 부분이 있을 수 있습니다.

# 1. 소개
`Spring Auto REST Docs`는 `Spring REST Docs`를 이용하여 문서화를 할 때, 더 적은 코드를 작성하도록 도와주는 라이브러리입니다.

## 1. Spring Auto REST Docs Links
- [Spring Auto REST Docs GitHub](https://github.com/ScaCap/spring-auto-restdocs)
- [Spring Auto REST Docs Reference](https://htmlpreview.github.io/?https://github.com/ScaCap/spring-auto-restdocs/blob/v2.0.11/docs/index.html)
- [Spring Auto REST Docs Samples](https://github.com/ScaCap/spring-auto-restdocs/tree/master/samples)


## 2. Spring REST Docs vs Spring Auto REST Docs
### Spring REST Docs
- 테스트 코드에 작성한 문서 정보들을 이용하여 문서화를 합니다. 테스트 코드에만 문서화 코드가 작성되므로, 프로덕션 코드의 변경이 없습니다.
- 작성해야 할 코드가 상당히 많습니다. 코드를 작성하다가 '엑셀 혹은 마크다운으로 했으면 더 빨랐겠다.' 라고 생각이 들었습니다. 또, 테스트 코드가 너무 길어져서 어떤 테스트를 하고 싶은 건지 알기 어려워집니다.
  
### Spring Auto REST Docs
- 프로덕션 코드에 작성한 Javadoc을 이용하여 문서화를 합니다.
- 프로덕션 코드에 Javadoc이 추가되므로, 다른 사람이 코드를 읽었을 때, 더 쉽게 이해할 수 있도록 도와줍니다. 하지만 주석이 길어진다면, 코드를 읽을 때 불편함이 생깁니다.
- Spring REST Docs에서는 없는 기능들이 제공되어, 같은 문서라도 Spring REST Docs보다 더 쉽게 문서를 작성할 수 있습니다.

## 3. Spring Auto REST Docs 도입 이유
1. Javadoc을 무조건 작성해야 합니다. Javadoc을 작성하면, 다른 사람들이 코드를 읽을 때 더 이해하기 쉽습니다.
2. Spring REST Docs보다 작성해야 하는 코드의 양이 적어 생산성이 올라가고, 다른 문서화 코드가 없어 테스트 메소드가 무엇을 하는지를 명확히 할 수 있습니다.

# 2. 적용
## 1. 환경
- Spring Boot 2.5.2
- Gradle 7.0.2
- Java 8
- JUnit5

## 2. build.gradle
다음은 `Spring Auto REST Docs`를 적용한 `build.gradle` 입니다.

```groovy
plugins{
    id 'org.springframework.boot' version '2.5.2'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id "org.asciidoctor.jvm.convert" version "3.3.2"
}

group = 'com.wootech'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
    mavenCentral()
}

configurations {
    jsondoclet
}

ext {
    snippetsDir = file('build/generated-snippets') // snippet 이 생성될 디렉토리 지정
    javadocJsonDir = file("$buildDir/generated-javadoc-json") // javadoc-json 이 생성될 디렉토리 지정
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // log
    implementation 'net.rakugakibox.spring.boot:logback-access-spring-boot-starter:2.7.1'

    runtimeOnly 'mysql:mysql-connector-java'
    testImplementation 'com.h2database:h2:1.4.199'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'

    // Spring Auto REST Docs
    testImplementation 'capital.scalable:spring-auto-restdocs-core:2.0.11'
    jsondoclet "capital.scalable:spring-auto-restdocs-json-doclet:2.0.11"
}

// Java Source 코드에서 Javadoc 을 읽고, json 파일로 반환
task jsonDoclet(type: Javadoc, dependsOn: compileJava) {
    source = sourceSets.main.allJava
    classpath = sourceSets.main.compileClasspath
    destinationDir = javadocJsonDir
    options.docletpath = configurations.jsondoclet.files.asType(List)
    options.doclet = 'capital.scalable.restdocs.jsondoclet.ExtractDocumentationAsJsonDoclet'
    options.memberLevel = JavadocMemberLevel.PACKAGE
}

// 테스트 실행 후, snipperDir에 snippet(.adoc 파일들) 생성 
test {
    systemProperty 'org.springframework.restdocs.outputDir', snippetsDir
    systemProperty 'org.springframework.restdocs.javadocJsonDir', javadocJsonDir

    dependsOn jsonDoclet
    useJUnitPlatform()

    outputs.dir snippetsDir
}

// src/docs/asciidoc/*.adoc 파일을 build/docs/asciidoc에 *.html 파일로 변환  
asciidoctor {
    inputs.dir snippetsDir
    dependsOn test
}

asciidoctor.doFirst {
    delete file('src/main/resources/static/docs')
}

// build/docs/asciidoc 파일을 src/main/resources/static/docs 패키지로 이동
task copyDocument(type: Copy) {
    dependsOn asciidoctor
    from file("build/docs/asciidoc")
    into file("src/main/resources/static/docs")
}

// assemble을 copyDocument에 의존하게 하여, assemble보다 copyDocument가 먼저 실행되도록 함
// jar 파일은 copyDocument가 끝나야 생성이 됨
assemble {
    dependsOn copyDocument
}

```

다음은 Task들의 소개입니다.
1. `jsonDoclet` : Java 코드에 작성되어 있는 Javadoc 을 읽어 json 파일을 생성합니다.
2. `test` : Test 를 실행하면서 json 파일을 바탕으로 snippet 들을 생성합니다.
3. `asciidoctor` : 작성한 adoc 파일을 HTML 파일로 변환합니다.
4. `copyDocument` : 변환한 HTML 파일을 정적 패키지로 이동시킵니다.

다음은 `build` 시 task flow입니다.

![flow](https://user-images.githubusercontent.com/56301069/125194154-107ea980-e28b-11eb-8122-c3486549fb56.jpg)

## 3. javadoc 작성
`Spring Auto REST Docs`는 `javadoc` 을 읽어 문서화를 하기 때문에, 프로덕션 코드에 `javadoc`을 작성해줍시다.

다음은 간단한 Controlle의 코드입니다.

- `@param` 어노테이션 이전까지 작성한 문자열들은 `auto-description.adoc` 파일에 작성됩니다. `description` 에는 `<p>`, `<ul>`, `<li>` 태그가 사용 가능합니다.
- `@param` 어노테이션에 작성한 문자열들은 `auto-request-paramters.adoc` 파일에 작성됩니다.
- `@RequestParam` 어노테이션의 `required` 옵션을 `false`로 설정하면, `auto-request-paramters.adoc` 파일에서 `Optional`이 `false`로 작성됩니다. 이 때 클래스의 타입은 `int`와 같은 `primitive` 타입이면 안됩니다.


```java
@RestController
public class MemberController {

    /**
     * 리뷰어 목록을 조회한다.
     *
     * @param skills 선생님 기술 스택
     * @param career 선생님 경력 연차
     * @param limit 한 페이지에 보여줄 선생님 수
     * @param page 현재 페이지 번호
     * @return return
     */
    @GetMapping("/teachers")
    public ResponseEntity<Void> findAllTeacher(
            @RequestParam(required = false) List<String> skills,
            @RequestParam(required = false) Integer career,
            @RequestParam Integer limit,
            @RequestParam int page) {
        return ResponseEntity.ok().build();
    }


    /**
     * 리뷰어를 등록한다.
     */
    @PostMapping(value = "/teachers")
    public ResponseEntity<Void> registerTeacher(
            @Valid @RequestBody TeacherRegistrationRequest teacherRegistrationRequest) {
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

다음은 Controller에서 사용하는 DTO입니다.

`Spring Auto REST Docs`는 객체의 필드 및 제약 조건(constraints)도 작성해줍니다.
필드마다 설명과 제약 조건을 작성해줍니다.

`@NotNull`, `@NotEmpty`, `@NotBlank` 어노테이션을 추가하면, 스니펫에 제약 조건에 대한 설명이 작성되는 것이 아니라 `Optional` 컬럼에 `false` 값이 작성됩니다. 

```java
import java.util.List;
import javax.validation.constraints.*;

public class TeacherRegistrationRequest {

    /**
     * 선생님 기술 경력
     */
    @NotEmpty
    private List<String> skills;

    /**
     * 선생님 연차
     */
    @NotNull
    @PositiveOrZero
    private Integer career;

    /**
     * 선생님 자기 소개 제목
     */
    @NotBlank
    private String title;

    /**
     * 선생님 자기 소개 내용
     */
    @NotBlank
    private String content;

    public TeacherRegistrationRequest() {
    }

    public TeacherRegistrationRequest(List<String> skills, int career, String title, String content) {
        this.skills = skills;
        this.career = career;
        this.title = title;
        this.content = content;
    }

    public List<String> getSkills() {
        return skills;
    }

    public int getCareer() {
        return career;
    }

    public String getTitle() {
        return title;
    }

    public String getContent() {
        return content;
    }
}
```

## 4. 테스트 코드 작성
다음은 테스트 코드에서 사용하는 MockMvc에 문서화 설정을 추가하는 코드입니다. 테스트 패키지에 다음과 같은 클래스를 추가합니다.

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.wootech.dropthecode.exception.GlobalExceptionHandler;

import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.restdocs.RestDocumentationContextProvider;
import org.springframework.restdocs.http.HttpDocumentation;
import org.springframework.restdocs.mockmvc.MockMvcRestDocumentation;
import org.springframework.restdocs.mockmvc.RestDocumentationResultHandler;
import org.springframework.restdocs.snippet.Snippet;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.filter.CharacterEncodingFilter;

import capital.scalable.restdocs.AutoDocumentation;

import static capital.scalable.restdocs.SnippetRegistry.*;
import static capital.scalable.restdocs.jackson.JacksonResultHandlers.prepareJackson;
import static capital.scalable.restdocs.response.ResponseModifyingPreprocessors.limitJsonArrayLength;
import static capital.scalable.restdocs.response.ResponseModifyingPreprocessors.replaceBinaryContent;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.documentationConfiguration;
import static org.springframework.restdocs.operation.preprocess.Preprocessors.*;

public class RestDocsMockMvcUtils {

    private static final String PUBLIC_AUTHORIZATION = "Resource is public.";
    public static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

    @TestConfiguration
    private static class Advice {
        
        // 필요한 ControllerAdvice 가 있다면 여기에 빈으로 등록합니다.
        @Bean
        public static GlobalExceptionHandler handler() {
            return new GlobalExceptionHandler();
        }
        
        // ContentType을 applicaiton/json;charset=UTF-8 로 설정하기 위해 인코딩 필터롤 추가합니다.
        @Bean
        public static CharacterEncodingFilter filter() {
            return new CharacterEncodingFilter("UTF-8", true);
        }
    }

    public static MockMvc successRestDocsMockMvc(RestDocumentationContextProvider provider, Object... controllers) {
        return MockMvcBuilders.standaloneSetup(controllers) // 문서화 할 컨트롤러들만을 빈으로 등록합니다.
                              .addFilters(Advice.filter())
                              .setControllerAdvice(Advice.handler())
                              .alwaysDo(prepareJackson(OBJECT_MAPPER))
                              .alwaysDo(restDocumentation())
                              .apply(documentationConfiguration(provider)
                                      .uris()
                                      .withScheme("http")
                                      .withHost("localhost")
                                      .withPort(8080)
                                      .and()
                                      .snippets()
                                      
                                      // 생성할 스니펫들을 정의합니다.
                                      .withDefaults(
                                              HttpDocumentation.httpRequest(),
                                              HttpDocumentation.httpResponse(),
                                              AutoDocumentation.requestFields(),
                                              AutoDocumentation.responseFields(),
                                              AutoDocumentation.pathParameters(),
                                              AutoDocumentation.requestParameters(),
                                              AutoDocumentation.description(),
                                              AutoDocumentation.methodAndPath(),
                                              AutoDocumentation.sectionBuilder()
                                                               
                                                               // section 스니펫에 추가할 스니펫들
                                                               // section 스니펫에 아래 작성한 스니펫 순서대로 작성됩니다.
                                                               .snippetNames(
                                                                       AUTO_METHOD_PATH,
                                                                       AUTO_DESCRIPTION,
                                                                       AUTO_AUTHORIZATION,
                                                                       AUTO_PATH_PARAMETERS,
                                                                       AUTO_REQUEST_PARAMETERS,
                                                                       AUTO_REQUEST_FIELDS,
                                                                       AUTO_RESPONSE_FIELDS,
                                                                       HTTP_REQUEST,
                                                                       HTTP_RESPONSE
                                                               )
                                                               .skipEmpty(true)
                                                               .build(),
                                              
                                              // auto-authorization.adoc 에 작성할 문자열
                                              AutoDocumentation.authorization(PUBLIC_AUTHORIZATION)
                                      ))
                              .build();
    }

    private static RestDocumentationResultHandler restDocumentation(Snippet... snippets) {
        return MockMvcRestDocumentation

                // RestDocs 스니펫 이름 및 생성 위치를 설정합니다.
                .document("{class-name}/{method-name}",
                        preprocessRequest(

                                //  Request 와 Response 를 정리하여 출력합니다.
                                prettyPrint(),

                                // 지정 헤더를 제외하고 스니펫을 생성합니다.
                                removeHeaders("Host", "Content-Length")
                        ),
                        preprocessResponse(
                                replaceBinaryContent(),
                                limitJsonArrayLength(OBJECT_MAPPER),
                                prettyPrint(),
                                removeHeaders("Content-Length")
                        ),
                        snippets
                );

    }
}
```

다음은 컨트롤러의 테스트 코드입니다.

```java
@ExtendWith({RestDocumentationExtension.class, SpringExtension.class})
@WebMvcTest(controllers = {
        MemberController.class
})
public class RestApiDocumentTest {

    @Autowired
    private MemberController memberController;

    @BeforeEach
    void setUp(RestDocumentationContextProvider provider) {
        this.restDocsMockMvc = RestDocsMockMvcUtils.successRestDocsMockMvc(provider, memberController);
    }
    
    @DisplayName("리뷰어 목록 조회 테스트 - 성공")
    @Test
    void findAllTeacherTest() throws Exception {
        this.restDocsMockMvc.perform(get("/teachers").param("skills", "Java", "Spring")
                                                     .param("career", "3")
                                                     .param("limit", "10")
                                                     .param("page", "5"))
                            .andExpect(status().isOk())
                            .andDo(print());
    }

    @DisplayName("리뷰어 등록 테스트 - 성공")
    @Test
    void registerTeacherTest() throws Exception {
        TeacherRegistrationRequest request
                = new TeacherRegistrationRequest(Arrays.asList("java", "spring"), 3, "백엔드 개발자입니다.", "환영합니다.");

        this.restDocsMockMvc.perform(post("/teachers").with(userToken())
                                                      .content(OBJECT_MAPPER.writeValueAsString(request))
                                                      .contentType(MediaType.APPLICATION_JSON))
                            .andExpect(status().isCreated())
                            .andDo(print());
    }

    // Request에 `Authroization Header`를 추가합니다.
    // auto-authorization의 메시지를 수정합니다.
    RequestPostProcessor userToken() {
        return request -> {
            request.addHeader("Authorization", "Bearer aaa.bbb.ccc");
            return documentAuthorization(request, "User jwt token required.");
        };
    }
}
```

Spring REST Docs와 다르게 문서를 작성하기 위한 메소드들이 전혀 작성되어 있지 않습니다. 굉장히 깔끔한 테스트 코드가 작성되었습니다.

테스트 코드를 작성하고, Gradle에서 build를 실행해보면 `build/generated-snippets` 라는 패키지가 생성된 것을 확인할 수 있습니다.

![스크린샷 2021-07-11 오후 9 54 45](https://user-images.githubusercontent.com/56301069/125195866-a2d67b80-e292-11eb-9f3c-ac25bbc8250a.png)

### 리뷰어 목록 조회의 스니펫
리뷰어 목록 조회의 스니펫들은 다음과 같은 형태로 생성됩니다.

- auto-authorization.adoc

```
Resource is public.
```

- auto-description.adoc

```
리뷰어 목록을 조회한다.
```

- auto-method-path.adoc

```
`+GET /teachers+`
```

- auto-path-parameters.adoc

```
No parameters.
```

- auto-request-fields.adoc

```
No request body.
```

- auto-request-parameters.adoc

```
|===
|Parameter|Type|Optional|Description

|skills
|Object
|true
|선생님 기술 스택.

|career
|Integer
|true
|선생님 경력 연차.

|limit
|Integer
|false
|한 페이지에 보여줄 선생님 수.

|page
|Integer
|false
|현재 페이지 번호.

|===
```

- auto-response-fields.adoc

```
No response body.
```

- auto-section.adoc

```
[[resources-auto-member-controller-test-find-all-teacher-test]]
=== Find All Teacher

include::auto-method-path.adoc[]

include::auto-description.adoc[]

==== Authorization

include::auto-authorization.adoc[]

==== Query parameters

include::auto-request-parameters.adoc[]

==== Response fields

include::auto-response-fields.adoc[]

==== Example request

include::http-request.adoc[]

==== Example response

include::http-response.adoc[]
No parameters.
```

- http-request.adoc

```
[source,http,options="nowrap"]
----
GET /teachers?skills=Java&skills=Spring&career=3&limit=10&page=5 HTTP/1.1

----
```

- http-response.adoc

```
[source,http,options="nowrap"]
----
HTTP/1.1 200 OK

----
```

### 리뷰어 등록의 스니펫
리뷰어 등록의 스니펫들은 다음과 같은 형태로 생성됩니다.

- auto-authorization.adoc

```
User jwt token required.
```

- auto-description.adoc

```
리뷰어를 등록한다.
```

- auto-method-path.adoc

```
`+POST /teachers+`
```

- auto-path-parameters.adoc

```
No parameters.
```

- auto-request-fields.adoc

```
|===
|Path|Type|Optional|Description

|skills
|Array[String]
|false
|선생님 기술 경력.

|career
|Integer
|false
|선생님 연차.

Must be positive or zero.

|title
|String
|false
|선생님 자기 소개 제목.

|content
|String
|false
|선생님 자기 소개 내용.

|===
```

- auto-request-parameters.adoc

```
No parameters.
```

- auto-response-fields.adoc

```
No response body.
```

- http-request.adoc

```
[source,http,options="nowrap"]
----
POST /teachers HTTP/1.1
Content-Type: application/json;charset=UTF-8
Authorization: Bearer aaa.bbb.ccc

{
  "skills" : [ "java", "spring" ],
  "career" : 3,
  "title" : "백엔드 개발자입니다.",
  "content" : "환영합니다."
}
----
```

- http-response.adoc

```
[source,http,options="nowrap"]
----
HTTP/1.1 201 Created

----
```

### TroubleShooting
- 인텔리제이에서 build 패키지가 보이지 않을 경우, `Show Excluded Files`가 체크 해제 되어 있는지 확인해보세요. 

![스크린샷 2021-07-11 오후 9 56 15](https://user-images.githubusercontent.com/56301069/125195924-d44f4700-e292-11eb-8440-45278016b17a.png)

---

- `generated-javadoc-json` 패키지가 생성되지 않을 경우, Gradle에서 `jsonDoclet`을 실행해보세요.

![스크린샷 2021-07-11 오후 10 02 47](https://user-images.githubusercontent.com/56301069/125196171-c64df600-e293-11eb-963b-f2547091e163.png)
  
`jsonDoclet` 실행해도 json 패키지가 생성되지 않는다면, `build.gradle`에서 `jsonDoclet`이 제대로 설정되어 있는지 확인해주세요.
```groovy
task jsonDoclet(type: Javadoc, dependsOn: compileJava) {
    source = sourceSets.main.allJava
    classpath = sourceSets.main.compileClasspath
    destinationDir = javadocJsonDir
    options.docletpath = configurations.jsondoclet.files.asType(List)
    options.doclet = 'capital.scalable.restdocs.jsondoclet.ExtractDocumentationAsJsonDoclet'
    options.memberLevel = JavadocMemberLevel.PACKAGE
}
```

json 패키지가 생성되었다면, build.gradle에서 test가 jsonDoclet을 의존하고 있는지 다시 한 번 확인해주세요.
```groovy
test {
    systemProperty 'org.springframework.restdocs.outputDir', snippetsDir
    systemProperty 'org.springframework.restdocs.javadocJsonDir', javadocJsonDir

    dependsOn jsonDoclet
    useJUnitPlatform()

    outputs.dir snippetsDir
}
```
## 5. adoc 파일 작성
IntelliJ를 사용하고 있다면, 먼저 asciidoc 플러그인을 설치합니다.
![스크린샷 2021-07-11 오후 10 12 35](https://user-images.githubusercontent.com/56301069/125196489-285b2b00-e295-11eb-989d-1ca09734232a.png)

그 후, `src/docs/asciidoc` 패키지 안에 `.adoc` 으로 끝나는 문서를 작성해주세요. 저는 `api.adoc`으로 생성했습니다.
![스크린샷 2021-07-11 오후 10 07 19](https://user-images.githubusercontent.com/56301069/125196307-6146d000-e294-11eb-8fd4-1d82af4c253f.png)

해당 파일에서 `asciidoc` 문법을 이용해서 API 문서를 꾸며줍니다.


> asciidoc 문법 
> 
> [https://narusas.github.io/2018/03/21/Asciidoc-basic.html](https://narusas.github.io/2018/03/21/Asciidoc-basic.html)

다음은 간단히 작성한 API 문서입니다.

```
ifndef::snippets[]
:snippets: ./build/generated-snippets
endif::[]
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 3
:sectlinks:
:docinfo: shared-head

= REST API Document

[[introduction]]
== Introduction

코드 봐줘 👀

누구나 편하고 쉽게 리뷰어를 만날 수 있는 환경을 제공하는 서비스

[[common]]
== Common

=== Domain

|===
| 환경 | Domain

| 개발서버
| `https://aaaaaaaaaaa`

| 운영서버
| `https://aaaaaaaaaaa`
|===

=== Exception

|===
| 상태 코드 | 설명

| 400
| `잘못된 데이터`

| 401
| `권한 없음`
|===

== Member API

include::{snippets}/auto-member-controller-test/oauth-test/auto-section.adoc[]

[[teacher-list]]
=== 선생님 목록 조회

include::{snippets}/auto-member-controller-test/find-all-teacher-test/auto-section.adoc[]
```

Gradle에서 build를 실행하면, `src/main/resources/static/docs` 패키지 안에 `api.html` 파일이 생성됩니다.

SpringApplication을 실행하여 문서가 잘 작성되었는지 확인해봅시다!

![스크린샷 2021-07-11 오후 10 15 42](https://user-images.githubusercontent.com/56301069/125196628-8be55880-e295-11eb-9bb8-7e6ddb79523b.png)


### TroubleShooting
> include::/Users/kimhyeonsik/Desktop/programming/wootech/level3/dtc/backend/build/generated-snippets/auto-member-controller-test/find-all-teacher-test/auto-section.adoc

만약 adoc 파일에서 위와 같은 메시지가 출력된다면, 다음과 같이 해당 메시지 끝 `.adoc` 뒤에 대괄호 `[]` 를 추가하세요.

> include::/Users/kimhyeonsik/Desktop/programming/wootech/level3/dtc/backend/build/generated-snippets/auto-member-controller-test/find-all-teacher-test/auto-section.adoc[]

---

src/main/resources/static/docs/api.html 파일이 생성되지 않는다면, `build/docs/asciidoc/api.html` 파일이 있는지 확인해보세요. 
- 만약 해당 파일이 존재하지 않는다면, build.gradle에서 `asciidoctor`가 제대로 작성되어 있는지 확인해보세요.
- 만약 해당 파일이 존재한다면, build.gradle에서 `copyDocument`가 제대로 작성되어 있는지 확인해보세요.

---

nohup java -jar 로 어플리케이션을 실행 후, api.html 파일에 접근 시 404 Not Found가 나타난다면, build.gradle를 확인해보세요.

api.html 파일을 생성(asciidoctor 후 copyDocument)한 후, jar 파일이 생성되어야 합니다.
하지만 기본 Gradle의 build는 jar 파일을 먼저 생성한 후, api.html 파일을 생성합니다.

> 의존 설정을 안한 build 실행 시 task flow
> 
![무제 2](https://user-images.githubusercontent.com/56301069/125197438-9bb26c00-e298-11eb-844b-7c7969b4acb8.jpg)

assemble은 jar를 의존하고 있으며, jar는 jar 파일을 생성합니다. 그러므로 copyDocument가 실행되고, assemble이 실행되어야 합니다. 순서가 뒤바뀌면 api.html이 없는 상태로 jar가 생성된 후, api.htm 파일이 생깁니다.

build.gradle에서 assemble이 copyDocument를 의존하고 있는지 확인해보세요.

```groovy
assemble {
    dependsOn copyDocument
}
```
