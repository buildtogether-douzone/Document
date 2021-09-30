## Eclipse 로 작성한 Spring Boot VSCode로 실행 시키기

### maven 설치 (작성자는 Window로 실행 하였음)

1. VSCode 에는 maven이 설치가 안 되어 있으므로

   [maven 홈페이지](https://maven.apache.org/download.cgi)에서 빨간색으로 표시 된걸 다운받아 적절한 위치에 압축 풀기

   [![캡처](https://user-images.githubusercontent.com/67888402/129435371-42c3d119-0878-4c8a-88ef-40350e1704d2.PNG)](https://user-images.githubusercontent.com/67888402/129435371-42c3d119-0878-4c8a-88ef-40350e1704d2.PNG)

2. 환경 변수 셋팅하기

   - 자신이 압축 해제한 폴더의 경로를 복사
   - 제어판 -> 시스템 -> 고급 시스템 설정에 들어가 환경 변수를 클릭

   [![캡처](https://user-images.githubusercontent.com/67888402/129435491-1f167799-dc6f-4e01-ad95-e41aa2719e19.PNG)](https://user-images.githubusercontent.com/67888402/129435491-1f167799-dc6f-4e01-ad95-e41aa2719e19.PNG)

   - 새로 만들기를 클릭하고 이름을 지정한후 복사한 경로 붙여 넣기

   [![캡처](https://user-images.githubusercontent.com/67888402/129435558-cdde1af1-d855-4b94-90fd-f6a7f75b1c97.PNG)](https://user-images.githubusercontent.com/67888402/129435558-cdde1af1-d855-4b94-90fd-f6a7f75b1c97.PNG)

   - 시스템 변수에서 Path 를 찾아 편집을 누르고 새로 만들기로 다음과 같이 설정

[![캡처](https://user-images.githubusercontent.com/67888402/129435606-e1741687-d577-47db-bc2c-9d828bd88e53.PNG)](https://user-images.githubusercontent.com/67888402/129435606-e1741687-d577-47db-bc2c-9d828bd88e53.PNG)

1. cmd 창에서 maven 이 실행 되는지 확인

   - mvn -version 을 입력 하여 확인

   [![캡처](https://user-images.githubusercontent.com/67888402/129435659-75764885-6864-4659-840b-68a7355f6254.PNG)](https://user-images.githubusercontent.com/67888402/129435659-75764885-6864-4659-840b-68a7355f6254.PNG)

   이렇게 나오면 성공

2. maven 설치가 완료 되었으면 vscode 를 실행

- 터미널을 열어서 powershell 에서 mvn -version 실행

[![캡처](https://user-images.githubusercontent.com/67888402/129435697-2e8c49fb-bc3d-4b69-93ee-8bb3e4ad2d64.PNG)](https://user-images.githubusercontent.com/67888402/129435697-2e8c49fb-bc3d-4b69-93ee-8bb3e4ad2d64.PNG)

 다음 과 같이 나오면 완료

1. 이제 CLI 로 빌드 해서 MAVEN을 실행 시키기 위해서 Wrapper 를 실행 해야함

- 우선 `Maven wrapper` 란 개발자들이 Maven 을 별도의 환경에서 개발할 때 local machine 에 `따로 설치를 원하지 않거나` `Maven 의 특정 버전을 빌드하길 원할 때` 사용될 수 있음
- 다시말해, `Maven wrapper` 을 사용하면 빌드 시 `버전`이나 `개발 환경`에 더이상 의존하지 않아도 되며 `독립적`이게 됨
- Maven wrapper 만 가지고 있으면 별도로 설치하지 않아도 되며, classpath 별다른 Maven version 을 지정할 필요도 없음

```
1. 현재 실행하려는 Project Directory 로 이동해서 "mvn -N io.takari:maven:wrapper" 를 실행
	- 실행하면 .mvn directory와 .mvnw, mvnw.cmd를 생성해줌
	- mvnw 와 mvnw.cmd는 Maven Wrapper 를 실행하기 위한 Script 파일임
		= mvnw: mvnw 는 Maven 대신에 사용되는 실행 가능한 unix shell script
		= mvnw.cmd: mvnw 의 윈도우 배치 버전 shell script 입니다.
	- .mvn directory 에는 wrapper directory 가 생성되며 그 안에는 
		MavenWrapperDownloader.java, maven-wrapper.jar, maven-wrapper.properties 3개 파일이 존재
		= MavenWrapperDownloader.java: java class file 인 이 파일을 compiling 및 실행하여 Maven 을 다운로드
		= maven-wrapper.jar: wrapper shell scripts 로 부터 maven 을 실행하고, 다운로드 하는데 사용
		= maven-wrapper.properties: Maven 이 존재하지 않는 경우 다운로드하기 위한 URL 을 명시하기 위해서 사용
	이 파일들을 통해서 Maven 이 존재하지 않아도 특정 버전과 classpath 를 가지고 공통된 Maven을 다운로드 및 사용할 수 있음

2. Maven Wrapper 실행하기
   추가된 Maven Wrapper 파일들로 해당 Maven 을 빌드 및 실행할 수 있음
	Maven으로 Spring boot를 백그라운드로 실행
   	 ./mvnw spring-boot:run
   	 하지만 이것을 하면  No plugin found for prefix 'spring-boot' in the current project and in the plugin groups 라는 오류가 발생함 
```

### 오류 해결 방법

```
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>(...)</version>
</parent>
```

pom에 이러한 설정을 해주거나 설정을 해주기 싫으면

```
mvn org.springframework.boot:spring-boot-maven-plugin:run
```

이러한 명령으로 실행 해야됨

-----------------------------------------------------------------------------------------------------------------------------------------------------------



+ 하지만 위의 방법으로 하였을 때 spring 의 yml  파일에 설정 해 놓은 livereload 가 작동이 안 되었다.
+ 그래서 다른 방법으로 실행 하는 방법을 찾아서 하였다.



#### 1. Java 11을 위의 mvn 설치 방법 대로 설치를 한다.

#### 2. VSCode 의  Extention에서 Spring Boot Extention Pack을 설치한다.

+ 이렇게 하면 일단 vscode에서 mvn spring-boot:run 을 치게 되면 spring boot 가 실행 되며 live Reload도 작동이 잘 된다.



#### 3. Concurrently 사용하기 

+ concurrently는 vscode 에서 따로 서버와 클라이언트를 실행 시켜 줄 필요 없이 한번의 동작으로 두개 다 실행 시켜주는 프로그램이다.



##### 개발환경

+ backend: Spring Boot
+ frontend: React 
+ Java 8



##### 설치

```bash
npm i -D concurrently
```



##### 경로

```bash
Ucare
  | --- bin
  | --- node_modules
  | --- ucare_backend
  | --- ucare_frontend
  | --- .gitignore
  | --- package-lock.json
  | --- package.json
  | --- pom.xml
```



##### 설정

+ package.json

```json
"scripts": {
    "dev": "concurrently \"npm run dev:backend\" \"npm run dev:frontend\" --kill-others",
    "dev:backend": "cd ucare_backend && mvn spring-boot:run",
    "dev:frontend": "cross-env NODE_ENV=development webpack serve --config ucare_frontend/config/webpack.config.js --mode development --progress"
}
```

1. forntend 는 webpack.config.js 파일의 경로를 찾아서 실행하게 해놨고 frontend만 실행 하려면 npm run dev:frontend 를 입력하면 실행이 된다.
2. backend 를 실행 하려면 일단 현재 package.json 파일이 Ucare 폴더 안에 있으므로 Ucare 폴더 안에서 cd 로 ucare_backend 폴더 안으로 들어간 다음 mvn spring-boot:run 으로 backend를 실행 시켜준다. backend만 실행하려면 npm run dev:backend 를 입력하면 실행 된다.
3. concurrently 는 concurrently \"npm run dev:backend\" \"npm run dev:frontend\" --kill-others 이렇게 입력하면 npm run dev:backend 와 npm run dev:frontend 를 동시에 실행 시켜 주는데 뒤에 --kill-others 를 적어주면 backend나 frontend 둘중 하나만 종료 되어도 전부 종료 되게 설정을 해놨다. 두개 다 실행 시키려면 npm run dev를 하면 실행이 된다.