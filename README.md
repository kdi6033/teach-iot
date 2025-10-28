# teach-iot
[교육지원사이트 경제과학진흥원](https://www.gbedu.or.kr/gbsa/education/course/view.do?deGrCode=DE_000002140&menuNo=400032)      
chatGPT Open AI를 개설했습니다. 제가 프로그램한 정보가 모두 담겨 있으니 이 지식을 시작으로 바이브 코딩을 시작하세요.
[i2r IoT PLC & Sensor](https://chatgpt.com/g/g-68fa17b36d3c819192a564d30d299395-i2r-iot-plc-sensor)

과제 제출
다음 사이트를 참조해서 과제를 제출하세요    
https://github.com/kdi6033/teach-iot/releases/tag/homework-v1.0

🌐 i2r IoT Cloud Server 구축 매뉴얼

Nginx + Mosquitto + MongoDB + Node.js 통합 설치 가이드

🧩 개요

이 문서는 Ubuntu 서버에
다음 3가지 서비스를 동시에 설치하여
IoT 시스템을 통합 운용하는 방법을 설명합니다.
| 구성요소                  | 역할                               | 포트            |
| --------------------- | -------------------------------- | ------------- |
| **Nginx**             | React 웹 UI 및 HTTPS Reverse Proxy | 80, 443, 8883 |
| **Mosquitto**         | MQTT 브로커 (IoT 센서 통신)             | 1883, 8080    |
| **MongoDB + Node.js** | 데이터 저장 및 API 서버                  | 27000, 1804   |

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
```
sudo apt install mosquitto mosquitto-clients -y
sudo systemctl enable mosquitto
```

📄 /etc/mosquitto/mosquitto.conf
```
persistence true
persistence_location /var/lib/mosquitto/
log_dest file /var/log/mosquitto/mosquitto.log

allow_anonymous true
listener 1883
protocol mqtt

listener 8080
protocol websockets
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

