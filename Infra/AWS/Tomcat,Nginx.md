필자는 EC2 인스턴스에서 스프링 프로젝트를 배포하기 위해 tomcat을 구동시켜야 했다.

URL로 url:8080 를 입력하면 Tomcat 포트인 8080으로 접속이 가능하나,

유저들이 접속하는 DNS인 http 80 포트로 접속할려면 nginx 인 80포트에서 Tomcat 8080포트로

redirect를 해주는 방법을 기록해 놓을려고 한다.

## Nginx 설치

```jsx
sudo apt-get install nginx
```

```jsx
//nginx 시작
sudo service nginx start
```

Nginx 를 서버에서 시작하였으면 EC2 인스턴스 ip주소에 80포트를 열어주어야 한다.

![image](https://user-images.githubusercontent.com/79154652/139858748-3b6d0175-0460-424d-805d-82b3cd6fabbd.png)

## Nginx의 핵심 nginx.conf파일

/etc/nginx 경로에 위치하는 nginx.conf 가 메인 파일이다.

```jsx
vi nginx.conf
//파일 내용을 확인해보면
```

![image](https://user-images.githubusercontent.com/79154652/139858792-ee2e2325-c0f5-4733-b387-81f249be945d.png)

위와 같은 내용을 볼수 있다.

- /etc/nging/conf.d의 모든 .conf 파일들과
- /etc/nginx/sites-enabled의 모든 파일을 include를 해온다는 뜻

우리가 건드려야할 곳은 sites-enabled 이다.

/etc/nignx 디렉토리 안에 sites-available 디렉토리와 site-enabled 디렉토리가 있다.

![image](https://user-images.githubusercontent.com/79154652/139858842-84aae496-2855-4930-a79d-1e58a76e0194.png)

우리가 실제로 설정하고 저장해야 하는 파일들은 .conf확장자로 sites-available 에 저장하고,

심볼릭 링크들을 sites-enabled에 위치시켜야 한다.

![image](https://user-images.githubusercontent.com/79154652/139858880-c7c103dd-e866-4bd6-af15-90a306ae8377.png)

## sites-available 에 설정파일 생성 및 적용

sites-available 에 들어가 .conf파일을 생성

```jsx
cd /etc/nginx/sites-available
sudo vim tomcat.conf
```

```jsx
//tomcat.conf 파일에 다음 내용을 붙여넣는다
upstream tomcat{
			ip_hash;
			server (ec2 ip주소 or 도메인):8080;
}

server {
    listen 80;
    listen [::]:80;

    server_name localhost;
 
    location / {
         proxy_pass http://tomcat;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header Host $http_host;
    }
}
```

위의 코드를 간단히 설명하자면

80포트로 들어오는 모든 request에 대해 8080포트로 접속하게 해주는 것

![image](https://user-images.githubusercontent.com/79154652/139859069-aab8c5cd-2b4f-4057-a0af-4a6feaa91e6f.png)

## sites-enabled 에 심볼릭 링크만들기

위의 tomcat.conf 파일을 저장한 후 sites-enabeld에 심볼링 링크를 만들어 준다.

```jsx
sudo ln -s /etc/nginx/sites-available/tomcat.conf /etc/nginx/sites-enabled
```

그 다음 sites-enabled로 이동해 확인

```jsx
cd /etc/nginx/sites-enabled
ls -al
```

위 명령을 통해 확인 해 보면 default 심볼릭 링크도 있는 것을 확인 할 수 있다. 이것은 원래부터 존재하는 기본 설정파일 이므로 삭제해주어야 필자가 만든 tomcat.conf를 적용 가능하다.

```jsx
sudo rm default

//삭제한후

sudo service nginx restart
//nginx를 재구동하면 성공
```

nginx 재구동 후 ip주소 혹은 도메인주소로 입력하면 Tomcat화면이 나오는것을 확인 할 수 있다

![image](https://user-images.githubusercontent.com/79154652/139859116-9b2549ae-125e-4d93-a8a8-c0096e772d0e.png)
