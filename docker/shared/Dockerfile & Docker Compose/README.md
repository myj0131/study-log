# 🐳 Docker 입문자를 위한 Dockerfile & 실행 가이드
이 문서는 도커를 처음 접하거나, 실무 경력은 있지만 도커 경험이 적은 입문자를 대상으로 작성되었습니다.<br>
목표는 다음 질문에 스스로 답할 수 있게 하는 것입니다.
* Dockerfile은 왜 필요한가?
* docker build / run은 정확히 무엇을 하는가?
* 코드 수정이 왜 바로 반영되지 않는가?
<hr>

## 1️⃣ Docker의 기본 개념 정리

### ✅ Dockerfile
* 이미지를 만들기 위한 설계도
* 어떤 OS, 어떤 프로그램, 어떤 파일을 포함할지 정의

### ✅ Image(이미지)
* Dockerfile을 기반으로 만들어진 실행 전 상태의 결과물
* 불변(immutable)

### ✅ Container(컨테이너)
* 이미지를 실제로 실행한 것
* 실행 중인 프로세스

```
Dockerfile → (docker build) → Image → (docker run) → Container
```

## 2️⃣ Dockerfile 최소 예제 (Hello Docker)
```
FROM alpine:latest
CMD ["echo","Hello Docker!"]
```
### 📍 설명
* **FROM:** 어떤 베이스 이미지를 사용할지 지정
*  **CMD:** 컨테이너 실행 시 실행할 명령

### 📍 실행
```
docker build -t hello-docker .
docker run hello-docker
```

### 📍 출력
```
Hello Docker!
```

## 3️⃣ docker build 명령어 설명
```
docker build -t hello-docker .
```
| 용어 | 쉬운 설명
| :--- | :--- |
| **docker** | Docker CLI 실행
| **build** | 이미지를 생성
| **-t** | 이미지 이름(tag) 지정
| **hello-docker** | 이미지 이름
| **.** | Dockerfile 위치(빌드 컨텍스트)



