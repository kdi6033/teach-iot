# 🌐 i2r IoT PLC AWS Cloud Server 구축 

🎯 과제 목표

Ubuntu 서버에서 Nginx + Mosquitto + MongoDB + Node.js 를 통합 설치하여
IoT Cloud 서버를 완성하고, HTTPS로 접속 가능한 환경을 구축한다.

📍 참조 자료

[교육지원사이트 : 경제과학진흥원](https://www.gbedu.or.kr/gbsa/education/course/view.do?deGrCode=DE_000002140&menuNo=400032)      
[ChatGPT Assistant :i2r IoT PLC & Sensor](https://chatgpt.com/g/g-68fa17b36d3c819192a564d30d299395-i2r-iot-plc-sensor)     
[과제 제출용 매뉴얼 (GitHub)](https://github.com/kdi6033/teach-iot/releases/tag/homework-v1.0)     

🧱 시스템 전체 구조
```
[React UI:443] ─▶ [Nginx] ─▶ [Node.js API:1804] ─▶ [MongoDB:27017]
                            └▶ [Mosquitto WebSocket:8080 → 8883]
[IoT Device:1883] ─────────▶ [Mosquitto MQTT Broker]
```

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

# ✅ 1️⃣ Nginx 설치 및 기본 웹서버 설정
```
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```
✅ 테스트
브라우저에서 http://서버IP 접속 → “Welcome to nginx!” 페이지 확인

--------------
# ✅ 2️⃣ DNS 설정

✅ nip.io 이용 (학생용)
DNS를 가지고 있지 않은 경우 다음을 사용하세요. 학생들은 이것을 사용하세요
[nip.io 사용](https://github.com/kdi6033/react#lets-encrypt-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EC%9E%90%EB%8F%99-%EB%B0%9C%EA%B8%89--https-%EC%84%A4%EC%A0%95%EC%9D%84-%EC%9C%84%ED%95%9C-nginx-%EA%B5%AC%EC%84%B1)    

✅ AWS Route 53 이용 (도메인 보유자)
AWS에서 DNS를 가지고 있는 경우는 이를 이용하세요
[AWS Rout 53 이용](https://github.com/kdi6033/react#dns-htttp-https-%EC%84%A4%EC%A0%95)     

도메인 동작을 확인하세요👇
```
http://서버IP.nip.io
````

-----------------
# ✅ 3️⃣ HTTPS 인증서 설정 (Certbot + Nginx)    

여기서는 AWS Rout 53 에서 test.i2r.link 로 발급해 이것으로 기술 하겠습니다. 서버IP.nip.io 와 같이 발급받으신 분들은 이것을 사용 하세요    

1) 설치
```
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

2) Nginx 서버 블록 생성
```
sudo nano /etc/nginx/sites-available/test.i2r.link.conf
```

아래 내용 입력:
```
server {
    listen 80;
    listen [::]:80;
    server_name test.i2r.link;

    root /var/www/html;
    index index.html;

    # SPA가 아니라면 /index.html 포워딩은 빼도 됩니다.
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

3️) 테스트 페이지 작성
```
sudo mkdir -p /var/www/html
echo "<h1>test.i2r.link OK</h1>" | sudo tee /var/www/html/index.html
```

4️) 사이트 활성화
```
sudo ln -s /etc/nginx/sites-available/test.i2r.link.conf /etc/nginx/sites-enabled/
```

5) Nginx 재시작
```
sudo nginx -t
sudo systemctl restart nginx
```

6) SSL 인증서 발급
```
sudo certbot --nginx -d test.i2r.link
```

입력 가이드
|항목|입력|
|---|---|
이메일 입력|kdi6033@gmail.com
약관 동의|Y
EFF 이메일 수신|N (선택)
리디렉션|2 (Redirect) ✅

7) 인증 자동 갱신 확인
```
sudo systemctl status certbot.timer
```

테스트 실행:
```
sudo certbot renew --dry-run
```

8) 접속 테스트

브라우저에서 아래 입력:
```
https://test.i2r.link
```

✅ 자물쇠(SSL) 표시
✅ 화면: test.i2r.link OK

---------------------------
# ✅ 4️⃣ Node.js 설치 및 API 서버 구축    
✅ 1) 설치
```
sudo apt install curl -y
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
node -v && npm -v
sudo npm install -g pm2
```

✅ 2) backend 디렉토리 생성
현재 디렉토리가 ubutu 임을 확인 후 여기에 backend 생성
```
pwd
mkdir ~/backend
```


✅ 3) db-server.js 파일 생성 및 내용 넣기    

nano 편집기로 열기
```
nano ~/backend/db-server.js
```
다음 내용으로 작성한다.
📄 sudo nano db-server.js
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

✅ 4) 실행
```
cd ~/backend
npm init -y
npm install express cors
node db-server.js
```
✅ 5) 테스트 (새 터미널에서)
```
curl -i http://localhost:1804/api/health
```

----------------------------
# ✅ 5️⃣ Mosquitto 설치 (MQTT 브로커)    
1) 모스키토 설치 및 자동실행
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

2) 재시작    
```
sudo systemctl restart mosquitto
```

3) 테스트
```
mosquitto_sub -h localhost -t test/topic
mosquitto_pub -h localhost -t test/topic -m "Hello MQTT"
```
다음 사이트에서 테스트 할 수 힜습니다.     

[mqtt 통신을 테스트는 다음 사이트를 이용하세요.](https://www.hivemq.com/demos/websocket-client/)

------------

# ✅ 6️⃣ MongoDB 설치  

다음 유튜브와 메누얼을 참조하여 mongoDB를 설치하세요    
유튜브 : https://www.youtube.com/watch?v=WJrOxAN7ZH0    
메뉴얼 : https://github.com/kdi6033/i2r/blob/main/txt/aws%20mongoDB%20install     

-------------------

# ✅ 7️⃣ Iot 서버 프로그램 AWS에 설치    

[IoT 서버 소스프로그램 다운로드-간단한 교육용](https://github.com/kdi6033/i2r-03/releases/tag/react-25-11-test-v1.0)    

상업용으로 사용하실 분들은 다음을 다운로드 해서 사용하세요    
[IoT 서버 소스프로그램 다운로드](https://github.com/kdi6033/react/releases/tag/react-nip-ip-v1.0)     
서버에는 backend (데이터베이스 프로그램) 와 frontend (UI 프로그램) 가 있습니다.    
✅backend 구축     
- 데이터베이스 프로그램(db-server.js)을 서버에 설치한 후에 PM2를 사용하여 자동으로 실행되게 설정한다.

[PM2 설정](https://github.com/kdi6033/react?tab=readme-ov-file#%EF%B8%8F-4%EB%8B%A8%EA%B3%84-backend-db-serverjs-%EB%B6%80%ED%8C%85%EC%8B%9C-%EC%9E%90%EB%8F%99%EC%8B%A4%ED%96%89-%EC%84%A4%EC%A0%95)     

[filezilla 사용하여 파일전송](https://github.com/kdi6033/react/blob/main/README.md#ec2-%EC%84%9C%EB%B2%84%EC%97%90-filezilla%EB%A1%9C-%EC%97%B0%EA%B2%B0%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)    

1) backend db-server.js 부팅시 자동실행 설정
backend 디레토리로 이동후
```
npm install
```
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
✅frontend 구축
react로 구성한 홈페이지 프로그램으로 "npm run build"로 build 를 만들고 AWS 서버의 html 디렉토리에 업로드 한다.    

Nginx 설정에서 root 경로를 /var/www/html 로 지정합니다.
```
sudo nginx -t && sudo systemctl reload nginx
```

1) 서비스 동작 확인
db-server.js 의 api 명령을 하나 실행해 봅니다. test.i2r.link 은 자신의 DNS를 입력하세요
예시:
```
curl http://127.0.0.1:1804/api/health
```
응답: 저장되어 있는 데이터가 출력된다.
```
{"ok":true,"pid":13738,"time":"2025-10-31T01:38:46.186Z"}
```

✅ 최종 서버 설정    

파일내용 수정     

📄 sudo nano /etc/nginx/sites-available/test.i2r.link.conf
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
Nginx 재시작
```
sudo nginx -t
sudo systemctl restart nginx
```

------------------------
# ✅8. 아두이노 실습
Additional Boards Manager URLs
```
https://arduino.esp8266.com/stable/package_esp8266com_index.json
https://dl.espressif.com/dl/package_esp32_index.json
```
연습1    
```
와이파이 연결 프로그램 만들어줘
ssid : 8F_academy    password: gbsa123@@
```
```
/*
 * i2r-03 Wi-Fi Connection Example
 * ---------------------------------------
 * 보드: ESP32 (i2r-03)
 * 기능: Wi-Fi 연결 후 IP 주소를 시리얼로 표시
 * 제작: 김동일 교수 i2r 플랫폼 (https://i2r.link)
 * GitHub: https://github.com/kdi6033/i2r-03
 */

#include <WiFi.h>  // ESP32 Wi-Fi 라이브러리

// 🔹 Wi-Fi 정보 입력
const char* ssid = "8F_academy";
const char* password = "gbsa123@@";

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("\n📡 i2r-03 Wi-Fi 연결 시도 중...");

  // Wi-Fi 연결 시도
  WiFi.begin(ssid, password);

  int attempt = 0;
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    attempt++;
    if (attempt > 30) {
      Serial.println("\n❌ 연결 실패! Wi-Fi 정보를 확인하세요.");
      return;
    }
  }

  Serial.println("\n✅ Wi-Fi 연결 성공!");
  Serial.print("📶 연결된 SSID: ");
  Serial.println(WiFi.SSID());
  Serial.print("🌐 IP 주소: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  // Wi-Fi가 유지되는지 확인
  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("⚠️ Wi-Fi 연결 끊김! 재연결 시도 중...");
    WiFi.reconnect();
  }
  delay(5000);
}
```

mqtt 접속
```
mqtt 접속프로그램 추가해줘
boker : test.i2r.link  port : 1883
intopic: i2r/kdi6933@gmail.com/in
outtopic: i2r/kdi6933@gmail.com/out
```
```
/*
 * i2r-03 Wi-Fi + MQTT Example (Modified for OUT2)
 * -----------------------------------------------
 * Board : ESP32 (i2r-03)
 * 기능 : Wi-Fi + MQTT 통신 및 OUT2 릴레이 제어
 * 수정 : 릴레이 제어 핀을 GPIO 25 → 26 으로 변경
 */

#include <WiFi.h>
#include <PubSubClient.h>

// -------------------------------------------------
// 🔹 Wi-Fi 정보
// -------------------------------------------------
const char* ssid = "8F_academy";
const char* password = "gbsa123@@";

// -------------------------------------------------
// 🔹 MQTT 서버 정보
// -------------------------------------------------
const char* mqtt_server = "test.i2r.link";
const int   mqtt_port   = 1883;
const char* inTopic     = "i2r/kdi6933@gmail.com/in";
const char* outTopic    = "i2r/kdi6933@gmail.com/out";

// -------------------------------------------------
WiFiClient espClient;
PubSubClient client(espClient);

// -------------------------------------------------
// 🔹 Wi-Fi 연결 함수
// -------------------------------------------------
void setup_wifi() {
  delay(10);
  Serial.println("\n📡 Wi-Fi 연결 중...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\n✅ Wi-Fi 연결 성공!");
  Serial.print("📶 IP 주소: ");
  Serial.println(WiFi.localIP());
}

// -------------------------------------------------
// 🔹 MQTT 메시지 수신 콜백
// -------------------------------------------------
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("\n📩 수신 토픽: ");
  Serial.println(topic);
  Serial.print("📦 메시지: ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  if (String(topic) == inTopic) {
    if ((char)payload[0] == '1') {
      digitalWrite(26, HIGH);   // ✅ OUT2 ON
      Serial.println("🔔 OUT2 ON");
    } else {
      digitalWrite(26, LOW);    // ✅ OUT2 OFF
      Serial.println("💤 OUT2 OFF");
    }
  }
}

// -------------------------------------------------
// 🔹 MQTT 서버 재연결
// -------------------------------------------------
void reconnect() {
  while (!client.connected()) {
    Serial.print("🔄 MQTT 연결 시도 중...");
    String clientId = "i2r-03-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("\n✅ MQTT 연결 성공!");
      client.subscribe(inTopic);
      client.publish(outTopic, "Hello from i2r-03 (OUT2)!");
    } else {
      Serial.print("❌ 실패, rc=");
      Serial.print(client.state());
      Serial.println(" → 5초 후 재시도");
      delay(5000);
    }
  }
}

// -------------------------------------------------
// 🔹 메인 설정
// -------------------------------------------------
void setup() {
  Serial.begin(115200);
  pinMode(26, OUTPUT);       // ✅ OUT2 릴레이 핀 선언
  digitalWrite(26, LOW);     // 초기 상태 OFF

  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

// -------------------------------------------------
// 🔹 메인 루프
// -------------------------------------------------
void loop() {
  if (!client.connected()) reconnect();
  client.loop();

  // 10초마다 상태 메시지 전송
  static unsigned long lastMsg = 0;
  if (millis() - lastMsg > 10000) {
    lastMsg = millis();
    client.publish(outTopic, "i2r-03 OUT2 is alive...");
    Serial.println("📤 상태 메시지 전송");
  }
}

```

온도 습도 측정
```
/*
 * i2r-03 Wi-Fi + MQTT + AHTX0 (I2C) Example
 * ---------------------------------------------
 * Board : ESP32 (i2r-03)
 * Author: 김동일 교수 i2r (https://i2r.link)
 * GitHub: https://github.com/kdi6033/i2r-03
 */

#include <WiFi.h>
#include <PubSubClient.h>
#include <Wire.h>
#include <Adafruit_AHTX0.h>

// -------------------------------------------------
// 🔹 Wi-Fi 정보
// -------------------------------------------------
const char* ssid = "8F_academy";
const char* password = "gbsa123@@";

// -------------------------------------------------
// 🔹 MQTT 서버 정보
// -------------------------------------------------
const char* mqtt_server = "test.i2r.link";
const int   mqtt_port   = 1883;
const char* inTopic     = "i2r/kdi6933@gmail.com/in";
const char* outTopic    = "i2r/kdi6933@gmail.com/out";

// -------------------------------------------------
// 🔹 핀 설정
// -------------------------------------------------
#define RELAY_OUT2 26
#define SDA_PIN    21
#define SCL_PIN    22

// -------------------------------------------------
// 🔹 객체 선언
// -------------------------------------------------
WiFiClient espClient;
PubSubClient client(espClient);
Adafruit_AHTX0 aht;

// -------------------------------------------------
// 🔹 Wi-Fi 연결
// -------------------------------------------------
void setup_wifi() {
  delay(10);
  Serial.println("\n📡 Wi-Fi 연결 중...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\n✅ Wi-Fi 연결 성공!");
  Serial.print("📶 IP 주소: ");
  Serial.println(WiFi.localIP());
}

// -------------------------------------------------
// 🔹 MQTT 메시지 수신 콜백
// -------------------------------------------------
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("\n📩 수신 토픽: ");
  Serial.println(topic);
  Serial.print("📦 메시지: ");
  for (int i = 0; i < length; i++) Serial.print((char)payload[i]);
  Serial.println();

  if (String(topic) == inTopic) {
    if ((char)payload[0] == '1') {
      digitalWrite(RELAY_OUT2, HIGH);
      Serial.println("🔔 OUT2 ON");
    } else {
      digitalWrite(RELAY_OUT2, LOW);
      Serial.println("💤 OUT2 OFF");
    }
  }
}

// -------------------------------------------------
// 🔹 MQTT 서버 재연결
// -------------------------------------------------
void reconnect() {
  while (!client.connected()) {
    Serial.print("🔄 MQTT 연결 시도 중...");
    String clientId = "i2r-03-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("\n✅ MQTT 연결 성공!");
      client.subscribe(inTopic);
      client.publish(outTopic, "i2r-03 AHTX0 Sensor Ready!");
    } else {
      Serial.print("❌ 실패, rc=");
      Serial.print(client.state());
      Serial.println(" → 5초 후 재시도");
      delay(5000);
    }
  }
}

// -------------------------------------------------
// 🔹 초기 설정
// -------------------------------------------------
void setup() {
  Serial.begin(115200);
  pinMode(RELAY_OUT2, OUTPUT);
  digitalWrite(RELAY_OUT2, LOW);

  Wire.begin(SDA_PIN, SCL_PIN);
  if (!aht.begin()) {
    Serial.println("⚠️ AHTX0 센서 감지 실패! (배선 또는 주소 확인)");
    while (1) delay(1000);
  } else {
    Serial.println("✅ AHTX0 센서 초기화 완료!");
  }

  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

// -------------------------------------------------
// 🔹 메인 루프
// -------------------------------------------------
void loop() {
  if (!client.connected()) reconnect();
  client.loop();

  static unsigned long lastMsg = 0;
  if (millis() - lastMsg > 5000) {
    lastMsg = millis();

    sensors_event_t humidity, temp;
    aht.getEvent(&humidity, &temp); // AHTX0 데이터 읽기

    float t = temp.temperature;
    float h = humidity.relative_humidity;

    if (isnan(t) || isnan(h)) {
      Serial.println("⚠️ AHTX0 센서 데이터 읽기 실패!");
      return;
    }

    char payload[100];
    snprintf(payload, sizeof(payload), "{\"temp\":%.2f,\"humi\":%.2f}", t, h);
    client.publish(outTopic, payload);

    Serial.print("📤 MQTT 전송: ");
    Serial.println(payload);
  }
}

```

# ✅8. 프로토콜 연습
프로토콜을 가지고 i2r-03 보드를 제어 합니다.    

1) 출력제어 setInfo (si)    
맥어드레스가 B0:A7:32:1D:AF:50 인 보드의 1번 포트 on off 제어
on
```
{ "c": "so", "m": "B0:A7:32:1D:AF:50", "n": 1, "v": 1 }
```
on
```
{ "c": "so", "m": "B0:A7:32:1D:AF:50", "n": 1, "v": 0 }
```

2) 보드정보 읽어오기 : getStatus (gs)    
```
{ "c": "gs", "m": "B0:A7:32:1D:AF:50" }
```

# ✅9. iot 핵심 연습   

🔹 3️⃣ Node.js + MongoDB 서버 구축

📂 /server/server.js
```
import express from 'express';
import mongoose from 'mongoose';
import mqtt from 'mqtt';

const app = express();
mongoose.connect('mongodb://localhost:27017/local', { useNewUrlParser: true });

const DataSchema = new mongoose.Schema({
  mac: String,
  temp: Number,
  humi: Number,
  ts: { type: Date, default: Date.now },
});
const Sensor = mongoose.model('Sensor', DataSchema);

const client = mqtt.connect('mqtt://broker.i2r.link');
client.on('connect', () => client.subscribe('i2r/sensor'));
client.on('message', (topic, msg) => {
  const data = JSON.parse(msg.toString());
  console.log('Data received:', data);
  new Sensor(data).save();
});

app.listen(5000, () => console.log('Server running at http://localhost:5000'));

```

🔹 4️⃣ React UI (실시간 모니터링)

📂 /src/components/TempDashboard.tsx
```
import React, { useEffect, useState } from 'react';
import mqtt from 'mqtt';
import { LineChart, Line, XAxis, YAxis, Tooltip, CartesianGrid } from 'recharts';

export default function TempDashboard() {
  const [data, setData] = useState<any[]>([]);
  const client = mqtt.connect('wss://broker.i2r.link:8083');

  useEffect(() => {
    client.on('connect', () => client.subscribe('i2r/sensor'));
    client.on('message', (_, msg) => {
      const payload = JSON.parse(msg.toString());
      setData(prev => [...prev.slice(-19), { time: new Date().toLocaleTimeString(), temp: payload.temp }]);
    });
  }, []);

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold">🌡️ 실시간 온도 그래프</h2>
      <LineChart width={600} height={300} data={data}>
        <CartesianGrid stroke="#ccc" />
        <XAxis dataKey="time" />
        <YAxis domain={[0, 50]} />
        <Tooltip />
        <Line type="monotone" dataKey="temp" stroke="#f97316" strokeWidth={2} />
      </LineChart>
    </div>
  );
}

```

------------------

# 📚 참고 리소스

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

