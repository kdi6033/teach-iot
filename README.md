# teach-iot
[교육지원사이트 경제과학진흥원](https://www.gbedu.or.kr/gbsa/education/course/view.do?deGrCode=DE_000002140&menuNo=400032)    

[ChatGPT-1]    
```
esp32 wroom 을 이용해서 온도와 습도를 측정하는 프로그램 만들어줘 온습도 센서는 aht21 을 사용했습니다.
```
[생성된 프로그램]    
```
#include <Wire.h>
#include <Adafruit_AHTX0.h>

Adafruit_AHTX0 aht;

void setup() {
  Serial.begin(115200);
  
  // AHT21 센서 초기화
  if (!aht.begin()) {
    Serial.println("AHT21 센서를 찾을 수 없습니다. 연결을 확인하세요!");
    while (1) delay(10);
  }
  Serial.println("AHT21 센서 초기화 완료.");
}

void loop() {
  sensors_event_t humidity, temp;
  
  // 온습도 데이터 읽기
  aht.getEvent(&humidity, &temp);
  
  // 출력
  Serial.print("온도: ");
  Serial.print(temp.temperature);
  Serial.println(" °C");

  Serial.print("습도: ");
  Serial.print(humidity.relative_humidity);
  Serial.println(" %");

  delay(2000);  // 2초마다 측정
}
```
보드 입출력 기초 입력 정보
```
#include <Wire.h>

// Output pin numbers
const int outputPins[4] = {26, 27, 32, 33};
// Input pin numbers
const int inputPins[4] = {16, 17, 18, 19};

void setup() {
  // Initialize Serial for debugging
  Serial.begin(115200);

  // Set each output pin as an output
  for (int i = 0; i < 4; i++) {
    pinMode(outputPins[i], OUTPUT);
  }

  // Set each input pin as an input
  for (int i = 0; i < 4; i++) {
    pinMode(inputPins[i], INPUT);
  }
}
void loop() {
}
```
[ChatGPT]    
```
// Output pin numbers
const int outputPins[4] = {26, 27, 32, 33};
// Input pin numbers
const int inputPins[4] = {16, 17, 18, 19};
esp32 wroom 26번 핀이 1초 간격으로 점등되게 프로그램 해줘
```
[생성된 프로그램]  
```
// Output pin numbers
const int outputPins[4] = {26, 27, 32, 33};
// Input pin numbers
const int inputPins[4] = {16, 17, 18, 19};

void setup() {
  // 26번 핀을 출력으로 설정
  pinMode(outputPins[0], OUTPUT);
}

void loop() {
  // 26번 핀을 HIGH로 설정하여 LED를 켠다
  digitalWrite(outputPins[0], HIGH);
  delay(1000);  // 1초 대기
  
  // 26번 핀을 LOW로 설정하여 LED를 끈다
  digitalWrite(outputPins[0], LOW);
  delay(1000);  // 1초 대기
}
```
[ChatGPT] 
```
// Output pin numbers
const int outputPins[4] = {26, 27, 32, 33};
// Input pin numbers
const int inputPins[4] = {16, 17, 18, 19};
esp32 wroom 16번 핀의 입력에 따라 26번으로 출력이 나가게 프로그램 해줘
```
[생성된 프로그램]  
```
// Output pin numbers
const int outputPins[4] = {26, 27, 32, 33};
// Input pin numbers
const int inputPins[4] = {16, 17, 18, 19};

void setup() {
  // 26번 핀을 출력으로 설정
  pinMode(outputPins[0], OUTPUT);
  
  // 16번 핀을 입력으로 설정
  pinMode(inputPins[0], INPUT);
}

void loop() {
  // 16번 핀의 입력 상태를 읽음
  int inputState = digitalRead(inputPins[0]);

  // 입력 상태에 따라 26번 핀에 출력
  if (inputState == HIGH) {
    digitalWrite(outputPins[0], HIGH);  // 16번 핀이 HIGH이면 26번 핀을 켬
  } else {
    digitalWrite(outputPins[0], LOW);   // 16번 핀이 LOW이면 26번 핀을 끔
  }

  delay(10);  // 짧은 대기 시간으로 입력 상태 지속적으로 확인
}
```
[ChatGPT] 
```
esp32 wroom 와이파이 연결해줘 ssid=academy5 pw는 없음 학국시간 1초 간격으로 표시해줘
```
[생성된 프로그램]  
```
#include <WiFi.h>
#include <time.h>

// Wi-Fi 정보
const char* ssid = "academy5";  // Wi-Fi SSID
const char* password = "";      // 비밀번호 없음

// NTP 서버 설정
const char* ntpServer = "pool.ntp.org";
const long gmtOffset_sec = 9 * 3600;  // 한국 시간대 (GMT +9)
const int daylightOffset_sec = 0;     // 일광 절약 시간 없음

void setup() {
  // 시리얼 모니터 시작
  Serial.begin(115200);

  // Wi-Fi 연결 시도
  Serial.printf("WiFi에 연결 중: %s\n", ssid);
  WiFi.begin(ssid, password);

  // Wi-Fi 연결이 완료될 때까지 대기
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\nWiFi 연결 성공!");
  Serial.print("IP 주소: ");
  Serial.println(WiFi.localIP());

  // NTP 서버와 시간 동기화
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
}

void loop() {
  // 시간 정보 구조체
  struct tm timeinfo;
  
  // 현재 시간 가져오기
  if (!getLocalTime(&timeinfo)) {
    Serial.println("시간을 가져올 수 없습니다.");
    return;
  }

  // 시간 출력 (1초 간격)
  Serial.printf("현재 한국 시간: %02d:%02d:%02d\n", timeinfo.tm_hour, timeinfo.tm_min, timeinfo.tm_sec);

  delay(1000);  // 1초 대기
}
```
[ChatGPT] 
```
앞에 프로그램 매일 13시15분부터 13시16분까지  26번 포트 on으로 출력하게 프로그램 해줘
```
[생성된 프로그램]  
```
#include <WiFi.h>
#include <time.h>

// Wi-Fi 정보
const char* ssid = "academy5";  // Wi-Fi SSID
const char* password = "";      // 비밀번호 없음

// NTP 서버 설정
const char* ntpServer = "pool.ntp.org";
const long gmtOffset_sec = 9 * 3600;  // 한국 시간대 (GMT +9)
const int daylightOffset_sec = 0;     // 일광 절약 시간 없음

// Output pin 설정
const int outputPin = 26;

void setup() {
  // 시리얼 모니터 시작
  Serial.begin(115200);

  // Wi-Fi 연결 시도
  Serial.printf("WiFi에 연결 중: %s\n", ssid);
  WiFi.begin(ssid, password);

  // Wi-Fi 연결이 완료될 때까지 대기
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\nWiFi 연결 성공!");
  Serial.print("IP 주소: ");
  Serial.println(WiFi.localIP());

  // NTP 서버와 시간 동기화
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);

  // 26번 핀을 출력 모드로 설정
  pinMode(outputPin, OUTPUT);
}

void loop() {
  // 시간 정보 구조체
  struct tm timeinfo;
  
  // 현재 시간 가져오기
  if (!getLocalTime(&timeinfo)) {
    Serial.println("시간을 가져올 수 없습니다.");
    return;
  }

  // 시간 출력 (1초 간격)
  Serial.printf("현재 한국 시간: %02d:%02d:%02d\n", timeinfo.tm_hour, timeinfo.tm_min, timeinfo.tm_sec);

  // 13시 15분부터 13시 16분까지 26번 핀을 ON으로 설정
  if (timeinfo.tm_hour == 13 && timeinfo.tm_min >= 15 && timeinfo.tm_min < 16) {
    digitalWrite(outputPin, HIGH);  // 26번 핀 켬
    Serial.println("26번 핀이 ON 상태입니다.");
  } else {
    digitalWrite(outputPin, LOW);   // 그 외 시간에는 26번 핀 끔
    Serial.println("26번 핀이 OFF 상태입니다.");
  }

  delay(1000);  // 1초 대기
}
```

# MQTT 통신    
[ChatGPT] 
```
브로커=ai.doowon.ac.kr 와이파이정보 ssid=academy5 pw=없음 intopic=intopic/ outtopic=outtopic mqtt 5초에 한번 씩 "김동일" 메세비보내는 통신 프로그램 만들어줘
```
[생성된 프로그램]  
다음은 수정해야 합니다.     
client.connect("ESP32Client")      
여기 ESP32Client 는 고유한 이름으로 고쳐서 넣어 주세요 같은 이름으로 접속하는 사람이 있으면 둘다 통신이 않됩니다. 
```
#include <WiFi.h>
#include <PubSubClient.h>

// Wi-Fi 정보
const char* ssid = "academy5";
const char* password = "";

// MQTT 브로커 정보
const char* mqttServer = "ai.doowon.ac.kr";
const int mqttPort = 1883;

// MQTT 주제
const char* inTopic = "intopic/";
const char* outTopic = "outtopic/";

// 클라이언트 객체
WiFiClient espClient;
PubSubClient client(espClient);

// Wi-Fi 연결 함수
void setupWiFi() {
  delay(10);
  Serial.println();
  Serial.print("WiFi에 연결 중: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi 연결 성공");
  Serial.print("IP 주소: ");
  Serial.println(WiFi.localIP());
}

// MQTT 메시지 수신 콜백 함수
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("메시지 수신 [");
  Serial.print(topic);
  Serial.print("]: ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

// MQTT 브로커 연결 함수
void connectMQTT() {
  while (!client.connected()) {
    //Serial.print("MQTT 브로커에 연결 중...");

    // 클라이언트 ID로 연결 시도
    if (client.connect("ESP32Client")) {
      //Serial.println("MQTT 브로커 연결 성공");

      // intopic을 구독
      client.subscribe(inTopic);
    } else {
      Serial.print("MQTT 연결 실패, 상태코드: ");
      Serial.print(client.state());
      delay(2000);  // 2초 후 재시도
    }
  }
}

void setup() {
  // 시리얼 모니터 시작
  Serial.begin(115200);

  // Wi-Fi 연결
  setupWiFi();

  // MQTT 브로커 설정
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);

  // MQTT 브로커에 연결
  connectMQTT();
}

void loop() {
  // MQTT 연결 유지
  if (!client.connected()) {
    connectMQTT();
  }
  client.loop();

  // 5초마다 메시지 발행
  static unsigned long lastMsg = 0;
  unsigned long now = millis();
  if (now - lastMsg > 5000) {  // 5초마다 실행
    lastMsg = now;
    String message = "김동일";  // 발행할 메시지
    Serial.print("메시지 발행: ");
    Serial.println(message);

    // outtopic에 메시지 발행
    client.publish(inTopic, message.c_str());
  }
}
```
[ChatGPT] 
```
#include <WiFi.h>
#include <PubSubClient.h>

// Wi-Fi 정보
const char* ssid = "academy5";
const char* password = "";

// MQTT 브로커 정보
const char* mqttServer = "ai.doowon.ac.kr";
const int mqttPort = 1883;

// MQTT 주제
const char* inTopic = "intopic/";
const char* outTopic = "outtopic/";

// 클라이언트 객체
WiFiClient espClient;
PubSubClient client(espClient);

// Wi-Fi 연결 함수
void setupWiFi() {
  delay(10);
  Serial.println();
  Serial.print("WiFi에 연결 중: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi 연결 성공");
  Serial.print("IP 주소: ");
  Serial.println(WiFi.localIP());
}

// MQTT 메시지 수신 콜백 함수
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("메시지 수신 [");
  Serial.print(topic);
  Serial.print("]: ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

// MQTT 브로커 연결 함수
void connectMQTT() {
  while (!client.connected()) {
    Serial.print("MQTT 브로커에 연결 중...");

    // 클라이언트 ID로 연결 시도
    if (client.connect("ESP32Client")) {
      Serial.println("MQTT 브로커 연결 성공");

      // intopic을 구독
      client.subscribe(inTopic);
    } else {
      Serial.print("MQTT 연결 실패, 상태코드: ");
      Serial.print(client.state());
      delay(2000);  // 2초 후 재시도
    }
  }
}

void setup() {
  // 시리얼 모니터 시작
  Serial.begin(115200);

  // Wi-Fi 연결
  setupWiFi();

  // MQTT 브로커 설정
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);

  // MQTT 브로커에 연결
  connectMQTT();
}

void loop() {
  // MQTT 연결 유지
  if (!client.connected()) {
    connectMQTT();
  }
  client.loop();

}
 이 프로그램에 
// Output pin numbers
const int outputPins[4] = {26, 27, 32, 33};
// Input pin numbers
const int inputPins[4] = {16, 17, 18, 19};
esp32 wroom 프로그램
{"no":0,"value":1} 수신되면 outputPins[no]가 on이 되게 프로그램 해줘
```

[생성된 프로그램]  
client.connect("ESP32Client") 이름은 수정하세요
```
#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

// Wi-Fi 정보
const char* ssid = "academy5";
const char* password = "";

// MQTT 브로커 정보
const char* mqttServer = "ai.doowon.ac.kr";
const int mqttPort = 1883;

// MQTT 주제
const char* inTopic = "intopic/";
const char* outTopic = "outtopic/";

// Output pin numbers
const int outputPins[4] = {26, 27, 32, 33};

// 클라이언트 객체
WiFiClient espClient;
PubSubClient client(espClient);

// Wi-Fi 연결 함수
void setupWiFi() {
  delay(10);
  Serial.println();
  Serial.print("WiFi에 연결 중: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi 연결 성공");
  Serial.print("IP 주소: ");
  Serial.println(WiFi.localIP());
}

// MQTT 메시지 수신 콜백 함수
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("메시지 수신 [");
  Serial.print(topic);
  Serial.print("]: ");

  // 수신된 payload를 출력
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // JSON 파싱을 위한 버퍼 설정
  StaticJsonDocument<200> doc;

  // 수신된 메시지를 JSON으로 파싱
  DeserializationError error = deserializeJson(doc, payload, length);
  if (error) {
    Serial.print("JSON 파싱 실패: ");
    Serial.println(error.c_str());
    return;
  }

  // JSON 객체에서 "no"와 "value" 추출
  int no = doc["no"];
  int value = doc["value"];

  // 추출한 번호가 유효한지 확인
  if (no >= 0 && no < 4) {
    // value에 따라 핀을 ON/OFF
    if (value == 1) {
      digitalWrite(outputPins[no], HIGH);  // 핀을 ON
      Serial.printf("Pin %d ON\n", outputPins[no]);
    } else {
      digitalWrite(outputPins[no], LOW);   // 핀을 OFF
      Serial.printf("Pin %d OFF\n", outputPins[no]);
    }
  } else {
    Serial.println("유효하지 않은 핀 번호");
  }
}

// MQTT 브로커 연결 함수
void connectMQTT() {
  while (!client.connected()) {
    Serial.print("MQTT 브로커에 연결 중...");

    // 클라이언트 ID로 연결 시도
    if (client.connect("ESP32Client")) {
      Serial.println("MQTT 브로커 연결 성공");

      // intopic을 구독
      client.subscribe(inTopic);
    } else {
      Serial.print("MQTT 연결 실패, 상태코드: ");
      Serial.print(client.state());
      delay(2000);  // 2초 후 재시도
    }
  }
}

void setup() {
  // 시리얼 모니터 시작
  Serial.begin(115200);

  // Wi-Fi 연결
  setupWiFi();

  // MQTT 브로커 설정
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);

  // Output 핀들을 출력 모드로 설정
  for (int i = 0; i < 4; i++) {
    pinMode(outputPins[i], OUTPUT);
    digitalWrite(outputPins[i], LOW);  // 초기 상태는 OFF
  }

  // MQTT 브로커에 연결
  connectMQTT();
}

void loop() {
  // MQTT 연결 유지
  if (!client.connected()) {
    connectMQTT();
  }
  client.loop();
}
```
[ChatGPT] 
```
이 프로그램에서 입력포트를 감시하다가
// Input pin numbers
const int inputPins[4] = {16, 17, 18, 19};
in0=inputPins[0], in1=inputPins[1], in2=inputPins[2], in3=inputPins[3] 
mqtt outtopic 으로 publish 해줘
```
[생성된 프로그램]  
```
#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

// Wi-Fi 정보
const char* ssid = "academy5";
const char* password = "";

// MQTT 브로커 정보
const char* mqttServer = "ai.doowon.ac.kr";
const int mqttPort = 1883;

// MQTT 주제
const char* inTopic = "intopic/";
const char* outTopic = "outtopic/";

// Output pin numbers
const int outputPins[4] = {26, 27, 32, 33};

// Input pin numbers
const int inputPins[4] = {16, 17, 18, 19};

// 클라이언트 객체
WiFiClient espClient;
PubSubClient client(espClient);

// 이전 입력 상태 저장 변수
int lastInputState[4] = {LOW, LOW, LOW, LOW};  // 초기 입력 상태

// Wi-Fi 연결 함수
void setupWiFi() {
  delay(10);
  Serial.println();
  Serial.print("WiFi에 연결 중: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi 연결 성공");
  Serial.print("IP 주소: ");
  Serial.println(WiFi.localIP());
}

// MQTT 메시지 수신 콜백 함수
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("메시지 수신 [");
  Serial.print(topic);
  Serial.print("]: ");

  // 수신된 payload를 출력
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // JSON 파싱을 위한 버퍼 설정
  StaticJsonDocument<200> doc;

  // 수신된 메시지를 JSON으로 파싱
  DeserializationError error = deserializeJson(doc, payload, length);
  if (error) {
    Serial.print("JSON 파싱 실패: ");
    Serial.println(error.c_str());
    return;
  }

  // JSON 객체에서 "no"와 "value" 추출
  int no = doc["no"];
  int value = doc["value"];

  // 추출한 번호가 유효한지 확인
  if (no >= 0 && no < 4) {
    // value에 따라 핀을 ON/OFF
    if (value == 1) {
      digitalWrite(outputPins[no], HIGH);  // 핀을 ON
      Serial.printf("Pin %d ON\n", outputPins[no]);
    } else {
      digitalWrite(outputPins[no], LOW);   // 핀을 OFF
      Serial.printf("Pin %d OFF\n", outputPins[no]);
    }
  } else {
    Serial.println("유효하지 않은 핀 번호");
  }
}

// MQTT 브로커 연결 함수
void connectMQTT() {
  while (!client.connected()) {
    Serial.print("MQTT 브로커에 연결 중...");

    // 클라이언트 ID로 연결 시도
    if (client.connect("ESP32Client")) {
      Serial.println("MQTT 브로커 연결 성공");

      // intopic을 구독
      client.subscribe(inTopic);
    } else {
      Serial.print("MQTT 연결 실패, 상태코드: ");
      Serial.print(client.state());
      delay(2000);  // 2초 후 재시도
    }
  }
}

// 입력 핀 상태를 MQTT로 발행하는 함수
void publishInputState() {
  StaticJsonDocument<200> doc;

  // 각 입력 핀 상태를 읽고, 변경된 경우만 MQTT로 발행
  for (int i = 0; i < 4; i++) {
    int currentState = digitalRead(inputPins[i]);
    if (currentState != lastInputState[i]) {
      // 입력 상태가 변경되었으므로 메시지 생성
      doc.clear();
      doc["input"] = i;
      doc["state"] = currentState;

      char buffer[100];
      serializeJson(doc, buffer);

      // MQTT로 발행
      client.publish(outTopic, buffer);
      Serial.printf("Input %d 상태 변경: %d\n", i, currentState);

      // 이전 상태를 업데이트
      lastInputState[i] = currentState;
    }
  }
}

void setup() {
  // 시리얼 모니터 시작
  Serial.begin(115200);

  // Wi-Fi 연결
  setupWiFi();

  // MQTT 브로커 설정
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);

  // Output 핀들을 출력 모드로 설정
  for (int i = 0; i < 4; i++) {
    pinMode(outputPins[i], OUTPUT);
    digitalWrite(outputPins[i], LOW);  // 초기 상태는 OFF
  }

  // Input 핀들을 입력 모드로 설정
  for (int i = 0; i < 4; i++) {
    pinMode(inputPins[i], INPUT);
  }

  // MQTT 브로커에 연결
  connectMQTT();
}

void loop() {
  // MQTT 연결 유지
  if (!client.connected()) {
    connectMQTT();
  }
  client.loop();

  // 입력 핀 상태 모니터링 및 MQTT 발행
  publishInputState();

  // 짧은 지연 (50ms)
  delay(50);
}
```
[ChatGPT] 
```
// 각 입력 핀 상태를 읽고, 변경된 경우만 MQTT로 발행
  for (int i = 0; i < 4; i++) {
    int currentState = digitalRead(inputPins[i]);
    if (currentState != lastInputState[i]) {
      // 입력 상태가 변경되었으므로 메시지 생성
      doc.clear();
      doc["input"] = i;
      doc["state"] = currentState;

      char buffer[100];
      serializeJson(doc, buffer);

      // MQTT로 발행
      client.publish(outTopic, buffer);
      Serial.printf("Input %d 상태 변경: %d\n", i, currentState);

      // 이전 상태를 업데이트
      lastInputState[i] = currentState;
    }
  }
여기서 상태 변화가 있으면 for loop가 끝나고 in0 in1 in2 in3 4개의 값을 보내줘
```

[생성된 프로그램]
```
#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

// Wi-Fi 정보
const char* ssid = "academy5";
const char* password = "";

// MQTT 브로커 정보
const char* mqttServer = "ai.doowon.ac.kr";
const int mqttPort = 1883;

// MQTT 주제
const char* inTopic = "intopic/";
const char* outTopic = "outtopic/";

// Output pin numbers
const int outputPins[4] = {26, 27, 32, 33};

// Input pin numbers
const int inputPins[4] = {16, 17, 18, 19};

// 클라이언트 객체
WiFiClient espClient;
PubSubClient client(espClient);

// 이전 입력 상태 저장 변수
int lastInputState[4] = {LOW, LOW, LOW, LOW};  // 초기 입력 상태

// Wi-Fi 연결 함수
void setupWiFi() {
  delay(10);
  Serial.println();
  Serial.print("WiFi에 연결 중: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi 연결 성공");
  Serial.print("IP 주소: ");
  Serial.println(WiFi.localIP());
}

// MQTT 메시지 수신 콜백 함수
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("메시지 수신 [");
  Serial.print(topic);
  Serial.print("]: ");

  // 수신된 payload를 출력
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // JSON 파싱을 위한 버퍼 설정
  StaticJsonDocument<200> doc;

  // 수신된 메시지를 JSON으로 파싱
  DeserializationError error = deserializeJson(doc, payload, length);
  if (error) {
    Serial.print("JSON 파싱 실패: ");
    Serial.println(error.c_str());
    return;
  }

  // JSON 객체에서 "no"와 "value" 추출
  int no = doc["no"];
  int value = doc["value"];

  // 추출한 번호가 유효한지 확인
  if (no >= 0 && no < 4) {
    // value에 따라 핀을 ON/OFF
    if (value == 1) {
      digitalWrite(outputPins[no], HIGH);  // 핀을 ON
      Serial.printf("Pin %d ON\n", outputPins[no]);
    } else {
      digitalWrite(outputPins[no], LOW);   // 핀을 OFF
      Serial.printf("Pin %d OFF\n", outputPins[no]);
    }
  } else {
    Serial.println("유효하지 않은 핀 번호");
  }
}

// MQTT 브로커 연결 함수
void connectMQTT() {
  while (!client.connected()) {
    Serial.print("MQTT 브로커에 연결 중...");

    // 클라이언트 ID로 연결 시도
    if (client.connect("ESP32Client1234890")) {
      Serial.println("MQTT 브로커 연결 성공");

      // intopic을 구독
      client.subscribe(inTopic);
    } else {
      Serial.print("MQTT 연결 실패, 상태코드: ");
      Serial.print(client.state());
      delay(2000);  // 2초 후 재시도
    }
  }
}

// 입력 핀 상태를 MQTT로 발행하는 함수
void publishInputState() {
  bool stateChanged = false;  // 상태 변화 여부 확인 변수
  StaticJsonDocument<200> doc;

  // 각 입력 핀 상태를 읽고, 상태 변화가 있는지 확인
  for (int i = 0; i < 4; i++) {
    int currentState = digitalRead(inputPins[i]);
    if (currentState != lastInputState[i]) {
      stateChanged = true;  // 상태가 변경된 경우 true로 설정
      lastInputState[i] = currentState;  // 상태 업데이트
    }
  }

  // 상태 변화가 있으면 모든 핀의 상태를 MQTT로 발행
  if (stateChanged) {
    // 각 핀 상태를 JSON으로 저장
    for (int i = 0; i < 4; i++) {
      doc["in" + String(i)] = lastInputState[i];  // "in0", "in1", "in2", "in3"로 핀 상태 저장
    }

    // JSON을 문자열로 직렬화하여 MQTT로 발행
    char buffer[200];
    serializeJson(doc, buffer);

    // MQTT로 발행
    client.publish(outTopic, buffer);
    Serial.println("모든 입력 핀 상태 발행:");
    Serial.println(buffer);  // 발행한 메시지 시리얼 모니터에 출력
  }
}

void setup() {
  // 시리얼 모니터 시작
  Serial.begin(115200);

  // Wi-Fi 연결
  setupWiFi();

  // MQTT 브로커 설정
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);

  // Output 핀들을 출력 모드로 설정
  for (int i = 0; i < 4; i++) {
    pinMode(outputPins[i], OUTPUT);
    digitalWrite(outputPins[i], LOW);  // 초기 상태는 OFF
  }

  // Input 핀들을 입력 모드로 설정
  for (int i = 0; i < 4; i++) {
    pinMode(inputPins[i], INPUT);
  }

  // MQTT 브로커에 연결
  connectMQTT();
}

void loop() {
  // MQTT 연결 유지
  if (!client.connected()) {
    connectMQTT();
  }
  client.loop();

  // 입력 핀 상태 모니터링 및 MQTT 발행
  publishInputState();

  // 짧은 지연 (50ms)
  delay(50);
}
```
# Node RED
다음 그림은 스위치의 동작에 따라 Led 가 on off 합니다. 이것을 ChatGPT를 이용해 간단하게 오디오 음성으로 출력을 내보내는 프로그램을 만들어 보겠습니다.
<img src="https://github.com/user-attachments/assets/4aaa9657-a041-4de6-a86d-0b6ec39ab798" alt="연습문제1" width="200">

[node red 프로그램]
```
[
    {
        "id": "e0ccba070e41516f",
        "type": "ui_switch",
        "z": "16808b1eb2fdb744",
        "name": "",
        "label": "switch",
        "tooltip": "",
        "group": "3e39067bccc61553",
        "order": 0,
        "width": 0,
        "height": 0,
        "passthru": true,
        "decouple": "false",
        "topic": "topic",
        "topicType": "msg",
        "style": "",
        "onvalue": "true",
        "onvalueType": "bool",
        "onicon": "",
        "oncolor": "",
        "offvalue": "false",
        "offvalueType": "bool",
        "officon": "",
        "offcolor": "",
        "animate": false,
        "className": "",
        "x": 170,
        "y": 100,
        "wires": [
            [
                "a5f6f1bc8a1b6947"
            ]
        ]
    },
    {
        "id": "a5f6f1bc8a1b6947",
        "type": "ui_led",
        "z": "16808b1eb2fdb744",
        "order": 1,
        "group": "3e39067bccc61553",
        "width": 0,
        "height": 0,
        "label": "",
        "labelPlacement": "left",
        "labelAlignment": "left",
        "colorForValue": [
            {
                "color": "#ff0000",
                "value": "false",
                "valueType": "bool"
            },
            {
                "color": "#008000",
                "value": "true",
                "valueType": "bool"
            }
        ],
        "allowColorForValueInMessage": false,
        "shape": "circle",
        "showGlow": true,
        "name": "",
        "x": 330,
        "y": 100,
        "wires": []
    },
    {
        "id": "f152d3d26742f666",
        "type": "ui_audio",
        "z": "16808b1eb2fdb744",
        "name": "",
        "group": "3e39067bccc61553",
        "voice": "Google 한국의",
        "always": "",
        "x": 340,
        "y": 160,
        "wires": []
    },
    {
        "id": "3e39067bccc61553",
        "type": "ui_group",
        "name": "스위치",
        "tab": "d83f9ecf636984b5",
        "order": 1,
        "disp": true,
        "width": "6",
        "collapse": false,
        "className": ""
    },
    {
        "id": "d83f9ecf636984b5",
        "type": "ui_tab",
        "name": "Tab 1",
        "icon": "dashboard",
        "order": 1
    }
]
```
[ChatGPG]
```
다음은 node red 프로그램 입니다. 스위가 켜지면 "스위치가 켜졌습니다." 하고 오디오로 출력이 나가고 스위가 꺼지면 "스위치가 꺼졌습니다." 하고 오디오로 출력하는 프로그램 만들어줘
```
다음은 ChatGPT가 작성한 프로그램 입니다.
[결과 프로그램]    
<img src="https://github.com/user-attachments/assets/387555c1-d6ac-4783-afcb-497125bfe703" alt="연습문제1" width="200">
```
[
    {
        "id": "f2dbecbb8b139fb9",
        "type": "ui_switch",
        "z": "16808b1eb2fdb744",
        "name": "",
        "label": "switch",
        "tooltip": "",
        "group": "3e39067bccc61553",
        "order": 0,
        "width": 0,
        "height": 0,
        "passthru": true,
        "decouple": "false",
        "topic": "topic",
        "topicType": "msg",
        "style": "",
        "onvalue": "true",
        "onvalueType": "bool",
        "onicon": "",
        "oncolor": "",
        "offvalue": "false",
        "offvalueType": "bool",
        "officon": "",
        "offcolor": "",
        "animate": false,
        "className": "",
        "x": 270,
        "y": 300,
        "wires": [
            [
                "c205868608e19610",
                "e51cba5ed050ad8b"
            ]
        ]
    },
    {
        "id": "c205868608e19610",
        "type": "ui_led",
        "z": "16808b1eb2fdb744",
        "order": 1,
        "group": "3e39067bccc61553",
        "width": 0,
        "height": 0,
        "label": "",
        "labelPlacement": "left",
        "labelAlignment": "left",
        "colorForValue": [
            {
                "color": "#ff0000",
                "value": "false",
                "valueType": "bool"
            },
            {
                "color": "#008000",
                "value": "true",
                "valueType": "bool"
            }
        ],
        "allowColorForValueInMessage": false,
        "shape": "circle",
        "showGlow": true,
        "name": "",
        "x": 430,
        "y": 300,
        "wires": []
    },
    {
        "id": "e51cba5ed050ad8b",
        "type": "function",
        "z": "16808b1eb2fdb744",
        "name": "Set Audio Message",
        "func": "if (msg.payload === true) {\n    msg.payload = \"스위치가 켜졌습니다.\";\n} else {\n    msg.payload = \"스위치가 꺼졌습니다.\";\n}\nreturn msg;",
        "outputs": 1,
        "timeout": "",
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 420,
        "y": 380,
        "wires": [
            [
                "ba6cd5c7b5e48307"
            ]
        ]
    },
    {
        "id": "ba6cd5c7b5e48307",
        "type": "ui_audio",
        "z": "16808b1eb2fdb744",
        "name": "",
        "group": "3e39067bccc61553",
        "voice": "Google 한국의",
        "always": "",
        "x": 600,
        "y": 380,
        "wires": []
    },
    {
        "id": "3e39067bccc61553",
        "type": "ui_group",
        "name": "스위치",
        "tab": "d83f9ecf636984b5",
        "order": 1,
        "disp": true,
        "width": "6",
        "collapse": false,
        "className": ""
    },
    {
        "id": "d83f9ecf636984b5",
        "type": "ui_tab",
        "name": "Tab 1",
        "icon": "dashboard",
        "order": 1
    }
]
```
### [프로젝트] 스위치로 i2r-03 보드 on off 제어하기
다음은 node red 소스 프로그램 입니다. topic 과 맥 어드레스는 수정해서 입력하면 보유한 IoT-PLC를 제어 할 수 있습니다.
<img src="https://github.com/user-attachments/assets/cfa246dc-5945-470e-a59c-f47b81525dd6" alt="연습문제1" width="200">    
topic은 "i2r/email/in" 여기에 email은 어플에서 블루투스로 정보를 입력할 때의 email 정보를 입력하세요. 

```
[
    {
        "id": "e0ccba070e41516f",
        "type": "ui_switch",
        "z": "16808b1eb2fdb744",
        "name": "",
        "label": "switch",
        "tooltip": "",
        "group": "3e39067bccc61553",
        "order": 0,
        "width": 0,
        "height": 0,
        "passthru": true,
        "decouple": "false",
        "topic": "topic",
        "topicType": "msg",
        "style": "",
        "onvalue": "{\"mac\":\"A0:A3:B3:0D:8F:54\",\"order\":2,\"no\":1,\"value\":true}",
        "onvalueType": "json",
        "onicon": "",
        "oncolor": "",
        "offvalue": "{\"mac\":\"A0:A3:B3:0D:8F:54\",\"order\":2,\"no\":1,\"value\":false}",
        "offvalueType": "json",
        "officon": "",
        "offcolor": "",
        "animate": false,
        "className": "",
        "x": 170,
        "y": 200,
        "wires": [
            [
                "16036357d7953f7c"
            ]
        ]
    },
    {
        "id": "ef2789e281a654aa",
        "type": "inject",
        "z": "16808b1eb2fdb744",
        "name": "on",
        "props": [
            {
                "p": "payload"
            },
            {
                "p": "topic",
                "vt": "str"
            }
        ],
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "topic": "",
        "payload": "{\"mac\":\"A0:A3:B3:0D:8F:54\",\"order\":2,\"no\":1,\"value\":true}",
        "payloadType": "json",
        "x": 170,
        "y": 100,
        "wires": [
            [
                "16036357d7953f7c"
            ]
        ]
    },
    {
        "id": "16036357d7953f7c",
        "type": "mqtt out",
        "z": "16808b1eb2fdb744",
        "name": "",
        "topic": "i2r/a/in",
        "qos": "0",
        "retain": "false",
        "respTopic": "",
        "contentType": "",
        "userProps": "",
        "correl": "",
        "expiry": "",
        "broker": "b091c9249183897c",
        "x": 350,
        "y": 100,
        "wires": []
    },
    {
        "id": "0f782ff329062e9a",
        "type": "inject",
        "z": "16808b1eb2fdb744",
        "name": "off",
        "props": [
            {
                "p": "payload"
            },
            {
                "p": "topic",
                "vt": "str"
            }
        ],
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "topic": "",
        "payload": "{\"mac\":\"A0:A3:B3:0D:8F:54\",\"order\":2,\"no\":1,\"value\":false}",
        "payloadType": "json",
        "x": 170,
        "y": 140,
        "wires": [
            [
                "16036357d7953f7c"
            ]
        ]
    },
    {
        "id": "3e39067bccc61553",
        "type": "ui_group",
        "name": "스위치",
        "tab": "d83f9ecf636984b5",
        "order": 1,
        "disp": true,
        "width": "6",
        "collapse": false,
        "className": ""
    },
    {
        "id": "b091c9249183897c",
        "type": "mqtt-broker",
        "name": "",
        "broker": "ai.doowon.ac.kr",
        "port": "1883",
        "clientid": "",
        "autoConnect": true,
        "usetls": false,
        "protocolVersion": "4",
        "keepalive": "60",
        "cleansession": true,
        "autoUnsubscribe": true,
        "birthTopic": "",
        "birthQos": "0",
        "birthRetain": "false",
        "birthPayload": "",
        "birthMsg": {},
        "closeTopic": "",
        "closeQos": "0",
        "closeRetain": "false",
        "closePayload": "",
        "closeMsg": {},
        "willTopic": "",
        "willQos": "0",
        "willRetain": "false",
        "willPayload": "",
        "willMsg": {},
        "userProps": "",
        "sessionExpiry": ""
    },
    {
        "id": "d83f9ecf636984b5",
        "type": "ui_tab",
        "name": "Tab 1",
        "icon": "dashboard",
        "order": 1
    }
]
```
# Remote-RED
```
[
    {
        "id": "4595b4555ae50c3a",
        "type": "mqtt out",
        "z": "0df6564762a7dd10",
        "name": "",
        "topic": "i2r/kdi6033@gmail.com/in",
        "qos": "0",
        "retain": "false",
        "respTopic": "",
        "contentType": "",
        "userProps": "",
        "correl": "",
        "expiry": "",
        "broker": "f15d66b3.c7edf8",
        "x": 450,
        "y": 100,
        "wires": []
    },
    {
        "id": "593b80907d283a61",
        "type": "ui_switch",
        "z": "0df6564762a7dd10",
        "name": "",
        "label": "switch",
        "tooltip": "",
        "group": "3e39067bccc61553",
        "order": 0,
        "width": 0,
        "height": 0,
        "passthru": true,
        "decouple": "false",
        "topic": "topic",
        "topicType": "msg",
        "style": "",
        "onvalue": "{\"mac\":\"A0:A3:B3:0D:8F:54\",\"order\":2,\"no\":1,\"value\":true}",
        "onvalueType": "json",
        "onicon": "",
        "oncolor": "",
        "offvalue": "{\"mac\":\"A0:A3:B3:0D:8F:54\",\"order\":2,\"no\":1,\"value\":false}",
        "offvalueType": "json",
        "officon": "",
        "offcolor": "",
        "animate": false,
        "className": "",
        "x": 250,
        "y": 100,
        "wires": [
            [
                "4595b4555ae50c3a"
            ]
        ]
    },
    {
        "id": "24fd0712d0caec23",
        "type": "mqtt in",
        "z": "0df6564762a7dd10",
        "name": "",
        "topic": "i2r/kdi6033@gmail.com/out",
        "qos": "0",
        "datatype": "auto",
        "broker": "f15d66b3.c7edf8",
        "nl": false,
        "rap": true,
        "rh": 0,
        "inputs": 0,
        "x": 290,
        "y": 260,
        "wires": [
            [
                "80936eaccdb8fcd8"
            ]
        ]
    },
    {
        "id": "8e6ef4af57a25e22",
        "type": "ui_gauge",
        "z": "0df6564762a7dd10",
        "name": "",
        "group": "3e39067bccc61553",
        "order": 1,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "온도",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "40",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "20",
        "seg2": "30",
        "diff": false,
        "className": "",
        "x": 810,
        "y": 260,
        "wires": []
    },
    {
        "id": "80936eaccdb8fcd8",
        "type": "json",
        "z": "0df6564762a7dd10",
        "name": "",
        "property": "payload",
        "action": "",
        "pretty": false,
        "x": 490,
        "y": 260,
        "wires": [
            [
                "b4cdc55ca8e7eee1",
                "b2a81fe0eb374e05",
                "d721b1b0e64e3bcb"
            ]
        ]
    },
    {
        "id": "b4cdc55ca8e7eee1",
        "type": "function",
        "z": "0df6564762a7dd10",
        "name": "function 1",
        "func": "var newMsg={};\nnewMsg.payload = msg.payload.temp;\nreturn newMsg;",
        "outputs": 1,
        "timeout": 0,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 640,
        "y": 260,
        "wires": [
            [
                "8e6ef4af57a25e22"
            ]
        ]
    },
    {
        "id": "4a0391a494e09ab2",
        "type": "ui_gauge",
        "z": "0df6564762a7dd10",
        "name": "",
        "group": "3e39067bccc61553",
        "order": 2,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "습도",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "100",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "30",
        "seg2": "70",
        "diff": false,
        "className": "",
        "x": 810,
        "y": 320,
        "wires": []
    },
    {
        "id": "b2a81fe0eb374e05",
        "type": "function",
        "z": "0df6564762a7dd10",
        "name": "function 2",
        "func": "var newMsg={};\nnewMsg.payload = msg.payload.humi;\nreturn newMsg;",
        "outputs": 1,
        "timeout": 0,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 640,
        "y": 320,
        "wires": [
            [
                "4a0391a494e09ab2"
            ]
        ]
    },
    {
        "id": "f33df5d02afdd23a",
        "type": "ui_led",
        "z": "0df6564762a7dd10",
        "order": 3,
        "group": "3e39067bccc61553",
        "width": 0,
        "height": 0,
        "label": "",
        "labelPlacement": "left",
        "labelAlignment": "left",
        "colorForValue": [
            {
                "color": "#ff0000",
                "value": "false",
                "valueType": "bool"
            },
            {
                "color": "#008000",
                "value": "true",
                "valueType": "bool"
            }
        ],
        "allowColorForValueInMessage": false,
        "shape": "circle",
        "showGlow": true,
        "name": "",
        "x": 810,
        "y": 380,
        "wires": []
    },
    {
        "id": "d721b1b0e64e3bcb",
        "type": "function",
        "z": "0df6564762a7dd10",
        "name": "function 3",
        "func": "var newMsg={};\nif(msg.payload.in[0] === 1)\n    newMsg.payload = true;\nelse \n    newMsg.payload = false;\nreturn newMsg;",
        "outputs": 1,
        "timeout": 0,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 640,
        "y": 380,
        "wires": [
            [
                "f33df5d02afdd23a"
            ]
        ]
    },
    {
        "id": "f15d66b3.c7edf8",
        "type": "mqtt-broker",
        "name": "",
        "broker": "ai.doowon.ac.kr",
        "port": "1883",
        "tls": "509a45a9.089bbc",
        "clientid": "",
        "autoConnect": true,
        "usetls": false,
        "protocolVersion": "3",
        "keepalive": "60",
        "cleansession": true,
        "autoUnsubscribe": true,
        "birthTopic": "",
        "birthQos": "0",
        "birthPayload": "",
        "birthMsg": {},
        "closeTopic": "",
        "closeQos": "0",
        "closePayload": "",
        "closeMsg": {},
        "willTopic": "",
        "willQos": "0",
        "willPayload": "",
        "willMsg": {},
        "userProps": "",
        "sessionExpiry": ""
    },
    {
        "id": "3e39067bccc61553",
        "type": "ui_group",
        "name": "스위치",
        "tab": "d83f9ecf636984b5",
        "order": 1,
        "disp": true,
        "width": "6",
        "collapse": false,
        "className": ""
    },
    {
        "id": "509a45a9.089bbc",
        "type": "tls-config",
        "name": "",
        "cert": "",
        "key": "",
        "ca": "",
        "certname": "94191b3ef7-certificate.pem.crt",
        "keyname": "94191b3ef7-private.pem.key",
        "caname": "MtestCA.pem",
        "servername": "",
        "verifyservercert": true,
        "alpnprotocol": ""
    },
    {
        "id": "d83f9ecf636984b5",
        "type": "ui_tab",
        "name": "Tab 1",
        "icon": "dashboard",
        "order": 1
    }
]
```
