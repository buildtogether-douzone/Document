### Spring - Swagger 

Spring으로 Rest API를 개발하고 그 API에 대한 문서를 정리하여 해당 API를 사용하는 클라이언트 및 서버 개발자들에게 문서를 정리해서 공유해야하는데 이때 Swagger를 이용하게되면 이런 작업을 보다 편리하게 할 수 있고 API 문서 자동화 뿐만 아니라 UI에서 직접 API 테스트를 하는 것

### 개발 환경

+ Spring Boot 
+ Maven 
+ Java 8



### Spring 의 Pom.xml에 의존성 추가 

```bash
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version> 
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```



### Swagger 설정

+ SwaggerConfig.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Profile({"local","dev"})
@EnableSwagger2
@Configuration
public class SwaggerConfig {
	
    private static final String API_NAME = "UCARE API";
    private static final String API_VERSION = "0.0.1";
    private static final String API_DESCRIPTION = "UCARE API 명세서";
	
	@Bean
	public Docket api() {
		return new Docket(DocumentationType.SWAGGER_2)
           .apiInfo(apiInfo())
		   .select()
		   .apis(RequestHandlerSelectors.basePackage("com.douzone.ucare.controller.api"))
		   .paths(PathSelectors.any())
		   .build();
	}
	
    public ApiInfo apiInfo() {
	return new ApiInfoBuilder()
		.title(API_NAME)
		.version(API_VERSION)
		.description(API_DESCRIPTION)
		.build();
    }

}
```

* 설정에 대한 설명
  * ApiInfo apiInfo():
    *  API의 이름은 무엇이며 현재 버전은 어떻게 되는지에 대한 API 정보를 적는다
  * RequestHandlerSelectors.basePackage(String packageName): 
    * Swagger를 적용할 클래스의 package 명
  * PathSelectors.any()
    * 해당 package 하위에 있는 모든 url에 적용시킨다. /test/**로시작되는 부분만 적용해주고싶다면 PathSelectors.ant(String pattern) 메서드에 url parttern을 추가해주면된다.



+ WebConfig.java

```java
public class WebConfig implements WebMvcConfigurer {
		// Resource Mapping(URL Magic Mapping)
	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler(env.getProperty("fileupload.resourceMapping"))
				.addResourceLocations("file:" + env.getProperty("fileupload.uploadLocation"));
        registry.addResourceHandler("/swagger-ui.html")
        		.addResourceLocations("classpath:/META-INF/resources/");
		registry.addResourceHandler("/webjars/**")
		        .addResourceLocations("classpath:/META-INF/resources/webjars/");
	}
}
```

**주소창에  localhost:8080/swagger-ui.html를 입력하여 접속을 할 때 404 에러를 방지 하기위해 위와 같은 설정을 하였다.**

하지만 지금 하고 있는 프로젝트에서는 위와 같은 설정을 하였는데도 404 에러가 발생 하였는데 해당 프로젝트 설정에서는 다음과 같이 context-path가 설정 되어있어서 404 가 발생 하였다. 

```yaml
# server
server:
   port: 8080
   servlet:
      context-path: /ucare_backend
```



### Swagger 화면

http://localhost:8080/ucare_backend/swagger-ui.html 로 접속을 하니 화면이 잘 나왔다.

 ![캡처](https://user-images.githubusercontent.com/67888402/134760241-8fc5fb86-c60f-4af9-8f74-2cce15658877.PNG)





### Swagger 기능

+ 해당 API가 어떤 API 인지 알아야 함으로 @ApiOperation 어노테이션으로 해당 API에 대한 간단한 설명을 적을 수 있다.

```java
@GetMapping("/retrieveAll")
@ApiOperation(value="게시판 데이터 불러오기", 
              notes="홈페이지 렌더링시 게시판 정보를 가져오기 위한 API")
public ResponseEntity<?> retrieveAll() {
	return new ResponseEntity<>(boardService.retrieveAll(), HttpStatus.OK);
}
```

![캡처](https://user-images.githubusercontent.com/67888402/134760348-8416ee07-53a9-4e91-943d-85a1d1e4ef0e.PNG)



+ vo에서 @ApiModelProperty 어노테이션으로 해당 필드가 어떤의미인지 적을수 있다.

```java
@ApiModelProperty(example="사용자 아이디")
private String id;
@ApiModelProperty(example="사용자 비밀번호")
private String password;
@ApiModelProperty(example="사용자 이름")
private String name;
```

![캡처](https://user-images.githubusercontent.com/67888402/134789039-b222fb41-555f-4c0a-8275-fcf4c8fda829.PNG)



+ @ApiImplicitParam 어노테이션을 적어주면 해당 파라미터명은 무엇이고 파라미터값은 어떤의미인지 알 수 있다.

```java
@PostMapping("/create")
@ApiOperation(value="게시판 글 작성", notes="게시판 글 작성시 실행되는 API")
@ApiImplicitParams({
	@ApiImplicitParam(name="data", value="게시판 글 작성에 대한 정보"),
	@ApiImplicitParam(name="URL", value="글 작성시 올렸던 파일")
})
public ResponseEntity<?> create(
    @RequestPart("data") BoardVo data,
    @RequestPart(value="URL", required = false) MultipartFile file) {
	if(file != null) data.setURL(fileUploadService.restore(file));
	return new ResponseEntity<>(boardService.create(data), HttpStatus.OK);
}
```

![캡처](https://user-images.githubusercontent.com/67888402/134789051-569ac2eb-e375-4296-a165-b45631ad55e9.PNG)

