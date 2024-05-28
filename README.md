# Replication Controller를 이용하여 서비스 배포하기

#OS : rocky 8.8
#Docker version :  26.1.3, build b72abbb
#node.js : v20.12.2

# Nodejs 웹 어플리케이션 구성

server.js라는 이름으로 아래 코드를 작성한다.
===================================================================
/home/sjh/docker_k8s/server.js
var os = require('os');
var http = require('http');

var handleRequest = function(request, response) {
  response.writeHead(200);
  response.end("Hello World! I'm " + os.hostname());
  // log
  console.log("[" + Date(Date.now()).toLocaleString() + "] " + os.hostname());
}

var www = http.createServer(handleRequest);
www.listen(8080);


이 코드는 8080 포트로 웹서버를 띄워서 접속하면 “Hello World!” 문자열과 함께, 서버의 호스트명을 출력해준다. 그리고 stdout에 로그로, 시간과 서버의 호스트명을 출력해준다.
그리고 systemd에 등록한다.

vi /etc/systemd/system/nodejs-app.service

[Unit]
Description=Node.js server
After=network.target

[Service]
ExecStart=/bin/node /home/sjh/docker_k8s/server.js
Restart=always
User=root
Group=root
Environment=PATH=/usr/bin:/usr/local/bin:/bin
Environment=NODE_ENV=production
WorkingDirectory=/home/sjh/docker_k8s

[Install]
WantedBy=multi-user.target


등록후 데몬 재실행 및 server.js를 실행 및 enable한다.
===================================================================
systemctl daemon-reload
systemctl start nodejs-app.service
systemctl enable nodejs-app.service



도커로 패키징하기
===================================================================
node.js 애플리케이션을 도커 컨테이너로 패키징 한다.
Dockerfile 이라는 파일을 만들고 아래 코드를 작성한다.

# Base image
FROM node:carbon

# Set working directory
WORKDIR /usr/src/app

# Copy server.js to the working directory
COPY server.js .

# Expose port 8080
EXPOSE 8080

# Run the application
CMD ["sh", "-c", "node server.js > log.out"]


docker build 명령어로 이미지를 빌드 후 실행한다.
===================================================================
docker build -t my-node-app:v1 .
docker run -d -p 8080:8080 my-node-app:v1



IP주소:8080 으로 접속 했을 시 호스트명이 달라지는걸 확인할 수 있다.
===================================================================


