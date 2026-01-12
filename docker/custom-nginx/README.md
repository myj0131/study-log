# 🏗️ 내 입맛대로 웹 서버 이미지 만들기

## 1️⃣ 작업 공간 만들기
컴퓨터 바탕화면이나 편한 곳에 my-docker-web이라는 폴더를 하나 만드세요.(ex: D:\test\docker\my-docker-web)

## 2️⃣ 나만의 웹 페이지 작성 (index.html)
그 폴더 안에 index.html이라는 이름의 파일을 만들고 아래 내용을 복사해서 저장하세요.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>나의 첫 도커 페이지</title>
</head>
<body>
    <h1>성공입니다! 🎉</h1>
    <p>이 웹 서버는 제가 직접 만든 Dockerfile로 구워냈습니다.</p>
</body>
</html>
```

## 3️⃣ 설계도 작성 (Dockerfile)
같은 폴더에 확장자 없이 이름이 정확히 **Dockerfile**인 파일을 만들고 아래 두 줄을 적으세요.

```Dockerfile
# 1. 베이스 이미지로 nginx를 사용하겠다
FROM nginx

# 2. 내 컴퓨터의 index.html을 컨테이너 안의 웹 서버 경로로 복사해라
COPY index.html /usr/share/nginx/html/index.html
```

# 🛠️ 나만의 이미지 굽기 (Build)
이제 터미널(CMD/PowerShell)을 열고 해당 폴더로 이동한 뒤, 아래 명령어를 입력하세요.

```Bash
docker build -t my-custom-web .
```

**주의:** 맨 마지막에 마침표( . )를 꼭 찍어야 합니다! "현재 폴더에 있는 Dockerfile을 사용해라"라는 뜻입니다.

# 🏃 직접 만든 이미지 실행하기
방금 만든 따끈따끈한 이미지를 실행해 봅시다. (아까 만든 my-webserver와 겹치지 않게 포트 번호와 이름을 살짝 바꿀게요.)

```Bash
docker run -d -p 9000:80 --name my-custom-server my-custom-web
```

### **결과 확인**
브라우저를 열고 localhost:9000에 접속해 보세요.

아까 적은 "성공입니다! 🎉" 메시지가 보이나요?

docker images를 입력해 보세요. my-custom-web이라는 이미지가 생성되어 있고, <br>
CREATED가 **'방금(seconds ago)'**으로 되어 있는 걸 확인하실 수 있습니다.
<hr>

# 📝 오늘 배운 핵심 개념 (GitHub 기록용)
**Build(빌드):** Dockerfile이라는 레시피를 가지고 실제 이미지를 만드는 과정입니다.

**FROM:** 설계도의 기초가 될 이미지를 고르는 명령어입니다.

**COPY:** 내 컴퓨터에 있는 파일을 도커 이미지 안으로 집어넣는 명령어입니다.

