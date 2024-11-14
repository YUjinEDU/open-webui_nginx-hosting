# Open WebUI with Nginx Reverse Proxy

이 프로젝트는 [open-webui](https://github.com/open-webui/open-webui)에 nginx를  추가한 프로젝트 입니다.

##  Open WebUI Demo
![Open WebUI Demo](./demo.gif)



## 프로젝트 개요

이 프로젝트는 무료 버전의 Chat GPT를 사용하면서 느낀 불편함을 해결하고, 경제적으로 GPT 모델을 사용할 수 있는 방법을 찾기 위해 시작되었습니다. 무료 버전의 GPT는 사용량에 제한이 있고, 유료 버전은 월 $20의 비용이 발생합니다. 하지만 최근 출시된 4o-mini 모델은 매우 경제적이며, 성능도 4o 모델과 크게 차이가 나지 않습니다. 이를 API로 사용하면 터미널에서 텍스트로 요청하는 불편함을 해결할 수 있습니다.

Streamlit을 사용하여 웹 사이트를 만들어 Chat GPT 페이지처럼 채팅을 할 수 있지만, 원하는 수준으로 만들기에는 어려움이 있었습니다. 그래서 찾은 것이 Open WebUI입니다. Open WebUI는 로컬에서 작동하는 페이지이지만, 이를 홈서버에서 호스팅하여 외부에서도 사용할 수 있도록 추가하였습니다.

## API 사용의 이점

1. **현재 GPT의 무료 사용량과 한계**
   - 무료 버전의 GPT는 사용량에 제한이 있어 충분히 활용하기 어렵습니다.

2. **유료 버전의 가격**
   - 유료 버전은 월 $20의 비용이 발생하여 경제적 부담이 큽니다.

3. **4o-mini 모델의 토큰당 가격과 4o 모델의 토큰당 가격**
   - 4o-mini 모델은 매우 경제적이며, 성능도 4o 모델과 크게 차이가 나지 않습니다.

4. **4o-mini와 4o 성능 차이**
   - 4o-mini 모델은 4o 모델에 비해 성능 차이가 크지 않으면서도 비용이 저렴합니다.

5. **API로 4o-mini를 사용하면 좋은 점**
   - 터미널에서 텍스트로 요청하는 불편함을 해결할 수 있습니다.
   - 웹 인터페이스를 통해 쉽게 사용할 수 있습니다.

## open-webui 호스팅의 이점

- **홈서버 운영 경험**
  - 홈서버를 운영하면서 서버 관리와 호스팅 경험을 쌓을 수 있습니다.

- **경제적으로 GPT 사용**
  - 4o-mini 모델을 사용하여 경제적으로 GPT를 사용할 수 있습니다.

- **추후에 Ollama 모델도 사용 가능**
  - Ollama 모델을 추가하여 다양한 모델을 사용할 수 있습니다.

- **다양한 모델 사용 가능**
  - Open WebUI를 통해 다양한 GPT 모델을 사용할 수 있습니다.

- **User 롤 추가**
  - 팀원들에게 계정을 부여하여 함께 사용할 수 있습니다.

- **문서를 미리 등록할 수 있음**
  - 문서를 미리 등록하여 효율적으로 사용할 수 있습니다.

## 작업 설명

### Docker 실행

```sh
docker run -d -p 80:8080 -e OPENAI_API_KEY=your_secret_key -e DOMAIN=your-domain-address -v open-webui:/app/backend/data --name open-webui --restart always homesever/open-webui-nginx:1.0
```
2가지를 작성하고 실행 해야 합니다.

- `OPENAI_API_KEY =your_secret_key`
     - your_secret_key는 api키

- `DOMAIN=your-domain-address`
   - your-domain-address는 도메인 주소 입니다.

브라우져에 your-domain-address를 입력하시면 접속이 가능합니다.


### Nginx 설정
Nginx 설정 파일은 도메인 환경 변수를 사용하여 자동으로 설정됩니다.
```
// filepath: /c:/project/open-webui-main/nginx/nginx.conf
user nginx;
worker_processes auto;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    keepalive_timeout 65;
    
    server {
        listen 80;
        server_name ${DOMAIN}; # 🌟 환경 변수로 도메인 이름 설정 🌟

        location / {
            proxy_pass http://open-webui:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ollama/ {
            proxy_pass http://ollama:11434/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 443 ssl;
        server_name ${DOMAIN}; # 🌟 환경 변수로 도메인 이름 설정 🌟

        ssl_certificate /etc/ssl/certs/${DOMAIN}.crt; # 🌟 인증서 파일로 변경 🌟
        ssl_certificate_key /etc/ssl/certs/${DOMAIN}.key; # 🌟 키 파일로 변경 🌟

        location / {
            proxy_pass http://open-webui:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ollama/ {
            proxy_pass http://ollama:11434/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```



인증서 생성
Dockerfile에서 도메인 환경 변수를 사용하여 인증서를 자동으로 생성합니다.
```
// filepath: /c:/project/open-webui-main/Dockerfile
# ...existing code...
ARG DOMAIN

# ...existing code...

COPY ./nginx/nginx.conf /etc/nginx/nginx.conf

RUN sed -i "s/\${DOMAIN}/${DOMAIN}/g" /etc/nginx/nginx.conf

# 인증서 생성
RUN mkdir -p /etc/ssl/certs && \
    openssl genrsa -out /etc/ssl/certs/${DOMAIN}.key 2048 && \
    openssl req -new -key /etc/ssl/certs/${DOMAIN}.key -out /etc/ssl/certs/${DOMAIN}.csr -subj "/CN=${DOMAIN}" && \
    openssl x509 -req -days 365 -in /etc/ssl/certs/${DOMAIN}.csr -signkey /etc/ssl/certs/${DOMAIN}.key -out /etc/ssl/certs/${DOMAIN}.crt

# ...existing code...
```

