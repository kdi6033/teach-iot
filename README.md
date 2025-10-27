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
