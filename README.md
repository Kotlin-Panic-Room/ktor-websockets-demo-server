# ktor-wesockets-demo-server

>Ktor Framework는 주로 코틀린을 대상으로한 Web Framework입니다.

이 글에서 두 가지 애플리케이션으로 구성되는 간단한 채팅 서비스를 구현할 것입니다.

* **채팅 서버 어플리케이션**은 채팅 사용자의 연결을 수락 및 관리하고 메시지를 수신하여 연결된 모든 클라이언트에 배포합니다.

* **채팅 클라이언트 어플리케이션**을 사용하면 사용자가 공통 채팅 서버에 참여하고 다른 사용자에게 메시지를 보내고 터미널에서 다른 사용자의 메시지를 볼 수 있습니다.

애플리케이션의 두 부분 모두에 대해 WebSocket에 대한 Ktor에서 지원 되는 부분을 사용합니다. Ktor는 서버 측 프레임워크이자 클라이언트 측 프레임워크이기 때문에 클라이언트 구축과 관련하여 채팅 서버 구축에서 얻은 지식을 재사용할 수 있습니다.

Ktor를 사용하여 WebSocket으로 작업하는 방법, 클라이언트와 서버 간에 정보를 교환하는 방법, 그리고 동시에 여러 연결을 관리하는 방법에 대한 기본 아이디어를 얻을 수 있습니다.

### 왜 웹소켓인가요?

WebSocket은 채팅이나 간단한 게임과 같은 애플리케이션에 매우 적합합니다. 채팅 세션은 일반적으로 오래 지속되며 클라이언트는 장기간에 걸쳐 다른 참가자로부터 메시지를 수신합니다. 채팅 세션도 양방향입니다. 클라이언트는 채팅 메시지를 보내고 다른 사람의 채팅 메시지를 보고 싶어합니다.

일반 HTTP 요청과 달리 WebSocket 연결은 오랫동안 열려 있을 수 있으며 프레임 형태로 클라이언트와 서버 간에 데이터를 교환하기 위한 쉬운 인터페이스를 가지고 있습니다. 프레임은 다양한 유형(텍스트, 바이너리, 닫기, 핑/퐁)으로 제공되는 WebSocket 메시지로 생각할 수 있습니다. Ktor는 WebSocket 프로토콜을 통해 높은 수준의 추상화를 제공하기 때문에 텍스트 및 바이너리 프레임에 집중하고 다른 프레임 처리는 프레임워크에 맡길 수 있습니다.

WebSocket은 또한 널리 지원되는 기술입니다. 모든 최신 브라우저는 기본적으로 WebSocket과 함께 작동할 수 있으며 WebSocket과 함께 작동하는 프레임워크는 많은 프로그래밍 언어와 플랫폼에 존재합니다.

이제 프로젝트 구현에 사용할 기술에 대한 확신이 생겼으니 설정부터 시작하겠습니다!

### 프로젝트 만들기

[해당 주소](https://start.ktor.io/?_ga=2.134289528.801741867.1650019331-1632403444.1647769917&_gl=1*1joqrz8*_ga*MTYzMjQwMzQ0NC4xNjQ3NzY5OTE3*_ga_VCMCSM1ZZ7*MTY1MDAyOTQwMC4xLjEuMTY1MDAyOTQwNS4w#/settings)로 들어가면 Ktor 프로젝트를 만들 수 있습니다.

Plugin에서 Routing과 Websockets만 선택하고 프로젝트를 생성해줍니다.

---

## 프로젝트 살펴보기

![](https://velog.velcdn.com/images/roo333/post/2d28a963-9fd1-462e-9d6d-79a033be55c9/image.png)

* build.gradle.kts 파일에는 Ktor 서버 및 플러그인에 필요한 종속성이 포함되어 있습니다.

* main/resources 폴더에는 구성 파일이 포함되어 있습니다.

* main/kotlin 폴더에는 생성된 소스 코드가 포함되어 있습니다.

### Dependencies

먼저 build.gradle.kts 파일을 열고 추가된 Dependencies을 살펴보겠습니다.

![](https://velog.velcdn.com/images/roo333/post/03877dfe-06c5-456f-b64e-1c4cf25a7f32/image.png)

Dependencies을 하나씩 간단히 살펴보겠습니다.

* ktor-server-core는 Ktor의 핵심 구성 요소를 우리 프로젝트에 추가합니다.

* ktor-server-netty는 프로젝트에 Netty 엔진을 추가하여 외부 애플리케이션 컨테이너에 의존하지 않고도 서버 기능을 사용할 수 있도록 합니다.

* ktor-server-websockets를 사용하면 채팅의 주요 통신 메커니즘인 WebSocket 플러그인을 사용할 수 있습니다.

* logback-classic은 SLF4J 구현을 제공하여 콘솔에서 멋지게 형식화된 로그를 볼 수 있도록 합니다.

* ktor-server-test-host 및 kotlin-test-junit를 사용하면 프로세스에서 전체 HTTP 스택을 사용하지 않고도 Ktor 애플리케이션의 일부를 테스트할 수 있습니다.

### application.conf 및 logback.xml 구성

생성된 프로젝트에는 리소스 폴더에 있는 application.conf 및 logback.xml 구성 파일도 포함되어 있습니다.

* application.conf는 HOCON 형식의 구성 파일입니다. Ktor는 이 파일을 사용하여 실행해야 하는 포트를 결정하고 애플리케이션의 진입점도 정의합니다.

![](https://velog.velcdn.com/images/roo333/post/5c23c815-628c-48de-9d36-80f44f7642bd/image.png)

* logback.xml은 우리 서버의 기본 로깅 구조를 설정합니다. 

### 소스 코드


![](https://velog.velcdn.com/images/roo333/post/9c170840-60e4-4088-9269-4b8047d9bb97/image.png)

* configureRouting은 현재 아무 것도 하지 않는 plugins/Routing.kt에 정의된 기능입니다.

![](https://velog.velcdn.com/images/roo333/post/b2dd12d8-1d4a-4df4-84f0-0792e30e8163/image.png)

* configureSockets는 WebSockets 플러그인을 설치하고 구성하는 plugins/Sockets.kt에 정의된 기능입니다.

![](https://velog.velcdn.com/images/roo333/post/9fbaff35-87dc-41b1-981d-bfacb036c6d7/image.png)

---

## 첫번째 에코서버

### 에코서버 구현하기

WebSocket 연결을 수락하고, 텍스트 콘텐츠를 수신하고, 클라이언트로 다시 보내는 작은 "에코" 서비스를 구축하여 서버 개발 여정을 시작하겠습니다. 

먼저 WebSockets Ktor 플러그인을 설치하여 Ktor 프레임워크에서 제공하는 WebSocket 관련 기능을 활성화합니다. 이를 통해 WebSocket 프로토콜에 응답하는 라우팅의 끝점을 정의할 수 있습니다(이 경우 경로는 /chat임). webSocket 경로 기능의 범위 내에서 클라이언트와 상호 작용하기 위해 다양한 방법을 사용할 수 있습니다(DefaultWebSocketServerSession 수신기 유형). 여기에는 메시지를 보내고 받은 메시지를 반복하는 편리한 방법이 포함됩니다.

우리는 텍스트 콘텐츠에만 관심이 있기 때문에 수신 채널을 반복할 때 수신하는 텍스트가 아닌 프레임을 건너뜁니다. 그런 다음 수신된 텍스트를 읽고 "YOU SAID:"라는 접두사를 사용하여 사용자에게 바로 보낼 수 있습니다.

이 시점에서 우리는 이미 완벽하게 작동하는 에코 서버를 구축했습니다. 사용자가 보낸 모든 것을 되돌려 보내는 작은 서비스입니다. 사용해 봅시다!

### 에코 서버를 사용해보기

지금은 웹 기반 WebSocket 클라이언트를 사용하여 에코 서비스에 연결하고 메시지를 보내고 에코된 응답을 받을 수 있습니다. 서버 측 기능 구현을 마치면 Kotlin에서 자체 채팅 클라이언트도 빌드합니다.

우리 서버의 Application.kt에서 fun main의 정의 옆에 있는 여백에 있는 Play 버튼을 눌러 서버를 시작합시다. 프로젝트 컴파일이 완료되면 IntelliJ IDEA 실행 도구 창에서 서버가 실행 중이라는 확인이 표시되어야 합니다.

![](https://velog.velcdn.com/images/roo333/post/4173f424-0af0-4262-821c-952f353f82cd/image.png)



서비스를 시도하기 위해 Postman을 사용하여 ws://localhost:8080/chat에 연결하고 WebSocket 요청을 할 수 있습니다.

![](https://velog.velcdn.com/images/roo333/post/a4fc78ee-d358-4988-9c73-a9afbc5962d5/image.png)



그런 다음 편집기 창에 모든 종류의 메시지를 입력하고 로컬 서버로 보낼 수 있습니다. 모든 것이 계획대로 진행되면 보내고 받은 메시지와 메시지 창에서 에코 서버가 의도한 대로 작동하고 있음을 표시해야 합니다.

이를 통해 이제 WebSocket을 통한 양방향 통신을 위한 견고한 기반을 갖게 되었습니다. 다음으로, 여러 사용자가 다른 사람과 메시지를 공유할 수 있도록 프로그램을 채팅 서버와 더 유사하게 확장해 보겠습니다.

## 메시지 교환

에코 서버를 실제 채팅 서버로 바꿔야합니다. 동일한 사용자의 메시지에 모두 동일한 사용자 이름으로 태그가 지정되었는지 확인해야 합니다. 또한 메시지가 실제로 브로드캐스트되도록 하고 연결된 다른 모든 사용자에게 전송됩니다.

### 모델 연결

이 두 가지 기능을 모두 사용하려면 서버가 보유하고 있는 연결을 추적할 수 있어야 합니다. 즉, 메시지를 보내는 사용자와 메시지를 브로드캐스트할 대상을 알아야 합니다.

Ktor는 수신 및 발신 채널, 편리한 통신 방법 등을 포함하여 WebSocket을 통해 통신하는 데 필요한 모든 것을 포함하는 DefaultWebSocketSession 유형의 개체를 사용하여 WebSocket 연결을 관리합니다. 지금은 사용자 이름 할당 문제를 단순화하고 각 참가자에게 카운터를 기반으로 자동 생성된 사용자 이름을 제공할 수 있습니다.

Connection.kt라는 com.example 패키지의 새 파일에 다음 구현을 추가합니다.

![](https://velog.velcdn.com/images/roo333/post/5978b078-bc8e-4fb8-a8b9-beb1556febad/image.png)

카운터에 대한 스레드로부터 안전한 데이터 구조로 AtomicInteger를 사용하고 있습니다. 이렇게 하면 두 개의 연결 개체가 별도의 스레드에서 동시에 생성되는 경우에도 두 사용자가 자신의 사용자 이름에 대해 동일한 ID를 받지 않게 됩니다.

### 연결 처리 및 메시지 전송 구현

이제 연결 개체를 추적하고 올바른 사용자 이름이 접두사로 붙은 모든 연결된 클라이언트에 메시지를 보내도록 서버 프로그램을 조정할 수 있습니다. plugins/Sockets.kt의 라우팅 블록 구현을 다음 코드로 바꿉니다.

![](https://velog.velcdn.com/images/roo333/post/64ace265-7ba4-45fc-8dc4-09a431946a92/image.png)



우리 서버는 이제 (스레드로부터 안전한) Connections 컬렉션을 저장합니다. 사용자가 연결할 때 연결 개체(자신에게 고유한 사용자 이름을 할당함)를 만들고 컬렉션에 추가합니다. 그런 다음 사용자에게 인사하고 현재 연결 중인 사용자 수를 알려줍니다. 사용자로부터 메시지를 받으면 연결 개체와 연결된 고유한 이름을 접두사로 붙이고 현재 활성화된 모든 연결로 보냅니다. 마지막으로 연결이 종료될 때 컬렉션에서 클라이언트의 Connection 개체를 제거합니다. 정상적으로 들어오는 채널이 닫힐 때 또는 클라이언트와 서버 간의 네트워크 연결이 예기치 않게 중단된 경우 예외가 발생합니다.

서버가 이제 올바르게 작동하는지 확인하기 위해 사용자 이름을 할당하고 연결된 모든 사람에게 브로드캐스트합니다. 재생 버튼을 사용하여 애플리케이션을 다시 한 번 실행하고 Postman을 사용하여 ws://localhost:8080/chat에 연결할 수 있습니다. 이번에는 두 개의 개별 탭을 사용하여 메시지가 제대로 교환되었는지 확인할 수 있습니다.

![](https://velog.velcdn.com/images/roo333/post/f366352e-d6f6-42e0-b0db-6f4652c4a63b/image.png)

![](https://velog.velcdn.com/images/roo333/post/d047ce45-4e5b-4fe8-ae6f-07cbbda2596e/image.png)


보시다시피 완성된 채팅 서버는 이제 여러 참가자와 메시지를 주고받을 수 있습니다. 자유롭게 탭을 몇 개 더 열고 여기에서 만든 것을 가지고 놀아보세요!
