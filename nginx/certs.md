## 디렉토리 생성
```mkdir -p ./nginx/certs```

# 개인 키와 인증서 서명 요청(CSR) 생성
```openssl genrsa -out ./nginx/certs/mydomain.key 2048```

# 인증서 서명 요청(CSR) 생성
```openssl req -new -key ./nginx/certs/mydomain.key -out ./nginx/certs/mydomain.csr```

# 인증서 자체 서명 (유효기간: 365일)
```
openssl x509 -req -days 365 -in ./nginx/certs/mydomain.csr -signkey ./nginx/certs/mydomain.key -out ./nginx/certs/mydomain.crt
```
