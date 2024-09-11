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
