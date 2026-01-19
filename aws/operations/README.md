# 🐳 [글로벌/ppeum] AWS 설정 문서
이 문서는 글로벌 쁨 서버를 신규 구축 또는 장애 상황에서 재구축하기 위한 **사내 운영 표준 매뉴얼**입니다. <br>
이 문서는 아래 Step을 **순서대로 그대로 실행**하여 서버가 구축을 목표로 하고 있습니다.
<hr>

## 🧭 문서 목적

- 서버 재구축 절차 표준화
- 담당자 변경 시에도 동일한 방식으로 서버 재현
- PEM 키 관리 제거 및 AWS 콘솔 기반 접속

<hr>

## 👥 대상 독자

- 백엔드 개발자
- 인수인계 대상자

<hr>

## 📌 전제 조건

- AWS 콘솔 접근 권한
- EC2 / 보안 그룹 수정 권한
- Git 저장소 접근 권한
- 가비아 도메인 관리 권한

<hr>

## 📖 전체 흐름

1. EC2 생성 및 접속
2. Docker / Docker Compose 설치
3. 소스 코드 배포 및 환경 설정
4. (권한 설정 없음)
5. 서비스 실행 및 도메인 연결

<hr>

# 🔹 **STEP 1:** 환경 준비 – EC2 생성 및 OS 업데이트

## 1️⃣ EC2 인스턴스 생성

- AMI: **Ubuntu Server 22.04 LTS**
- Instance Type: **t3.small**
- 용량: 최소 15GB 이상으로 설정
- Storage: 기본 설정 사용
- Security Group 설정
    - SSH (22) – EC2 Instance Connect 용
    - HTTP (80)
    - HTTPS (443)

⚠️ SSH(22) 포트는 Instance Connect 사용을 위해 필수입니다.

<hr>

## 2️⃣ EC2 접속 – AWS EC2 Instance Connect (표준)
본 서버는 PEM Key를 사용하지 않고
**AWS EC2 Instance Conntect** 방식으로 접속합니다.

## 접속 절차
1. AWS 콘솔 → EC2
2. 대상 인스턴스 선택
3. **[연결(Connect)]** 버튼 클릭
4. **EC2 Instance Connect** 탭 선택
5. 사용자 이름 확인
```
ubuntu
```
6. **[연결]** 클릭
👉 브라우저 기반 SSH 접속(키 파일 불필요)

<hr>

## 3️⃣ OS 업데이트
```
sudo apt update
sudo apt upgrade -y
```
<hr>

# 🔹 **STEP 2:** 필수 패키지 설치 – Docker / Docker Compose
운영 서버는 Ubuntu 기본 docker.io 패키지를 사용하지 않고,
**Docker 공식 저장소 기준** 으로 설치합니다.

## 1️⃣ Docker 공식 저장소 등록
터미널 아래 명령어를 입력합니다.
```
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  |sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" \
  |sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
<hr>

## 2️⃣ Docker 설치
터미널 아래 명령어를 입력합니다.
```
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

### 설치되는 구성 요소
* **docker-ce :** Docker Engine (컨테이너 실행 핵심)
* **docker-ce-cli :** docker 명령어
*  **containerd.io :** 컨테이너 런타임
<hr>

## 3️⃣ Docker 권한 설정 (ubuntu 사용자)
```
sudo usermod -aG docker ubuntu
```
⚠️ 이후 Instance Connect **재접속 필요**
<hr>

## 4️⃣ Docker Compose 설치
```
sudo curl -L https://github.com/docker/compose/releases/download/v2.25.0/docker-compose-linux-x86_64 \
  -o /usr/local/bin/docker-compose
```
<hr>

## 5️⃣ 설치 확인
```
docker -v
docker compose version
```
<hr>

# 🔹 **STEP 3:** 소스 코드 배포 – Git Clone 및 환경 설정

## 1️⃣ 소스 코드 클론(clone)
backend, front 리포지터리 클론 필요
* docker-compose repo: **global_ppeum_docker(예시)**
* FE repo : **ppeum-next15-front(예시)**
* BE repo : **go-api(예시)**
```
cd ~ && mkdir app && cd app
git clone https://github.com/jun0925/global_ppeum_docker.git .
git clone https://github.com/tgkimgenius/ppeum-next15-front.git
git clone https://github.com/tgkimgenius/go-api.git
```
<hr>

## 2️⃣ 환경 변수 파일 설정
```
cp .env.example .env
```

go-api 경로 이동 - 하위에 .env 파일을 만들고 아래 코드를 넣습니다.
```
vim .env
```
go-api(.env)
```
ENV=local

DB_USERNAME=root
DB_PASSWORD='ppeum2024@)@$'
DB_HOST=139.150.65.34
DB_PORT=3306
DB_NAME=ppeum_homepage

TEST_DB_USERNAME=root
TEST_DB_PASSWORD=rootpass
TEST_DB_HOST=mariadb
TEST_DB_PORT=3306
TEST_DB_NAME=testdb

REDIS_HOST=redis:6379
REDIS_PORT=6379
REDIS_PASS=secret123

SWAGGER_HOST=asdfasfas
SWAGGER_PORT=8080
SWAGGER_BASE_PATH="/"

CORS_ALLOWED_ORIGINS="https://ppeum.com,http://admin-hub.ppeum.com,https://admin-hub.ppeum.com,http://localhost:*"
```
ppeum-next15-front 경로까지 이동 - 하위에 .env 파일을 만들고 아래 코드를 넣습니다.
```
vim .env
```
next.js(.env)
```
NEXT_PUBLIC_API_URL=https://api-hub.ppeum.com
NEXT_PUBLIC_API_BASE_URL=https://ppeumtest.api.namucrm.com

NEXT_PUBLIC_SUBDOMAIN=ppeumtest
NEXT_PUBLIC_LANG=ko

#Kakao URL로 리디렉션 (브라우저 접근)
NEXT_PUBLIC_KAKAO_REST_API_KEY=4b11e0de8e7caef60ec7ad0265bd2997

#서버 사이드 (브라우저 접근 불가)
KAKAO_REST_API_KEY=4b11e0de8e7caef60ec7ad0265bd2997

#이미지 URL
NEXT_PUBLIC_IMAGE_BASE_URL=https://newadmin.ppeum.com
```

## ✔️ .env 파일 필수 확인 항목
* APP_ENV
* APP_KEY
* DB 접속 정보
* 외부 서비스 연동 키

모든 도메인의 SSL을 발급하는 명령어
<hr>

# 🔹 **STEP 4:** 권한 설정 (별도 작업 없음)
본 서버는 Docker 컨테이너 내부에서 권한을 관리합니다. <br>
호스트 OS(Ubuntu)에서는 **별도의 권한 설정을 하지 않습니다.**
* storage, cache 권한 변경 X
* www-data 소유권변경 X
* ACL 설정 X
👉 권한 문제는 Dockerfile / 컨테이너 설정 기준으로 해결
<hr>

# 🔹 **STEP 5:** 서비스 가동 및 도메인 연결 (SSL 포함)
목적
* EC2 서버에 SSL 인증서 발급
* 도메인 DNS 연결
* Docker 서비스 기동
* HTTPS 접속 확인
<hr>

## 1️⃣ 사전 확인 사항(중요)
배포 전에 아래가 이미 완료되어 있어야 함
* ✔️ EC2 인스턴스 실행 중
* ✔️ Docker / Docker compose 설치 완료
* ✔️ 80, 443 포트 보안 그룹 오픈
* ✔️ 도메인(ppeum.com) 가비아에서 관리 중
<hr>

## 2️⃣ SSL 인증서 발급(Certbot + Docker)
DNS 인증 방식 (와일드카드/서브도메인 대응 가능)
서버에 Nginx/Apache 설치 없이 인증서만 발급

### 📌 EC2에서 실행
실습용으로 b-test.ppeum.com 도메인만 SSL 인증서를 발급하는 명령어
```
docker run -it --rm \
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/lib/letsencrypt:/var/lib/letsencrypt \
  certbot/certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "b-test.ppeum.com"
```
참고용
모든 도메인의 SSL을 발급하는 명령어
```
docker run -it --rm \
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/lib/letsencrypt:/var/lib/letsencrypt \
  certbot/certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "ppeum.com" \
  -d "*.ppeum.com" \
  -d "*.gangnam.ppeum.com" \
  -d "*.jeju.ppeum.com" \
  -d "*.myeongdong.ppeum.com" \
  -d "*.hongdae.ppeum.com" \
  -d "*.busan.ppeum.com"
```
<hr>

### 📌 실행 중 출력되는 내용
Certbot이 아래와 같은 메시지를 출력함
```
Please deploy a DNS TXT record under the name:
_acme-challenge.b-test.ppeum.com

with the following value:
xxxxxxxxxxxxxxxxxxxx
```
👉 처음 실행하는 경우 Email, Y, N을 입력하고 TXT 레코드를 발급 받으면 됨<br>
👉 이 값을 DNS에 TXT 레코드로 추가해야 함
<hr>

## 3️⃣ 가비아 DNS – TXT 레코드 등록 (인증용)

### 📍 가비아 경로

```
가비아
 → My 가비아
 → DNS 관리툴
 → ppeum.com 검색
 → [설정]
 → [레코드 수정]
 → 하단[레코드 추가]
```

### 📌 TXT 레코드 추가
| 항목 | 값
| :--- | :--- |
| **타입** | TXT
| **호스트** | _acme-challenge.b-test
| **값** | certbot에서 출력된 인증 값
| **TTL** | 기본값

⚠️ TXT 레코드는 인증 완료 후 삭제 가능
<hr>

### 📌 DNS 반영 확인 (선택이지만 권장)
```
nslookup -type=TXT _acme-challenge.b-test.ppeum.com
```
값이 보이면 → Certbot 화면으로 돌아가서 Enter
<hr>

## 4️⃣ 인증서 발급 완료 확인
인증 성공 시, EC2에 아래 경로 생성됨:
```
/etc/letsencrypt/live/b-test.ppeum.com/
  ├─ fullchain.pem
  └─ privkey.pem
```
👉 이 파일들을 Docker Nginx / Web 컨테이너에서 사용

실습용에서는 nginx/default.conf 파일을 아래와 같이 변경할 필요가 있음(특정 도메인의 SSL 인증서만 발급했기 때문)
```
# 발급받은 와일드카드 인증서 경로
ssl_certificate /etc/letsencrypt/live/b-test.ppeum.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/b-test.ppeum.com/privkey.pem;
```
<hr>

## 5️⃣ 도메인 연결 – 가비아 A 레코드 설정
이제 실제 트래픽이 EC2로 들어오도록 설정

### 📍 가비아 DNS 설정
| 항목 | 값
| :--- | :--- |
| **타입** | A
| **호스트** | b-test
| **값** | EC2 Public IP
| **TTL** | 기본값

⚠️ DNS 전파: 수 분 ~ 수십 분 소요 가능
<hr>

### 📌 연결 확인
```
nslookup b-test.ppeum.com
```
EC2 Public IP가 나오면 정상
<hr>

## 6️⃣ 🔥 DB / 웹 방화벽(Security Group) 설정 (중요)
Docker 실행 전에 반드시 설정<br>
이 단계가 빠지면 “서버는 살아있는데 접속 안 됨” 발생

### 📌 AWS EC2 보안 그룹 설정
### ① 웹 서비스 포트 (필수)
| 항목 | 값 | 포트 | 소스
| :--- | :--- | :--- | :--- |
| HTTP | TCP | 80 | 0.0.0.0/0
| HTTPS | TCP | 443 | 0.0.0.0/0
<hr>

### ② DB 포트 (필요한 경우만)
⚠️ 운영 환경에서는 전체 오픈 금지<br>
→ 사내 IP / Bastion Host만 허용
| 항목 | 값 | 포트 | 소스
| :--- | :--- | :--- | :--- |
| MySQL | TCP | 3306 | 사내 IP

DB 컨테이너를 외부에 직접 노출하지 않는 구조라면<br>
👉 보안 그룹에서 DB 포트 오픈 불필요
<hr>

## 7️⃣ Docker Compose 실행 (서비스 기동)
SSL 인증서 발급 및 DNS 연결 완료 후 실행
```
docker compose up -d --build
```
👉 -build : Dockerfile 변경사항 반영<br>
👉 d : 백그라운드 실행
<hr>

## 8️⃣ 컨테이너 상태 확인
### 📌 컨테이너 실행 상태
```
docker compose ps
```

### 📌 로그 확인 (웹 / 앱 컨테이너)
```
docker compose logs -f app
```
(필요 시 nginx, web 등 서비스명으로 변경)
<hr>

## 9️⃣ 서비스 접속 확인 (최종 검증)
### ✅ 브라우저 확인
```
https://b-test.ppeum.com
```
✔️ 확인 항목
* 🔒 HTTPS 적용 여부
* 인증서 오류 없는지
* 웹 페이지 / API 정상 응답
<hr>

## ✅ 최종 점검 체크리스트
❗ EC2 Instance Connect 접속 가능<br>
❗ Docker / Docker Compose 정상 설치<br>
❗ 컨테이너 정상 기동<br>
❗ 도메인 접속 정상<br>
❗ 서버 재부팅 후 서비스 정상 기동<br>
<hr>

## 🔐 EC2 접속 운영 정책
* 글로벌 쁨 서버는 PEM Key 기반 SSH 접속을 사용하지 않습니다.
* 모든 서버 접속은 **AWS EC2 Instance Connect**를 통해 수행합니다.
* 서버 접근 권한은 **IAM 권한 관리**로 통제합니다.



