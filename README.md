# 🌐 i2r IoT PLC AWS Cloud Server 구축 

🎯 과제 목표

Ubuntu 서버에서 Nginx + Mosquitto + MongoDB + Node.js 를 통합 설치하여
IoT Cloud 서버를 완성하고, HTTPS로 접속 가능한 환경을 구축한다.

📍 참조 자료

[교육지원사이트 : 경제과학진흥원](https://www.gbedu.or.kr/gbsa/education/course/view.do?deGrCode=DE_000002140&menuNo=400032)      
[ChatGPT Assistant :i2r IoT PLC & Sensor](https://chatgpt.com/g/g-68fa17b36d3c819192a564d30d299395-i2r-iot-plc-sensor)     
[과제 제출용 매뉴얼 (GitHub)](https://github.com/kdi6033/teach-iot/releases/tag/homework-v1.0)     

-------------------
⚙️ 설치 순서

| 순서  | 항목                        | 주요 포트         | 설명                             |
| --- | ------------------------- | ------------- | ------------------------------ |
| 1️⃣ | **Nginx**                 | 80, 443, 8883 | React 웹UI, HTTPS Reverse Proxy |
| 2️⃣ | **DNS (nip.io)**          | -             | AWS 공인 IP 기반 도메인 자동 생성         |
| 3️⃣ | **인증서 설치 및 HTTPS 설정**     | -             | Certbot + Let's Encrypt        |
| 4️⃣ | **Node.js 설치**            | 1804          | 백엔드 API 서버                     |
| 5️⃣ | **Mosquitto 설치**          | 1883, 8080    | MQTT 브로커 (센서 통신)               |
| 6️⃣ | **MongoDB 설치**            | 27017         | 데이터 저장소                        |
| 7️⃣ | **FileZilla 설치 및 파일 업로드** | 22 (SFTP)     | backend / frontend 업로드 및 설치    |

-----------------

⚙️ 1️⃣ Nginx 설치 및 기본 웹서버 설정
```
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```
✅ 테스트
브라우저에서 http://서버IP 접속 → “Welcome to nginx!” 페이지 확인

--------------
⚙️ 2️⃣ DNS 설정

DNS를 가지고 있지 않은 경우 다음을 사용하세요. 학생들은 이것을 사용하세요
[nip.io 사용](https://github.com/kdi6033/react#lets-encrypt-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EC%9E%90%EB%8F%99-%EB%B0%9C%EA%B8%89--https-%EC%84%A4%EC%A0%95%EC%9D%84-%EC%9C%84%ED%95%9C-nginx-%EA%B5%AC%EC%84%B1)    

AWS에서 DNS를 가지고 있는 경우는 이를 이용하세요
[AWS Rout 53 이용](https://github.com/kdi6033/react#dns-htttp-https-%EC%84%A4%EC%A0%95)     

AWS 서버 IP가 18.212.214.14라면
도메인은 다음처럼 사용합니다 도메인 동작을 확인하세요👇
```
http://18.212.214.14.nip.io
````

-----------------
⚙️ 3️⃣ HTTPS 인증서 설정 (Certbot 설치)    

여기서는 AWS Rout 53 에서 test9.i2r.link 로 발급해 이것으로 기술 하겠습니다. 18.212.214.14.nip.io 와 같이 발급받으신 분들은 이것을 사용 하세요
```
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

2️⃣ Nginx 서버 블록 생성
```
sudo nano /etc/nginx/sites-available/test9.i2r.link
```

아래 내용 입력:
```
server {
    listen 80;
    listen [::]:80;
    server_name test9.i2r.link;

    root /var/www/test9;
    index index.html;

    # SPA가 아니라면 /index.html 포워딩은 빼도 됩니다.
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

3️⃣ 웹 폴더 생성 및 테스트 페이지 작성
```
sudo mkdir -p /var/www/test9
echo "<h1>test9.i2r.link OK</h1>" | sudo tee /var/www/test9/index.html
```

4️⃣ 사이트 활성화 (심볼릭 링크 생성)
```
sudo ln -s /etc/nginx/sites-available/test9.i2r.link /etc/nginx/sites-enabled/
```

5️⃣ Nginx 설정 테스트 & 재시작
```
sudo nginx -t
sudo systemctl restart nginx
```

6️⃣ HTTPS 인증서 발급 (Let's Encrypt)
```
sudo certbot --nginx -d test9.i2r.link
```

입력 가이드
|항목|입력|
|---|---|
이메일 입력|kdi6033@gmail.com
약관 동의|Y
EFF 이메일 수신|N (선택)
리디렉션|2 (Redirect) ✅

7️⃣ 인증 자동 갱신 확인
```
sudo systemctl status certbot.timer
```

테스트 실행:
```
sudo certbot renew --dry-run
```

8️⃣ 접속 테스트

브라우저에서 아래 입력:
```
https://test9.i2r.link
```

✅ 자물쇠(SSL) 표시
✅ 화면: test8.i2r.link OK

---------------------------
🧩 3️⃣ Node.js 설치 및 API 서버 구축    
```
sudo apt install curl -y
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
node -v && npm -v
sudo npm install -g pm2
```

✅ 1) backend 디렉토리 생성
현재 디렉토리가 ubutu 임을 확인 후 여기에 backend 생성
```
pwd
mkdir ~/backend
```


✅ 2) db-server.js 파일 생성 및 내용 넣기    

nano 편집기로 열기
```
nano ~/backend/db-server.js
```
다음 내용으로 작성한다.
📄 db-server.js
```
const express = require('express');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

const PORT = 1804;

// 헬스 체크만 제공
app.get('/api/health', (_, res) => {
  res.json({ ok: true, time: new Date().toISOString() });
});

app.listen(PORT, () => {
  console.log(`✅ Server running on port ${PORT}`);
});
```

📦 실행
```
cd ~/backend
npm init -y
npm install express cors
node db-server.js
```
✅ 테스트 (새 터미널에서)
```
curl -i http://localhost:1804/api/health
```

----------------------------
📡 4️⃣ Mosquitto 설치 (MQTT 브로커)    
모스키토 설치 및 자동실행
```
sudo apt install mosquitto mosquitto-clients -y
sudo systemctl enable mosquitto
sudo systemctl restart mosquitto
``` 
📄 sudo nano /etc/mosquitto/mosquitto.conf
```
persistence true
allow_anonymous true
listener 1883
protocol mqtt

listener 8080
protocol websockets

include_dir /etc/mosquitto/conf.d
```

적용
```
sudo systemctl restart mosquitto
```

✅ 테스트
```
mosquitto_sub -h localhost -t test/topic &
mosquitto_pub -h localhost -t test/topic -m "Hello MQTT"
```

🗃️ 5️⃣ MongoDB 설치
sudo apt update
sudo apt install gnupg curl -y
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl enable mongod
sudo systemctl start mongod

✅ 확인
mongo --eval 'db.runCommand({ connectionStatus: 1 })'


🧱 전체 연동 구조
[Nginx:443] → [Node.js:1804] → [MongoDB:27017]
                    └→ [Mosquitto:8080 → 8883]
[IoT Device:1883] → [Mosquitto Broker]


✅ 추천 순서의 장점
장점설명즉시 결과 확인Nginx 설치 후 바로 화면 출력 가능SSL 우선 확보인증서 문제를 초기에 해결 가능API → MQTT → DB 흐름통신 → 데이터 저장 순으로 자연스럽게 연결교육 효율성각 단계가 시각적으로 확인 가능 (웹/터미널)

원하신다면 위 내용을 학생용 실습 문서 (PDF) 로 자동 변환해드릴 수도 있습니다.
➡️ “PDF로 만들어줘” 라고 하면 바로 생성해드리겠습니다.







-------
⚙️ 1️⃣ 기본 환경 설정
```
sudo apt update && sudo apt upgrade -y
sudo apt install git curl vim ufw net-tools -y
```
⚙️ 2️⃣ MongoDB 설치 및 실행
다음 유튜브와 메누얼을 참조하여 mongoDB를 설치하세요    
유튜브 : https://www.youtube.com/watch?v=WJrOxAN7ZH0    
메뉴얼 : https://github.com/kdi6033/i2r/blob/main/txt/aws%20mongoDB%20install     

⚙️ 3️⃣ Node.js 설치   

✅ 1. Node.js 설치 (최신 LTS 버전)    

Node.js는 공식 설치 스크립트를 통해 설치하는 것이 가장 안전합니다.

🔹 터미널 명령어:
```
# 필수 도구 설치
sudo apt update
sudo apt install curl -y

# NodeSource 저장소 등록 (LTS 최신)
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

# Node.js 설치
sudo apt install -y nodejs
```
🔸 설치 확인
```
node -v     # 예: v18.x.x 또는 v20.x.x
npm -v      # 예: 9.x.x
```

✅ 2. TypeScript 전역 설치
TypeScript는 npm으로 설치합니다.
```
sudo npm install -g typescript
```
🔸 설치 확인
```
tsc -v      # 예: Version 5.x.x
```
📦 선택: ts-node, nodemon도 함께 설치하면 자동 재시작하여 편리
```
sudo npm install -g ts-node nodemon
```
ts-node: .ts 파일을 바로 실행
nodemon: 자동 리로드 (서버 개발 시 유용)

📁 프로젝트 폴더 생성
```
mkdir ~/backend && cd ~/backend

```

📄 db-server.js
```
const express = require('express');
const { MongoClient } = require('mongodb');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

const PORT = process.env.PORT || 1804;
const MONGO_URL = process.env.DATABASE_URL || 'mongodb://127.0.0.1:27017';
const DB_NAME = 'local';
const COL_RECORD = 'localRecord';

// 기본 헬스 체크
app.get('/api/health', (_, res) => res.json({ ok: true, time: new Date().toISOString() }));

app.listen(PORT, () => console.log(`✅ Server running on http://localhost:${PORT}`));
```
✅ backend db-server.js 부팅시 자동실행 설정
컴퓨터가 부팅시 자동으로 실행 하도롤 설정한다.
```
sudo npm install -g pm2
sudo pm2 start db-server.js --name db-server
sudo pm2 save
sudo pm2 startup
```
프로그램 수정을 할 경우는 다음과 같이 다시 실행한다.
```
sudo pm2 restart db-server
```
pm2 실행 중인것을 보려면
```
sudo pm2 list
```

⚙️ 4️⃣ Mosquitto (MQTT 브로커) 설치
설치 및 자동실행 설정
```
sudo apt install mosquitto mosquitto-clients -y
sudo systemctl enable mosquitto
sudo systemctl restart mosquitto
```

📄 /etc/mosquitto/mosquitto.conf    

sudo nano /etc/mosquitto/mosquitto.conf
```
persistence true
allow_anonymous true

listener 1883
protocol mqtt

listener 8080
protocol websockets

include_dir /etc/mosquitto/conf.d
```

✅ 기본 포트

1883 → IoT 디바이스 (MQTT TCP)    
8080 → WebSocket (Nginx에서 WSS로 프록시됨)

⚙️ 5️⃣ Nginx 설치 
```
sudo apt update
sudo apt install nginx -y
```
홈페이지 수정 후에는 
Nginx 서비스 재시작
```
sudo systemctl restart nginx
```
✅ 확인
브라우저에서 http://your-ec2-ip 접속 시 React 웹 앱이 보이면 성공입니다. 처음 공부하는 분들은 여기까지 해서 홈페이지를 접속하시고 다음 과정은 나중에 진행 하세요

⚙️ 5️⃣ HTTPS 설정, 인증서 설치    

✅ Certbot 설치 (Nginx용)
```
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```
✅ 점검 포인트 (필수)
sites-enabled 링크/중복 설정 정리
```
# default 끄기(충돌 방지)
sudo rm -f /etc/nginx/sites-enabled/default

# 현재 conf가 링크되어 있는지 확인
ls -l /etc/nginx/sites-enabled/
```

✅ Nginx 서버 설정에 server_name 명확히 설정되었는지 확인
```
sudo nano /etc/nginx/sites-available/test.i2r.link.conf
```
Nginx 다시 시작
```
sudo nginx -t
sudo systemctl restart nginx
```

✅ HTTPS 인증서 발급 및 자동 설정
```
sudo certbot --nginx -d test.i2r.link 
```
🚀 진행 중 아래와 같은 질문에 다음처럼 대답하세요:

이메일 입력 → 본인 이메일 입력

약관 동의 → Yes

마케팅 메일 수신 → No

HTTP → HTTPS 리디렉션 → 2번 (Redirect) 선택 권장

✅ 자동 갱신 설정 확인
Let's Encrypt 인증서는 90일짜리입니다. 자동 갱신을 위해 crontab 등록 상태 확인:
```
sudo systemctl status certbot.timer
```
보통 설치 시 자동 등록되어 있으며, 없다면 수동으로 추가해도 됩니다:
```
sudo crontab -e
```
맨 아래에 추가:
```
0 3 * * * certbot renew --quiet
```
✅ 성공 시 확인

설치가 완료되면 /etc/letsencrypt/live/test.i2r.link/ 폴더가 생깁니다:
```
sudo ls -l /etc/letsencrypt/live/test.i2r.link/
```

파일 예시:
```
cert.pem
chain.pem
fullchain.pem
privkey.pem
```
Nginx 설정도 자동으로 아래처럼 추가됩니다
/etc/nginx/sites-available/test.i2r.link.conf
```
    ssl_certificate /etc/letsencrypt/live/test.i2r.link/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/test.i2r.link/privkey.pem;
```

⚙️ 5️⃣ HTTPS 설정
📄 /etc/nginx/sites-available/test.i2r.link.conf
```
# ① MQTT WebSocket Secure Proxy
server {
    listen 8883 ssl;
    server_name test.i2r.link;

    ssl_certificate     /etc/letsencrypt/live/test.i2r.link/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/test.i2r.link/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
    }
}

# ② HTTP → HTTPS 리디렉션
server {
    listen 80;
    server_name test.i2r.link;
    return 308 https://$host$request_uri;
}

# ③ HTTPS (React UI + Node.js API)
server {
    listen 443 ssl;
    server_name test.i2r.link;

    ssl_certificate     /etc/letsencrypt/live/test.i2r.link/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/test.i2r.link/privkey.pem;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:1804;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

✅ Nginx 활성화
```
sudo ln -s /etc/nginx/sites-available/test.i2r.link.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

🧪 8️⃣ 테스트
🧠 MQTT (내부)
```
mosquitto_sub -h localhost -p 1883 -t test/topic
mosquitto_pub -h localhost -p 1883 -t test/topic -m "Hello MQTT"
```

🌐 HTTPS

브라우저에서 접속 → https://test.i2r.link
자물쇠 🔒 표시 확인.

🧩 Node.js API
```
curl -i https://test.i2r.link/api/health
```


⚙️ 8️⃣ Iot 서버 프로그램 AWS에 설치    
서버에는 backend (데이터베이스 프로그램) 와 frontend (UI 프로그램) 가 있습니다.    
✅backend : db-server.js 를 구동하고 시스템이 동작하면 자동으로 실행되게 한다.    
- 데이터베이스 프로그램을 서버에 설치한 후에 PM2를 사용하여 자동으로 실행되게 설정한다.

[PM2 설정](https://github.com/kdi6033/react?tab=readme-ov-file#%EF%B8%8F-4%EB%8B%A8%EA%B3%84-backend-db-serverjs-%EB%B6%80%ED%8C%85%EC%8B%9C-%EC%9E%90%EB%8F%99%EC%8B%A4%ED%96%89-%EC%84%A4%EC%A0%95)     

[filezilla 사용하여 파일전송](https://github.com/kdi6033/react/blob/main/README.md#ec2-%EC%84%9C%EB%B2%84%EC%97%90-filezilla%EB%A1%9C-%EC%97%B0%EA%B2%B0%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)    

✅frontend : react로 구성한 홈페이지 프로그램으로 "npm run build"로 build 를 만들고 AWS 서버의 html 디렉토리에 업로드 한다.    
[IoT 서버 소스프로그램 다운로드-간단한 교육용](https://github.com/kdi6033/i2r-03/releases/tag/react-25-11-test-v1.0)    

[IoT 서버 소스프로그램 다운로드](https://github.com/kdi6033/react/releases/tag/react-nip-ip-v1.0)     

✅ 4. 서비스 동작 확인
db-server.js 의 api 명령을 하나 실행해 봅니다. test.i2r.link 은 자신의 DNS를 입력하세요
예시:
```
curl -s -X POST https://test.i2r.link/api/records -k
```
응답: 저장되어 있는 데이터가 출력된다.
```
[{"_id":"6823eef0dec9a7b8b45ce2de","name":"a","temp":"27"},{"_id":"6823ef5cdec9a7b8b45ce2df","name":"a","temp":"27"}]
```

🧱 시스템 전체 구조
```
[React UI:443] ─▶ [Nginx] ─▶ [Node.js API:1804] ─▶ [MongoDB:27017]
                            └▶ [Mosquitto WebSocket:8080 → 8883]
[IoT Device:1883] ─────────▶ [Mosquitto MQTT Broker]
```

✅ 최종 점검 리스트
| 항목          | 포트            | 상태 | 설명                      |
| ----------- | ------------- | -- | ----------------------- |
| Nginx       | 80, 443, 8883 | ✅  | 웹서버 + HTTPS 프록시         |
| Node.js API | 1804          | ✅  | MongoDB 백엔드             |
| Mosquitto   | 1883, 8080    | ✅  | MQTT + WebSocket        |
| MongoDB     | 27017         | ✅  | 센서 데이터 저장               |
| SSL 인증서     | Let’s Encrypt | ✅  | 자동 갱신 certbot.timer 활성화 |

📚 참고 리소스

🔗 i2r 공식 GitHub: https://github.com/kdi6033

🎥 김동일 교수 공식 YouTube: https://www.youtube.com/@i2r-link

🌐 공식 사이트: https://i2r.link

🎯 요약

하나의 Ubuntu 서버에서
Nginx (웹서버 & HTTPS 프록시),
Mosquitto (MQTT Broker),
MongoDB + Node.js (데이터 저장 & API)
를 통합하여 IoT 센서 데이터를 수집·시각화·제어하는
완전한 i2r Cloud 환경을 구축할 수 있습니다. 🚀

