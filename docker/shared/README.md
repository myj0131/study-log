# 과장님께 공유받은 Docker 입문 내용
<details>
<summary><h3>🙋 도커 설치 시 wsl을 설치하는 이유</h3></summary>
  
### 1️⃣ 커널(Kernel)공유의 문제
Docker 컨테이너는 가상머신(VM)과 달리 호스트 운영체제의 **커널(OS의 핵심 부품)** 을 공유해서 사용합니다.
* **Docker 컨테이너:** 리눅스 커널 기반으로 설계됨.
* **Windows:** 리눅스와 커널 구조가 완전히 다름.

따라서 Windows에서 리눅스용 컨테이너를 실행하려면, Windows 안에서 리눅스 커널을 가상화해서 돌려줄 **중간다리** 가 필요한데, <br>
그 역할을 **WSL 2** 가 수행합니다.

### 2️⃣ 성능과 속도(Hyper-V와의 차이)
과거에는 Hyper-V라는 무거운 가상화 기술을 사용했습니다. 하지만 WSL 2는 다음과 같은 압도적인 장점이 있습니다.
* **빠른 실행:** 가상머신을 통째로 띄우는 것보다 부팅 속도가 훨씬 빠릅니다.
* **리소스 효율:** 필요한 만큼만 RAM을 점유하며, 사용하지 않을 때는 자원을 반환합니다.
* **파일 시스템 성능:** 리눅스 파일 시스템과의 연동 속도가 이전 방식보다 수십 배 빠릅니다.

### 3️⃣ 완벽한 호환성
WSL 2는 마이크로소프트가 직접 리눅스 커널을 Windows에 내장시킨 형태입니다. 덕분에 개발 환경에서 다음과 같은 이점을 얻습니다.
* 실제 리눅스 서버와 거의 동일한 환경에서 Docker 명령어를 실행할 수 있습니다.
* localhost를 통해 Windows와 리눅스 컨테이너 간의 네트워크 통신이 매우 매끄럽게 이루어집니다.

</details>

# 🐳 Docker 시작하기: 설치부터 실무 예제까지
이 가이드는 Windows 환경에서 WSL 2를 활용하여 Docker Desktop을 설치하고, 웹 서버 PHP 개발 환경을 맛보는 예제를 담고 있습니다.

## 1️⃣ 사전준비: WSL 2 설치 및 리눅스 셋팅
Docker Desktop은 리눅스 커널을 사용하므로, Windows 환경에서는 WSL2 설치가 필수입니다.

### ✅ WSL 및 리눅스 배포판 설치
🔹 **STEP 1:** Windows 터미널(PowerShell)을 **관리자 권한** 으로 실행합니다. <br>
🔹 **STEP 2:** 설치 가능한 리눅스 배포판 목록을 확인합니다.<br>
```
wsl --list --online
```
🔹 **STEP 3:** 특정 배포판(예: Ubuntu 22.04)을 지정하여 설치합니다.<br>
```
wsl --install -d Ubuntu-22.04
```
### ✅ 우분투 초기 설정
🔹 **STEP 1:** 설치가 완료되면 리눅스 터미널 창이 자동으로 열리며 username / password 생성 화면이 나타납니다. <br>
* **Enter new UNIX username:** 사용할 아이디 입력<br>
* **New password:** 사용할 비밀번호 입력 (입력 시 화면에 글자가 보이지 않으니 주의)<br>

🔹 **STEP 2:** 설치 완료 후 반드시 **윈도우 재시작** 을 진행합니다.<br>

### ✅ 패키지 업데이트
설치된 리눅스를 최신 상태로 유지하기 위해 터미널에서 아래 명령어를 실행합니다.<br>
```
sudo apt update && sudo apt upgrade -y
```
<hr>

## 2️⃣ Docker Desktop 설치 및 설정

### ✅ 설치단계
🔹 **STEP 1:** Docker 공식 홈페이지에서 설치 파일을 다운로드합니다. <br>
🔹 **STEP 2:** 설치 중 "Use WSL 2 instead of Hyper-V" 옵션을 반드시 체크합니다. <br>

### ✅ 필수 설정(매우 중요)
🔹 **STEP 1:** WSL 엔진 활성화: Settings > General > Use the WSL 2 based engine 체크 확인 <br>
<img width="1263" height="713" alt="image" src="https://github.com/user-attachments/assets/30f6663a-eb31-44b9-8884-08a2c40f71f4" />

🔹 **STEP 2:** 배포판 연동: Settings > Resources > WSL Integration 에서 설치한 Ubuntu-22.04 스위치를 ON으로 변경합니다. <br>
* 이 설정을 해야 우분투 터미널에서 docker 명령어를 쓸 수 있습니다.
<img width="1261" height="716" alt="image" src="https://github.com/user-attachments/assets/e222e801-5428-42ac-ab61-0f06e35238c8" />

## 3️⃣ 웹 서버 실행해보기 (Nginx & Apache)

### ✅ Nginx 실행 (포트 8080)
```
docker run -d -p 8080:80 --name my-nginx nginx
```

### ✅ Apache(httpd) 실행 (포트 8081)
```
docker run -d -p 8081:80 --name my-httpd httpd
```

## 4️⃣ PHP 8.2 환경 구축 및 phpinfo() 출력

### ✅ PHP 8.2 컨테이너 실행
```
docker run -d -p 8000:80 --name my-php-app php:8.2-apache
```

### ✅ phpinfo() 파일 생성하기
```
docker exec -it my-php-app bash -c "echo '<?php phpinfo(); ?>' > index.php"
```

### ✅ 브라우저에서 확인하기
```
localhost:8000
```

## 5️⃣ 컨테이너 내부 파일 수정하기 (Vim 사용법 등)
공식 이미지에는 보통 vim이 없습니다. 다음 방법 중 하나를 선택하세요.

### ✅ 컨테이너 안에 Vim 설치하기 (일시적)
```
docker exec -it my-nginx bash
apt update && apt install vim -y
```

### ✅ 호스트에서 파일을 수정하여 복사하기
```
docker cp index.html my-nginx:/usr/share/nginx/html/index.html
```

### ✅ 볼륨 바인딩 (추천 방식 ⭐)
```
docker run -d -p 8080:80 -v $PWD:/usr/share/nginx/html nginx
```

## 6️⃣ 자주 쓰는 핵심 명령어 (Cheatsheet)
### ✅ WSL 관련 명령어 (PowerShell에서 실행)

| 명령어 | 설명
| :--- | :--- |
| **wsl -d [이름]** | 특정 리눅스 배포판 실행 (예: wsl -d Ubuntu-22.04)
| **wsl -t [이름]** | 실행 중인 리눅스 배포판 강제 종료 (Terminate)
| **wsl --shutdown** | 모든 WSL 배포판 및 도커 엔진 전체 종료
| **wsl -l -v** | 설치된 리눅스 목록 및 상태 확인

<hr>

### ✅ Docker 관련 명령어
#### 📦 컨테이너 생성 / 실행 / 관리
| 목적 | 명령어 | 설명
| :--- | :--- | :--- |
| **컨테이너 생성 + 실행** | docker run IMAGE | 이미지로 컨테이너 생성 후 실행
| **이름 지정해서 생성** | docker run --name my-container IMAGE | 컨테이너 이름 지정
| **백그라운드 실행** | docker run -d IMAGE | detached 모드
| **이름 + 백그라운드** | docker run -d --name my-container IMAGE | 실무에서 가장 흔함
| **포트 매핑** | docker run -p 8080:80 IMAGE | 호스트↔컨테이너 포트 연결
| **이름 + 포트 매핑** | docker run -d --name web -p 80:80 nginx | 웹 서버 기본 패턴

<hr>

### ✅ ▶️ 시작 / ⏸ 중지 / 🔁 재시작

| 목적 | 명령어 | 설명
| :--- | :--- | :--- |
| **컨테이너 시작** | docker start CONTAINER | 중지된 컨테이너 실행
| **컨테이너 중지** | docker stop CONTAINER | 정상 종료(SIGTERM)
| **강제 종료** | docker kill CONTAINER | 즉시 종료(SIGKILL)
| **재시작** | docker restart CONTAINER | stop → start

<hr>

### ✅ 🗑 삭제

| 목적 | 명령어 | 설명
| :--- | :--- | :--- |
| **컨테이너 삭제** | docker rm CONTAINER | 중지된 컨테이너만 가능
| **강제 삭제** | docker rm -f CONTAINER | 실행 중이어도 삭제
| **이미지 삭제** | docker rmi IMAGE | 이미지 제거

<hr>

### ✅ 🔍 로그 / 접속

| 목적 | 명령어 | 설명
| :--- | :--- | :--- |
| **로그 보기** | docker logs CONTAINER | 표준 출력 로그
| **실시간 로그** | docker logs -f CONTAINER | tail -f
| **컨테이너 접속** | docker exec -it CONTAINER sh | 셸 접속
| **bash 접속** | docker exec -it CONTAINER bash | bash 있을 때
