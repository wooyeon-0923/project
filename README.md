[[[[#include <SoftwareSerial.h>

// Grove Base Shield의 D2, D3 포트를 사용
SoftwareSerial mySerial(2, 3); // RX, TX

byte receivedData[9];
int co2_concentration = 0;

void setup() {
    Serial.begin(9600);
    mySerial.begin(9600); // CM1106 센서와 통신
    Serial.println("CM1106 초기화 완료");
}

void loop() {
    if (mySerial.available() >= 9) { // 데이터가 9바이트 이상인지 확인
        mySerial.readBytes(receivedData, 9);

        if (receivedData[0] == 0xFF && receivedData[1] == 0x86) { // 헤더 확인
            co2_concentration = (receivedData[2] << 8) | receivedData[3]; // CO2 계산
            Serial.print("CO2 농도 (ppm): ");
            Serial.println(co2_concentration);
        } else {
            Serial.println("데이터 오류");
        }
    }

    delay(1000); // 1초 간격
}
](https://docs.google.com/presentation/d/1pyIsjvTLsRYAT1k4PAtZAQIQBezVqqahGu_nZ46ccF4/edit#slide=id.g31b05e4f5e1_0_55)](https://docs.google.com/presentation/d/1pyIsjvTLsRYAT1k4PAtZAQIQBezVqqahGu_nZ46ccF4/edit#slide=id.g31b05e4f5e1_0_55)](https://docs.google.com/presentation/d/1pyIsjvTLsRYAT1k4PAtZAQIQBezVqqahGu_nZ46ccF4/edit#slide=id.g31b05e4f5e1_0_55)](https://docs.google.com/presentation/d/1pyIsjvTLsRYAT1k4PAtZAQIQBezVqqahGu_nZ46ccF4/edit#slide=id.g31b05e4f5e1_0_55)
