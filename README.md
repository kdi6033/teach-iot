# teach-iot
[교육지원사이트 경제과학진흥원](https://www.gbedu.or.kr/gbsa/education/course/view.do?deGrCode=DE_000002140&menuNo=400032)    

[ChatGPT]    
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


