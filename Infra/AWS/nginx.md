  
  # nginx 로드 밸런싱 설정
  
   - 필자는 무중단 자동 배포를 테스트 하기 위해서 nginx 로드 밸런싱 설정을 해야 했고 이를 기록해 놓기로 하여 이 글을 쓴다.


  
      __로드 밸런싱을 적용하는 이유__
      
        - WAS 가 한대만 서버에서 띄워져 있다고 한다면 만약 1대의 WAS 가 죽게되면 서비스는 정지해 버리게 된다.
        - 이를 해결하기 위해 WAS 를 1대 더 띄워서 요청을 분산시키는 작업을 nginx에서 하게될 것이다. 이것이 바로 로드밸런싱 이다.

   ![image](https://user-images.githubusercontent.com/79154652/166412080-6cac05e1-6436-455e-a200-676efed0f8ca.png)

   - 사용자는 서비스 주소로 접속합니다.(80 혹은 443 포트)
   - Nginx는 사용자의 요청을 받아 현재 연결된 WAS로 요청을 전달합니다.
   
   
   
  ## Nginx 설치
  
   - `sudo apt-get install nginx` 로 Nginx를 설치한다.
   - `/etc/nginx/nginx.conf` 를 수정한다.
   
~~~
    user www-data;
    worker_processes auto;
    pid /run/nginx.pid;
    include /etc/nginx/modules-enabled/*.conf;

    events {
            worker_connections 768;
            # multi_accept on;
    }

    http {

    upstream test {
            server IP:PORT;
            server IP:PORT;
    }
            

            sendfile on;
            tcp_nopush on;
            tcp_nodelay on;
            keepalive_timeout 65;
            types_hash_max_size 2048;
            

            include /etc/nginx/mime.types;
            default_type application/octet-stream;

            
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
            ssl_prefer_server_ciphers on;

            ##
            # Logging Settings
            ##

            access_log /var/log/nginx/access.log;
            error_log /var/log/nginx/error.log;

            ##
            # Gzip Settings
            ##

            gzip on;

            # gzip_vary on;
            # gzip_proxied any;
            # gzip_comp_level 6;
            # gzip_buffers 16 8k;
            # gzip_http_version 1.1;
            # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

            ##
            # Virtual Host Configs
            ##

            include /etc/nginx/conf.d/*.conf;
            include /etc/nginx/sites-enabled/*;

            server {
                    listen 80;

            location / {
                    proxy_pass http://test;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header Host $http_host;

                    proxy_buffer_size          128k;
                    proxy_buffers              4 256k;
                    proxy_busy_buffers_size    256k;
                    }
            }
}
~~~
    
    
    
- 중요하게 설정해야 할 부분은 `location` 의 proxy_pass 부분과 `upstream` 부분이다.
    - `upstream`은 설정해놓은 서버로 로드밸런싱 한다는 블럭이다. 따로 로드밸런싱 전략을 설정하지 않으면 `라운드 로빈` 방식으로 로드밸런싱 된다.
        - round-roibn : 라운드 로빈 방식으로 서버 할당
        - least-connected : 커넥션이 가장 적은 서버를 할당
        - ip-hash : 클라이언트 ip를 해쉬한 값을 기반으로 특정 서버 할당
    - `location`의 proxy_pass는 80포트로 요청이 들어 왔을 때 해당 url로 redirect 한다는 의미이다.
        - proxy_pass 의 http://test 는 upstream 에서 설정해준 `test`를 입력해주면 test에 설정된 서버로 로드밸런싱 된다.


필자는 위와 같이 설정을 하고 테스트를 해보았는데. `proxy_pass`가 작동하지 않는것 같았다.

그래서 `/etc/nginx/site-enabled/default`에도 proxy_pass를 설정해 주었다.


default
~~~
location / {
          proxy_pass http://test;

}
~~~

위와 같이 추가하였다. test가 동작하는 이유는 nginx.conf 에 `/etc/nginx/site-enabled/*` 가 include 되어 있기 때문이다.

위와 같이 설정해서 다시 테스트 해본 결과. 설정한 2대의 WAS로 분산되서 요청이 가는 것을 확인 할 수 있었다.
