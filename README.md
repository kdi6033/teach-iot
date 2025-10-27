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
```
sudo apt install mongodb -y
sudo systemctl enable mongodb
sudo systemctl start mongodb
sudo systemctl status mongodb
```

⚙️ 3️⃣ Node.js + PM2 백엔드 구성
```
sudo apt install nodejs npm -y
sudo npm install -g pm2
```

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
✅ PM2 실행
```
pm2 start db-server.js --name db-server
pm2 save
pm2 startup
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

⚙️ 5️⃣ Nginx 설치 및 HTTPS 설정
```
sudo apt install nginx certbot python3-certbot-nginx -y
```

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

⚙️ 6️⃣ SSL 인증서 발급 (Let’s Encrypt)
```
sudo certbot certonly --nginx -d test.i2r.link
sudo systemctl status certbot.timer
```

✅ 자동 갱신 활성화
✅ /etc/letsencrypt/live/test.i2r.link/fullchain.pem 사용


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

Ubuntu 한 대의 서버에
Nginx (HTTPS/WebSocket Proxy),
Mosquitto (MQTT Broker),
MongoDB + Node.js (데이터베이스 + API)
를 통합하여 i2r IoT 시스템을 완성합니다.

