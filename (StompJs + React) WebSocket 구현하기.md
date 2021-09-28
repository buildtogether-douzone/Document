### (StompJs + React) WebSocket 구현하기

#### 1. 개발 환경

+ Spring Boot
+ Maven
+ Java8
+ React

#### 2.  Spring의 Pom.xml에 의존성 추가

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

#### 3. Spring Websocket 설정

+ WebSocketConfig.java

``` java
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.stereotype.Controller;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Controller 
@Configurable 
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
	
	@Override 
	public void configureMessageBroker(MessageBrokerRegistry config) { 
		config.enableSimpleBroker("/topic", "/queue"); 
		config.setApplicationDestinationPrefixes("/");
	} 
	
	@Override 
    public void registerStompEndpoints(StompEndpointRegistry registry) { 
		registry.addEndpoint("/start").setAllowedOrigins("http://localhost:9999").withSockJS(); 
	}
}
```

+ 설정에 대한 설명
  + addEndpoint(): 클라이언트에서 websocket에 접속하기위한 endpoint를 등록
  + setApplicationDestinationPrefixes(): 아무것도 안 적으면 모든 접근에 대한 메시지를 핸들러로 라우팅 하고 만약  "/" 가 아닌 "/app" 으로 적혀 있으면 /app으로 접근하는 메시지만 핸들러로 라우팅 한다.
  + enableSimpleBroker(): 메모리 기반 메시지 브로커가 /topic 접두사가 붙은 클라이언트로 메시지를 전달할 수 있도록 설정
    - 쉽게 이야기하면 클라이언트 A,B,C가 각각 /topic/master, /topic/sub, /topic/master를 구독하고 있을때 /topic/master로 메시지를 전송하면 클라이언트 A,C만 메시지를 받는 구조이다.



#### 4. Websocket Controller

+ SocketController.java

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SocketController {
	
	@Autowired 
	private SimpMessagingTemplate webSocket; 
	
	@MessageMapping("/sendTo") 
	@SendTo("/topics/sendTo") 
	public String sendToMessage() throws Exception { 
		return "5"; 
	}
	
	@MessageMapping("/Doctor") 
	public void sendToDoctor() { 
		webSocket.convertAndSend("/topics/doctor" , Math.random()); 
	}
	
	@MessageMapping("/Nurse") 
	public void sendToNurse() { 
		webSocket.convertAndSend("/topics/nurse" , Math.random()); 
	}
	
	@MessageMapping("/Reservation") 
	public void sendToReservation() { 
		webSocket.convertAndSend("/topics/reservation" , Math.random()); 
	}
	
	@MessageMapping("/Message") 
	public void sendMessage(String receiveUser) {
		webSocket.convertAndSend("/topics/message/" + receiveUser , Math.random());
	}
	
	@MessageMapping("/Template") 
	public void SendTemplateMessage() { 
		webSocket.convertAndSend("/topics/template" , "Template"); 
	} 
	
	@RequestMapping(value="/api") 
	public void SendAPI() { 
		webSocket.convertAndSend("/topics/api" , "API"); 
	}
}
```

+ @MessageMapping: 클라이언트에서 보내는 메시지를 매핑 ex. localhost:8080:/Nurse
+ convertAndSend(): 첫번째 param에 해당하는 토픽을 구독중인 클라이언트에게 메시지를 전송
  + 즉 webSocket.convertAndSend("/topics/nurse" , Math.random());  에서 "topics/nurse" 를 구독중인 클라이언트에게 전송



#### 5. Backend(React)

+ 라이브러리 다운로드

```bash
npm install react-stomp
or
yarn add react-stomp
```

+ react-stomp import 해서 사용하기

```react
import React, { useRef } from 'react'
import SockJsClient from 'react-stomp';

export default function Client(){
    const $websocket = useRef(null); 
    const [value, setValue] = useState('');

    return (
        <div>       
            <SockJsClient
              url= 'http://localhost:8080/ucare_backend/start'
              topics={['/topics/nurse']}
              onMessage={msg => { setValue(msg) }}
              ref={$websocket}
            />
        </div>    
    )
}
```

+ msg 에는 server에서  webSocket.convertAndSend("/topics/nurse" , Math.random()); 의 두 번째 파라미터인 Math.random() 값이 들어가 있음
+ 현재 페이지는 /topics/nurse 를 구독하고 있음 따라서 server의 convertAndSend에서 "/topics/nurse" 를 구독하고 있는 모든 사람 한테  Math.random() 을 보낸다는 의미임

