'''''

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

'''''

'''''

#include <SoftwareSerial.h>

int dust_pin = 8; // Dust Sensor 핀
unsigned long duration;
unsigned long starttime;
unsigned long sampletime_ms = 30000; // 샘플링 시간: 30초
unsigned long lowpulseoccupancy = 0;
float ratio = 0;
float dust_concentration = 0;

// CM1106용 SoftwareSerial 설정
SoftwareSerial mySerial(2, 3); // RX, TX
byte receivedData[9];
int co2_concentration = 0;

void setup() {
    Serial.begin(9600);
    pinMode(dust_pin, INPUT);
    starttime = millis();

    // CM1106 직렬 통신 시작
    mySerial.begin(9600);
    Serial.println("Dust Sensor와 CM1106 초기화 완료");
}

void loop() {
    // Dust Sensor 측정
    duration = pulseIn(dust_pin, LOW);
    lowpulseoccupancy += duration;

    if ((millis() - starttime) > sampletime_ms) {
        ratio = lowpulseoccupancy / (sampletime_ms * 10.0);
        dust_concentration = 1.1 * pow(ratio, 3) - 3.8 * pow(ratio, 2) + 520 * ratio + 0.62;

        Serial.println("==============================");
        Serial.print("미세먼지 농도 (µg/m³): ");
        Serial.println(dust_concentration);

        lowpulseoccupancy = 0;
        starttime = millis();
    }

    // CM1106 CO2 농도 측정
    readCM1106();
    Serial.print("CO2 농도 (ppm): ");
    if (co2_concentration != -1) {
        Serial.println(co2_concentration);
    } else {
        Serial.println("데이터 오류");
    }

    delay(1000); // 측정 간격
}

// CM1106 데이터 읽기 함수
void readCM1106() {
    if (mySerial.available() >= 9) {
        mySerial.readBytes(receivedData, 9);

        if (receivedData[0] == 0xFF && receivedData[1] == 0x86) {
            co2_concentration = (receivedData[2] << 8) | receivedData[3];
        } else {
            co2_concentration = -1;
        }
    }
}

'''''

