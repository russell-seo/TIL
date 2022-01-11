
# Firebase Cloud Message(FCM)

  실시간 서버 푸시를 구현하는 방법이 소켓 프로그램을 통한 서버와 연결을 지속한 상태에서 데이터를 송수신 하는 방법 외에 `FCM(Firebase Cloud Message)` 를 이용하는 방법이 있다.
  
  Firebase는 구글의 모바일 앱 개발 통합 플랫폼이며, Firebase에서 제공하는 여러가지 서비스 중 하나가 FCM 입니다.
  
  
  
  ## FCM 동작 원리
  
  FCM 동작원리는 두 단께로 나누어 살펴 볼 수 있다.
  
   - __앱을 위한 키를 FCM 서버를 통해 얻는 단계__

   ![image](https://t1.daumcdn.net/cfile/tistory/99886F465A504D5C17)
   
   1. 스마트폰에 앱이 설치되거나, 로그인을 하는 순간 Firebase 서버에 키 획득을 위한 요청을 보낸다. Firebase 서버에 토큰를 위한 요청은 안드로이드
      시스템에 의해 자동화 되므로 개발자가 구현할 로직은 없습니다. FCM 방식을 위한 라이브러리와 플러그인이 설치되어야 하므로 시스템이 이를 인지하여 
      키 요청을 보낼 수 있게 되는 구조이다.
      
   2. Firebase 서버에서 토큰을 만들어 스마트폰에 전달합니다.키가 전달되면 시스템은 인텐트를 발생해 앱의 서비스를 구동시킨다. 개발자가 작성한 서비스가
      실행되고 서비스 내에서 토큰 값을 획득합니다.
      
   3. `앱에 전달된 토큰을 서버에 전송합니다.` 실시간 데이터 푸시 기능이 필요한 곳은 서버이며, 서버에서 전달된 토큰을 이용하여 데이터를 전송한다.
   4. `서버에서는 전달된 토큰값을 영속화한다`. 보통 DB에 저장해 둔다. 이렇게 하면 서버 DB에는 앱이 깔린 모든 클라이언트 스마트폰의 토큰이 저장된다.


   - __서버에서 데이터를 스마트폰에 전달하는 단계. 이는 Firebase 서버를 이용하여 데이터가 앱에 전달되게 하는 방식__

   ![image](https://t1.daumcdn.net/cfile/tistory/992CD0475A504D8013)
   
  1. 서버에서 데이터를 스마트폰에 전달하기 위해 `DB에서` 토큰을 획득 한다.
  2. DB의 키와 실제 앱에 전송하고자 하는 데이터를 Firebase 서버에 전달된다. Firebase 서버와는 HTTP 통신이 이용되며 Firebase 서버에서 원하는 방식대로 데이터를 구성하여 요청이 이루어 진다.
  3. Firebase 서버에서는 전달받은 키값을 식별해 어떤 스마트폰의 앱인지를 식별한다. 결국 키값에 의해 스마트폰에 데이터를 전달할 수 있다. 
     개발자가 앱에 데이터를 받기 위한 서비스를 만들어 두면 Firebase 서버로 부터 데이터가 수신 될 때 마다 서비스가 실행되어 데이터를 획득
     
     
# FCM 구현
  
  - Google 어플리케이션 기본 사용자 인증정보(ADC) 활용한 구현
  - HTTP v1 REST 활용한 구현

# ADC 활용한 구현

   1. build.gradle에 `implementation "com.google.firebase:firebase-admin:8.1.0"` 추가
   2. 먼저 사용자 인증 정보를 확인한다.(초기화 작업)
  
  

  ## FcmInit
  먼저 Spring 을 올리면서 사용자정보를 인증한다.
  ~~~java
  @Component
public class FcmInit {

    private final String firebaseConfigPath = "비공개 키 경로";

    @PostConstruct
    public void fcminit() throws IOException {


        FileInputStream refreshToken = new FileInputStream(firebaseConfigPath);

        FirebaseOptions options = new FirebaseOptions.Builder()
                .setCredentials(GoogleCredentials.fromStream(refreshToken))
                .setDatabaseUrl("firebase DB 주소/")
                .build();

        if(FirebaseApp.getApps().isEmpty()){ //조건을 걸지 않으면 여러번 초기화하면서 장애가 발생
            FirebaseApp.initializeApp(options);
        }
    }
  
  ~~~
  
  특정 기기 1개에 전송
  
  ~~~java
  @Override
    public void sendMulti(List<String> token){
        MulticastMessage message = MulticastMessage.builder().setNotification(Notification.builder().setTitle("주임님").setBody("행복하세요?").build())
                .addAllTokens(token)
                .build();

        try {
            BatchResponse batchResponse = FirebaseMessaging.getInstance().sendMulticast(message);
            System.out.println("batchResponse = " + batchResponse);
        } catch (FirebaseMessagingException e) {
            e.printStackTrace();
        }

    }
  ~~~
  
  다수 기기에 같은 메시지 전송
  
  ~~~java
  @Override
    public void sendMessages(String token){

        Message message = Message.builder().setNotification(Notification.builder().setTitle("주임님").setBody("행복하세요?").build())
                                 .setToken(token)
                                 .build();
        try {
            String send = FirebaseMessaging.getInstance().send(message);
            System.out.println("send = " + send);
        } catch (FirebaseMessagingException e) {
            e.printStackTrace();
        }
    }
  
  ~~~
  
  일괄 메시지 전송
  
  ~~~java
  @Override
    public void sendMessageAll(List<FcmDto> fcmDtos, String title, String body) {
        System.out.println("fcmDtos = " + fcmDtos);
        List<Message> messages =
                fcmDtos.stream().map(j -> {
                    System.out.println("j.getDevice_token() = " + j.getDevice_token());
                    Message allUsers = Message.builder()
                            .setNotification(Notification.builder().setTitle(title + j.getMessage()).setBody(body + j.getMessage()).build())
                            .setToken(j.getDevice_token())
                            .build();
                    System.out.println("allUsers = " + allUsers);
                    return allUsers;
                }
        ).collect(Collectors.toList());
        System.out.println("메시지 = " + messages);

        BatchResponse batchResponse = null;
        try {
            batchResponse = FirebaseMessaging.getInstance().sendAll(messages);
        } catch (FirebaseMessagingException e) {
            e.printStackTrace();
        }
        System.out.println("batchResponse.getSuccessCount() = " + batchResponse.getSuccessCount());

    }
  ~~~
  
  
  

# FCM 구현(HTTP REST 활용한 코드)

  먼저 FCM 서버를 구현하기 위해 작성해야 하는 코드는 총 3개 이다.
  
  1. FireBase 로 부터 `AccessToken` 받는 코드
  2. PUSH 요청 보낼 `메시지` 만드는 코드
  3. FCM에 PUSH 요청을 위한 `HTTP통신 POST` 코드

  ## getAccessToken
  FireBase 서버에 PUSH 요청을 위해서는 승인된 AccessToken이 필요하다.

  ~~~java
    @Override
    @PostConstruct
    public String getAccessToken() throws IOException {
        String firebaseConfigPath = "firebase/logenapp-firebase-adminsdk-1e9lc-f5311c4208.json";

        GoogleCredentials googleCredentials =
                GoogleCredentials.fromStream(new ClassPathResource(firebaseConfigPath).getInputStream())
                .createScoped(ImmutableList.of("https://www.googleapis.com/auth/cloud-platform"));

        // accessToken 생성
        googleCredentials.refreshIfExpired();

        return googleCredentials.getAccessToken().getTokenValue();
    }
  ~~~
  
  ## makeMessage
  FireBase 서버에 보낼 PUSH 메시지를 만드는 코드
  ~~~java
  @Override
    public String makeMessage(String token, String title, String body) throws IOException {
        FcmMessage sendMessage = 
                FcmMessage.builder().message(
                            FcmMessage.Message.builder()
                                .token(token)
                                .notification(FcmMessage.Notification.builder()
                                                .title(title)
                                                .body(body)
                                                .build())
                            .build())
                .validate_only(false)
                .build();

        return objectMapper.writeValueAsString(sendMessage);
    }
  ~~~

## sendMessage
HTTP v1 프로토콜을 사용하여 알림 메시지를 보내는 방법이다.
특정 PUSH 알림이 필요한 API로 Request가 오면 FireBase 서버로 조건에 맞게 보낸다.

~~~java
@Override
    public void sendMessage(String token, String title, String body) throws IOException {
        String msg = makeMessage(token, title, body);

        OkHttpClient okHttpClient = new OkHttpClient();
        RequestBody requestBody = RequestBody.create(msg, MediaType.get("application/json; charset=utf-8"));
        Request request = new Request.Builder()
                .url(SEND_URL)
                .post(requestBody)
                .addHeader(HttpHeaders.AUTHORIZATION, "Bearer " + fcmService.getAccessToken())
                .build();


        Response execute = okHttpClient.newCall(request).execute();
    }
~~~

[참고]()
---
[https://firebase.google.com/docs/cloud-messaging/migrate-v1?authuser=0](https://firebase.google.com/docs/cloud-messaging/migrate-v1?authuser=0)
