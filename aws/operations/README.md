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
```

