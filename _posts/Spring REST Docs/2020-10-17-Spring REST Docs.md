---
title : "[Spring REST Docs Reference] Spring REST Docs"

categories :
  - Spring REST Docs

tags :
  - Spring REST Docs
---

[원문](https://docs.spring.io/spring-restdocs/docs/2.0.5.RELEASE/reference/html5/)

# 1. Introduction

REST Docs의 목표는 RESTful 서비스를 정확하고 읽기 편하게 문서화 하는 것을 돕는 것이다.

높은 수준의 문서화를 하는 것은 어렵다. 이 작업에 잘 맞는 툴을 사용하는데 있어서 어려움을 편리하게 하는 방법으로 스프링 REST Docs는 `Ascii Doctor` 를 사용한다. 아스키닥터는 평문(Plain Text)을 처리하여 필요에 맞는 스타일과 레이어를 적용한 HTML을 만들어준다. 또한 원한다면, Spring REST Docs를 설정하여 `Markdown` 을 사용할 수 있다.

스프링 REST Docs는 Spring MVC의 Test Framework로 쓰여진 테스트를 통해 만들어진 코드 조각(snippet)들을 사용한다. 이 테스트 기반의 접근법(Test-Driven Approach)은 서비스에 대한 문서화의 정확도를 보장해준다. 스니펫이 올바르지 않다면 결과물 생성에 실패할 것이다.

# 2. Getting Started

## 2-1. Gradle Build Configuration

```java
// 아스키닥터 플러그인을 적용
plugins { 
	id "org.asciidoctor.convert" version "1.5.9.2"
}

dependencies {
	// asciidoctor 설정에 spring-restdocs-asciidoctor 의존성을 추가한다.
	// 이것은 `build/generated-snippets`을 가리키는 
	// `.adoc` 파일들을 사용하기 위한 snippets 속성들을 자동으로 설정할 것이다
	asciidoctor 'org.springframework.restdocs:spring-restdocs-asciidoctor:{project-version}' 

	// spring-restdocs-mockmvc 의존성을 추가한다
	testCompile 'org.springframework.restdocs:spring-restdocs-mockmvc:{project-version}' 
}

// 생성되는 스니펫을이 저장되는 위치를 설정한다
ext { 
	snippetsDir = file('build/generated-snippets')
}

// 결과물인 스니펫 디렉토리를 추가하는 `test` task를 설정
test { 
	outputs.dir snippetsDir
}

// `asciidoctor` task를 설정
asciidoctor { 

	// 입력 값으로 snippets 디렉토리 설정하기
	inputs.dir snippetsDir  

	// task를 test task에 의존하게 만들어 문서가 생성되기 전에 테스트가 실행된다
	dependsOn test  
}
```

## 2-2. Packaging The Documentation

프로젝트의 jar 파일 안에 생성된 문서를  패키지 하고 싶을 수도 있습니다.  예를 들어, 스프링 부트에 의해 제공되는 **정적인 컨텐츠 제공**이 있습니다. 프로젝트의 빌드를  설정함으로써, 

1. 문서는 jar가 빌드되기 전에 생성됩니다
2. 생성된 문서는 jar 파일 안에 포함됩니다

아래는 Gradle에서의 설정입니다.

```java
bootJar {

	// jar가 빌드 되기 전에 문서가 생성되도록 합니다
	dependsOn asciidoctor 

	// jar의 `static/docs` 디렉토리 안에 생성된 문서를 복사합니다
	from ("${asciidoctor.outputDir}/html5") {  
		into 'static/docs'
	}
}
```

## 2-3. Generating Documentation Snippets

Spring REST Docs는 당신이 문서화하고 있는 서비스에 요청을 보내기 위해 Spring MVC의 [test framework](https://docs.spring.io/spring-framework/docs/5.0.x/spring-framework-reference/testing.html#spring-mvc-test-framework)를 사용합니다. Spring REST Docs는 요청과 응답 결과 문서 스니펫들을 제공합니다.

### 2-3-1. Setting up Your Tests

테스트를 셋업하는 방법은 사용하고 있는 teste framework에 의존합니다. Spring REST Docs는 JUnit4와 JUnit5를 지원합니다. TestNG같은 다른 framework들도 지원하지만 조금 더 많은 설정이 요구됩니다.

### 2-3-2. Setting up Your JUnit5 Tests

JUnit5를 사용할 때, 문서 스니펫을 생성하기 위한 첫 번째 단계는 `RestDocumentaitionExtension`을 테스트 클래스에 적용하는 것입니다. 다음의 예는 그 방법을 보여주고 있습니다.

```java
@ExtendWith(RestDocumentationExtension.class)
public class JUnit5ExampleTests {
```

일반적인 스프링 어플리케이션을 테스트할 땐 `SpringExtension`도 적용해야 합니다.

```java
@ExtendWith({RestDocumentationExtension.class, SpringExtension.class})
public class JUnit5ExampleTests {
```

`RestDocumentaitionExtension` 은 프로젝트의 빌드 설정을 기반으로 결과 디렉토리가 자동으로 설정됩니다.

만약 JUnit 5.1을 사용하고 있는 중이라면, 테스트 클래스에 필드로 `@RegisterExtension`을 선언하여 기본 설정을 Override할 수 있습니다. `@RegisterExtension` 은 생성할 때 결과 디렉토리를 변수로 받습니다.

```java
public class JUnit5ExampleTests {

	@RegisterExtension
	final RestDocumentationExtension restDocumentation = 
			new RestDocumentationExtension ("custom");

}
```

다음, 당신은 `MockMvc` 를 설정하기 위해 반드시 `@BeforEach` 메소드를 선언해야 합니다. 

```java
private MockMvc mockMvc;

@BeforeEach
public void setUp(WebApplicationContext webApplicationContext,
		RestDocumentationContextProvider restDocumentation) {
	this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
			.apply(documentationConfiguration(restDocumentation)) // 1
			.build();
}
```

1. `MockMvc` 인스턴스는 `MockMvcRestDocumentationConfigurer` 을 사용하여 설정합니다. 정적 메소드인 `MockMvcRestDocumentation#documentationConfiguration()` 를 사용하여 이 클래스의 인스턴스를 얻을 수 있습니다.

**Configurer** 는 민감한 기본 설정들을 적용하고, 설정을 커스터마이징하기 위한 API를 제공합니다.

### 2-3-3. Invoking the RESTful Service

testing framework를 설정하였으므로, 이제 RESTful service를 호출하고, 요청과 응답을 문서화시킬 수 있습니다.

```java
this.mockMvc.perform(get("/").accept(MediaType.APPLICATION_JSON)) // 1
	.andExpect(status().isOk()) // 2
	.andDo(document("index")); // 3
```

1. 서비스의 root(`/`)를 호출하고, 응답은 `application/json` 형식이어야 함을 나타냅니다
2. 서비스가 만들어 내는 원하는 응답을 결정합니다
3. 결과 디렉토리로 설정된 디렉토리 밑에 `index`라는 이름을 가진 디렉토리 안에 스니펫을  작성함으로써 서비스로의 호출을 문서화합니다. 이 스니펫들은 `RestDocumentationResultHandler` 에 의해 작성됩니다. 또, 정적 메소드인 `MockMvcRestDocumentation#document()` 를 사용하여 이 클래스의 인스턴스를 얻을 수 있습니다

기본적으로 6개의 스니펫들이 생성됩니다.

- `<output-directory>/index/curl-request.adoc`
- `<output-directory>/index/http-request.adoc`
- `<output-directory>/index/http-response.adoc`
- `<output-directory>/index/httpie-request.adoc`
- `<output-directory>/index/request-body.adoc`
- `<output-directory>/index/response-body.adoc`

## 2-4. Using the Snippets

생성된 스니펫들을 사용하기 전에 반드시 `.adoc` 소스 파일을 생성해야 합니다. 끝에 `.adoc` suffix(접미사)만 붙는다면 파일 이름을 원하는만큼 길게 명명할 수 있습니다. HTML 결과 파일은 같은 이름을 갖지만 끝에 `.html` suffix를 갖습니다. 소스 파일의 기본 위치와 HTML 결과 파일은 `Maven` 이냐 `Gradle` 이냐에 따라 다릅니다.

**Maven**

- Source files : src/main/asciidoc/*.adoc
- Generated files : target/generated-docs/*.html

**Gradle**

- Source files : src/docs/asciidoc/*.adoc
- Generated files : build/asciidoc/html5/*.html

[include macro](https://asciidoctor.org/docs/asciidoc-syntax-quick-reference/#include-files)를 사용하여 생성된 스니펫들을 메뉴얼대로 생성한 Asciidoc file 안에 포함시킬 수 있습니다. 스니펫의 결과 디렉토리를 결정하기 위해 build configuration에서 설정한 `spring-restdocs-asccidoctor` 에 의해 자동으로 설정되는`snippets` 속성들을 사용할 수 있습니다.

```
include::{snippets}/index/curl-request.adoc[]
```

# 3. Documenting your API

## 3-1. Hypermedia

Spring REST Docs는 [Hypermedia-based](https://en.wikipedia.org/wiki/HATEOAS) API의 링크들을 문서화를 지원합니다. 

```java
this.mockMvc.perform(get("/").accept(MediaType.APPLICATION_JSON))
	.andExpect(status().isOk())
	.andDo(document("index", links( // 1
			linkWithRel("alpha").description("Link to the alpha resource"), // 2
			linkWithRel("bravo").description("Link to the bravo resource")))); // 3
```

1. 응답 링크들을 설명하는 스니펫을 만들기 위해 Spring REST Docs를 설정합니다. 스태틱 메소드인 `HypermediaDocumentation#links()`를 사용합니다
2. `rel`은 `alpha`인 링크를 예상합니다. 스태틱 메소드인 `HypermediaDocumentation#linkWithRel()` 를 사용합니다
3. `rel`은 `bravo`인 링크를 예상합니다

결과는 리소스의 링크들을 설명하는 테이블을 가지고 있는 `links.adoc` 이라는 이름을 가진 스니펫입니다.

링크들을 문서화할 때, 만약 응답에서 문서화되지 않은 링크가 발견되면 테스트는 실패합니다. 유사하게 만약 응답에 문서화된 링크가 존재하지 않고, 링크가 optional로 마크(설정)되어 있지 않다면, 테스트는 실패합니다. 

만약 링크를 문서화하고 싶지 않다면, 무시(Ignored)로 마크합니다. 이렇게 하면 위에서 설명한 오류를 방지하면서 생성된 스니펫에 표시되지 않습니다.

문서화되지 않은 링크로 인해 테스트 실패가 발생하지 않는 `relaxed mode`에서`HypermediaDocumentation#relaxedLinks()`를 사용하여 문서화할 수 있습니다. 

## 3-1-1. Hypermedia Link Formats

두 링크 포맷들은 기본 값으로 사용됩니다.

- Atom : 링크들은 `links`라는 이름의 배열 안에 있어야 합니다. 응답의 컨텐츠 유형이 `application/json`과 호환 가능할 때 기본적으로 사용됩니다
- HAL : 링크들은 `_links` 라는 이름의 맵 안에 있어야 합니다. 응답의 컨텐츠 유형이 `application/hal+json`과 호환 가능할 때 기본적으로 사용됩니다

Atom 또는 HAL 포맷 링크들을 사용하지만 컨텐츠 유형이 다르다면, 기본적으로 제공되는 `LinkExtractor` 구현체 중 하나를 링크에게 제공합니다.

```java
.andDo(document("index", links(halLinks(), // 1
		linkWithRel("alpha").description("Link to the alpha resource"),
		linkWithRel("bravo").description("Link to the bravo resource"))));
```

링크가 HAL 포맷이라는 것을 나타냅니다. 스태틱 메소드인 `HypermediaDocumentation#halLinks()`를 사용합니다.

만약 API가 Atom이나 HAL이 아닌 다른 포맷이라면, 응답에서 링크를 추출하기 위해 `LinkExtractor` 인터페이스의 구현체를 직접 만들어 응답에서 링크를 추출할 수 있습니다.

## 3-1-2. Ignoring Common Links

HAL을 사용할 때 `self` 및 `curies` 같은 모든 응답에 공통적인 링크를 문서화하는 대신 개요 섹션에 한 번 문서화한 다음 나머지 API 문서에서 무시할 수 있습니다. 이를 위해, 특정 링크를 무시하도록 미리 구성된 스니펫에 링크에 대한 설명을 추가하기 위한 스니펫의 재사용 지원을 기반으로 만들 수 있습니다.

```java
public static LinksSnippet links(LinkDescriptor... descriptors) {
	return HypermediaDocumentation.links(linkWithRel("self").ignored().optional(),
			linkWithRel("curies").ignored()).and(descriptors);
}
```

## 3-2. Request and Response Payloads

**hypermedia-specific** 앞에서 설명한 지원 이외에, 일반적인 요청과 응답 내용 문서 지원도 제공됩니다.

기본적으로, Spring REST Docs는 자동으로 Request와 Response Body에 스니펫을 만듭니다. 이 스니펫들은 각 각 `request-body.adoc` 과 `response-body.adoc`으로 불립니다. 

### 3-2-1. Request and Response Fields

요쳥과 응답 페이로드의 더 상세한 문서를 제공하기 위해 페이로드의 필드 문서화가 제공됩니다.

```json
{
	"contact": {
		"name": "Jane Doe",
		"email": "jane.doe@example.com"
	}
}
```

위의 예시의 필드들을 다음과 같이 문서화할 수 있습니다.

```java
this.mockMvc.perform(get("/user/5").accept(MediaType.APPLICATION_JSON))
		.andExpect(status().isOk())
		.andDo(document("index",
				responseFields( // 1
						fieldWithPath("contact.email")
								.description("The user's email address"), // 2
				fieldWithPath("contact.name").description("The user's name")))); // 3
```

1. 응답 페이로드의 필드를 설명하는 스니펫을 생산하도록 Spring REST Docs를 설정합니다. `requestFields`를 사용하여 요청을 문서화할 수 있습니다. 둘 다 `PayloadDocumentation` 클래스에 있는 정적 메소드입니다
2. `contact.email` 경로를 가진 필드가 필요합니다. 스태틱 메소드인 `PayloadDocumentation#fieldWithPath()` 를 사용합니다
3. `contact.name` 경로를 가진 필드가 필요합니다

결과는 필드를 설명하는 테이블을 가진 스니펫입니다. 요청에서 이 스니펫은 `request-fields.adoc` 의 이름을 갖습니다. 응답에서, `response-fields.adoc` 의 이름을 갖습니다.

필드를 문서화할 때, 만약 문서화되지 않은 필드가 페이로드에 있다면 테스트는 실패합니다. 유사하게 만약 문서화된 필드가 페이로드에 없고 필드가 `optional` 로 마크(설정)되어 있지 않다면 테스트는 실패합니다.

만약 모든 필드의 상세한 문서를 제공하는 것을 원하지 않는다면, 한 페이로드의 전체 서브섹션만 문서화할 수 있습니다.

```java
this.mockMvc.perform(get("/user/5").accept(MediaType.APPLICATION_JSON))
		.andExpect(status().isOk())
		.andDo(document("index",
				responseFields( // 1
						subsectionWithPath("contact")
								.description("The user's contact details")))); //1
```

1. `contact` 경로로 서브 섹션을 문서화합니다. `contact.email` 과 `contact.name` 은 이제 문서화된 것으로 간주합니다. 정적 메소드인  `PayloadDocumentation#subsectionWithPath()` 를 사용합니다.

`subsectionWithPath` 은 한 페이로드의 특정 섹션에 대한 높은 수준의 개요을 제공할 때 유용합니다. 이제 독립적이고 더 상세한 서브 섹션 문서를 작성할 수 있습니다.

`ignored`로 마크(설정)하면 필드나 서브 섹션을 문서화하지 않습니다. 이렇게 하면 앞에서 설명한 오류를 피하면서 생성된 스니펫에 표시되지 않습니다.

문서화되지 않은 필드로 인해 테스트 실패가 발생하지 않는 `relaxed mode`에서 필드를 문서화할 수도 있습니다. 이를 위해 `PayloadDocumentation#relaxedRequestFields(), PayloadDocumentation#relaxedResponseFields()` 를 사용합니다. 이건 페이로드의 한 집합에만 집중하길 원하는 특정 시나리오를 문서화할 때 유용합니다.

Spring REST Docs 기본적으로 문서화하고 있는 페이로드는 JSON일 것이라고 가정합니다.

**JSON Field Types**

필드가 문서화되었을 때, Spring REST Docs는 페이로드를 검사하여 타입을 결정하려고 합니다. 

지원되는 JSON 타입들은 다음과 같습니다.

- array
- boolean
- object
- number
- null
- string
- varies : 이 필드는 다양한 유형의 페이로드에서 여러 번 발생합니다

`FieldDescriptor()` 메소드의 `type(Object)`로 확실하게 타입을 정할 수 있습니다. `Object#toString()` 의 결과는 문서에 사용됩니다. 보통 `JsonFieldType`에 의해 계산된 값들 중 하나가 사용됩니다.

```java
.andDo(document("index",
		responseFields(
				fieldWithPath("contact.email").type(JsonFieldType.STRING) // 1
						.description("The user's email address"))));
```

1. 필드의 타입을 `String` 으로 정합니다

**Reusing Field Descriptors**

스니펫 재사용에 대한 일반적인 지원 외에도 요청 및 응답 스니펫을 통해 **path prefix**로 추가 Descriptor를 구성할 수 있습니다. 이를 통해 요청 또는 응답 페이로드의 반복 부분에서 Descriptor를 한 번만 생성한 다음 재사용할 수 있습니다.

다음은 책을 반환하는 엔드포인트입니다.

```json
{
	"title": "Pride and Prejudice",
	"author": "Jane Austen"
}
```

다음은 책 배열을 반환하는 엔드포인트입니다.

```json
[{
	"title": "Pride and Prejudice",
	"author": "Jane Austen"
},
{
	"title": "To Kill a Mockingbird",
	"author": "Harper Lee"
}]
```

`title` 과 `author` 의 경로는 각 각 `[].title` 과 `[].author` 입니다. 책 한권과 책 배열 사이의 차이점은 오직 필드의 경로가 지금은 접두사로 `[].` 를 가지고 있다는 것 뿐입니다.

다음과 같이 책을 문서화하는 설명자를 만들 수 있습니다.

```java
FieldDescriptor[] book = new FieldDescriptor[] {
		fieldWithPath("title").description("Title of the book"),
		fieldWithPath("author").description("Author of the book") };
```

다음과 같이, 책 한 권을 문서화하기 위해 위의 설명자들을 사용할 수 있습니다.

```java
this.mockMvc.perform(get("/books/1").accept(MediaType.APPLICATION_JSON))
		.andExpect(status().isOk()).andDo(document("book", responseFields(book)));
```

또, 책 배열을 문서화할 수도 있습니다.

```java
this.mockMvc.perform(get("/books").accept(MediaType.APPLICATION_JSON))
		.andExpect(status().isOk())
		.andDo(document("book",
				responseFields(
						fieldWithPath("[]").description("An array of books")) // 1
								.andWithPrefix("[].", book))); // 2
```

1. 배열을 문서화합니다
2. `[].title` 과 `[].author` 를 존재하는 `[].`로 시작하는 접두사를 가진 Descriptor를 사용하여 문서화합니다

### 3-2-2. Documenting a Subsection of a Request or Response Payload

REST Docs는 페이로드가 크거나 구조적으로 복잡할 때, 페이로드의 개인 섹션들을 문서화하는데 도움이 됩니다. REST Docs는 당신이 페이로드의 서브섹션을 추출하고, 그것을 문서화시킴으로써 그것을 가능하게 합니다.

**Documenting a Subsecsion of a Request or Response Body**

다음과 같은 JSON Response Body가 있습니다.

```json
{
	"weather": {
		"wind": {
			"speed": 15.3,
			"direction": 287.0
		},
		"temperature": {
			"high": 21.2,
			"low": 14.8
		}
	}
}
```

다음과 같이 `temperature` 오브젝트를 문서화하는 스니펫을 만들 수 있습니다.

```java
this.mockMvc.perform(get("/locations/1").accept(MediaType.APPLICATION_JSON))
		.andExpect(status().isOk()).andDo(document("location",
				responseBody(beneathPath("weather.temperature")))); // 1
```

1. `PayloadDocumentation` 클래스 안에 있는 정적 메소드 `responseBody()`,  `beneathPath()` 를 사용하여, Response Body의 서브 섹션을 가지고 있는 스니펫을 생산합니다. Request Body에서 스니펫을 생상하기 위해선 `responseBody` 자리에 `requestBody`를 적으면 됩니다

위의 예시의 결과는 다음과 같습니다.

```json
{
	"temperature": {
		"high": 21.2,
		"low": 14.8
	}
}
```

스니펫의 이름을 분명하게 만들기 위해, 서브 섹션에 식별자가 포함되야 합니다. 기본적으로, 이 식별자는 `beneath-${path}`입니다. 예를 들어, 앞의 코드 결과는 `response-body-beneath-weather.temperature.adoc` 의 이름을 갖습니다. 다음과 같이 `withSubsectionId(String)` 메소드를 사용하여 식별자를 커스터마이징 할 수 있습니다.

```java
responseBody(beneathPath("weather.temperature").withSubsectionId("temp"));
```

스니펫 결과의 이름은 `request-body-temp.adoc` 입니다.

**Documenting the Fields of a Subsection of a Request or Response**
