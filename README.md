'''
#include <SoftwareSerial.h>

// CM1106 센서용 직렬 통신 핀 설정
SoftwareSerial mySerial(2, 3); // RX, TX (CM1106과 연결)

// CM1106 데이터 저장 변수
byte receivedData[9]; // CM1106 데이터는 9바이트 길이
int co2_concentration = 0;

// CM1106 데이터 읽기 함수
void readCM1106() {
    if (mySerial.available() >= 9) { // 데이터 길이 확인
        mySerial.readBytes(receivedData, 9);

        // 데이터 유효성 검사 (헤더 및 체크섬 확인)
        if (receivedData[0] == 0xFF && receivedData[1] == 0x86) {
            co2_concentration = (receivedData[2] << 8) | receivedData[3]; // CO2 농도 계산
        } else {
            co2_concentration = -1; // 데이터 오류 시 -1 반환
        }
    }
}

'''
