# Docker 
- container 기술을 기반으로 앱 실행 환경을 격리하고 관리하는 도구 
- OS 레벨의 가상화
- 컨테이너 단위로 테스트/배포 가능 

## Docker Hub
- tag를 통해 image를 구분하고 버전 관리 

## Docker 주요 명령어 
```bash
docker build -t {image_name:tag} . # 이미지 생성 
docker tag {image_name:tag} {id/image_name:tag} # 생성한 이미지에 태그 부여
docker push {id/image_name:tag} # 도커 허브 원격 저장소에 이미지 push 
docker pull {id/image_name:tag} # 원격 저장소에서 이미지 pull 
docker run {id/image_name:tag} # 이미지로부터 컨테이너를 만들고 시작하는 (create & start) 한 번짜리 축약형 명령 // docker create <image:tag> docker start <container> 
```
- `--name {container_name}` 컨테이너 이름 지정
- `-d` detach. 백그라운드 실행
  - 터미널에서 다른 명령을 계속 칠 수 있다. 
  - 안 치면 터미널이 계속 묶여있게 된다. 
- `-p {외부포트}:{내부포트}` 외부 호스트의 포트:컨테이너 내부 포트 포워딩
- `--env-file {env파일 경로}` 환경변수 설정 
```bash
docker logs {container_name}
```
- `-f` follow. 로그 실시간 출력
```bash
docker exec -it {container_name}
```
`-i` 표준 입력을 열어둠. 명령이 끝날 때까지 입력을 계속 받을 수 있게 함.

`-t` 가짜 터미널을 붙여서 터미널처럼 줄편집, 색상, 프롬프트 등을 정상 동작하게 함. 
```bash
docker container stop # 컨테이너 중지 
docker container rm # 컨테이너 삭제 
docker image rm {id/image_name:tag} # 이미지 삭제 
```
- 컨테이너가 그 이미지를 참조중이거나 같은 이미지 id를 가리키는 다른 태그가 있으면 삭제 안됨 
```bash
docker container ls # 컨테이너 목록 조회. docker ps도 됨. 
```
- `-a` 종료된 컨테이너도 포함해서 전체 조회 
```bash
docker image ls # 이미지 목록 조회 
```

# EC2
- AWS에서 제공하는 VM 클라우드 서비스 
- 지정된 OS만 깔린 가상 컴퓨터 한 대를 빌려준다 

## 인스턴스 
- EC2 서비스에서 빌린 VM 하나하나 

## 인바운드 규칙 
- 인스턴스 밖에서 안으로 들어오는 트래픽에 대한 규칙 
- 기본값은 아무 포트도 안 열려 있음 
- ssh 접속: 터미널 접근 (PORT 22)
- 웹 서버 접속: http(PORT 80), https(PORT 443)

## MobaXterm에서 docker 접속
```bash 
sudo su # superuserdo, switch user
```
- 관리자 권한으로 사용자 전환을 실행해서 root 계정으로 들어가겠다. 
```bash
apt update # 설치 가능한 프로그램 목록을 최신으로 갱신 
```
- Advanced Packaging Tool 

- EC2 인스턴스 터미널에서 docker 조작
```bash
apt install docker.io
docker login -u sohee123
docker pull sohee123/wisheasy:sha-abc123
docker image ls
docker run -d --name wisheasy_web -p 8000:8000
docker ps
docker logs -f wisheasy_web
```
- `--name`은 컨테이너 이름 정해주는 거 

# Nginx 설정 파일 
- 기본적으로 두 개의 설정 파일이 있음

`/etc/nginx/nginx.conf` : 전역 설정
`/etc/nginx/sites-available/default` : 기본 서버블록 설정
- nginx가 실행되는 공간 내에 요청을 받을 서버가 하나 뿐이라면, default를 편집해서 사용
- 여러 서버가 서비스되고 있다면 default를 비활성화하고 개별 파일을 통해 설정해줘야 함

## Ngnix 설치 및 동작 확인 
```bash 
apt install nginx
systemctl status nginx
curl http://localhost
```
- IPv4 주소 접속 확인(Nginx 기본 페이지 렌더링)

## 설정 파일 수정 
- nano 편집기를 통한 설정 파일 수정 
```bash 
server {
    # 80번 포트에서 들어오는 요청을 받습니다.
    listen 80;

    # 이 서버가 응답할 도메인 이름이나 IP 주소를 지정합니다.
    # 지금은 EC2의 IPv4 주소를 적고, 도메인을 구매하면 도메인 이름으로 바꿉니다.
    server_name 15.164.97.227;

    # Django 애플리케이션으로 요청을 넘겨주는 부분입니다.
    location / {
        # 이 헤더들은 클라이언트의 원래 요청 정보를 그대로 Django에게 전달하는 중요한 역할을 합니다.
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 요청을 넘겨줄 주소. Django 컨테이너가 8000번 포트에서 실행 중이므로 아래와 같이 설정합니다.
        proxy_pass http://127.0.0.1:8000;
    }
}
```
## 문법 검사 및 재시작 
```bash
ngnix -t
systemctl reload nginx
```
- http 접속 확인(웹 앱 정상 실행)

## 접속 로그 확인 
- Nginx 로그 
```bash 
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

- Docker 로그 
```bash 
docker logs <container_id>
```

# SSL 인증서 발급 및 등록 

## SSL certificate
- 웹사이트의 신원을 증명하고, 데이터를 암호화해서 안전하게 전송하도록 해주는 디지털 증명서 
- 일반적으로 Nginix를 통해 관리됨 
  - 인증 key 관리 
  - http -> https 리디렉션 

---

- 보안 그룹에서 443 PORT 열기

- 호스트 엔진에 certbot 설치
  - `apt install certbot python3-certbot-nginx`

- SSL 인증서 발급
  - `certbot --nginx -d {domain.com} -d {www.domain.com} --redirect`

- Nginx 설정 파일 수정
  - `nano /etc/nginx/sites-available/default`
```bash 
    # HTTP 요청 → HTTPS 리디렉션
server {
    listen 80;
    server_name domain.com www.domain.com;

    return 301 https://$host$request_uri;
}

# HTTPS 요청 처리
server {
    listen 443 ssl;
    server_name domain.com www.domain.com;

    # SSL 인증서 경로 (Certbot 발급 기준)
    ssl_certificate     /etc/letsencrypt/live/domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;

    # SSL 보안 설정 (권장)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://127.0.0.1:8000;  # uvicorn 내부 포트
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
- 문법 검사 `nginx -t`
- ngnix 재실행 `systemctl restart nginx`
- 자동갱신 적용 여부 확인 `certbot renew --dry-run`
- https 접속 확인