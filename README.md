Arduino의 데이터를 CSV 파일로 저장하려면 아래 방법을 사용할 수 있습니다. Arduino 자체로 CSV 파일을 저장할 수는 없지만, 데이터를 PC로 전송한 뒤 Python 스크립트를 사용해 저장할 수 있습니다.

### **1. Arduino에서 데이터를 직렬 포트로 전송**
기존 코드에 데이터를 CSV 형식으로 직렬 출력하도록 수정합니다. Python에서 이 데이터를 읽어와 CSV 파일로 저장합니다.

#### 수정된 Arduino 코드
```cpp
int pin = 8;
unsigned long duration;
unsigned long starttime;
unsigned long sampletime_ms = 30000;  // Sampling for 30 seconds
unsigned long lowpulseoccupancy = 0;
float ratio = 0;
float concentration = 0;

void setup()
{
    Serial.begin(9600);
    pinMode(pin, INPUT);
    starttime = millis();
    Serial.println("Starting dust measurement...");
    Serial.println("==============================");
}

void loop()
{
    duration = pulseIn(pin, LOW);
    lowpulseoccupancy = lowpulseoccupancy + duration;

    if ((millis() - starttime) > sampletime_ms)  // Measure every 30 seconds
    {
        ratio = lowpulseoccupancy / (sampletime_ms * 10.0);
        concentration = 1.1 * pow(ratio, 3) - 3.8 * pow(ratio, 2) + 520 * ratio + 0.62; // Unit: ug/m3

        Serial.println("==============================");
        Serial.print("Dust concentration: ");
        Serial.print(concentration);
        Serial.println(" ug/m3");

        // Determine air quality
        Serial.print("Air quality: ");
        if (concentration <= 30) {
            Serial.println("Good");
        } else if (concentration <= 80) {
            Serial.println("Moderate");
        } else if (concentration <= 150) {
            Serial.println("Unhealthy");
        } else {
            Serial.println("Very Unhealthy");
        }

        Serial.println("------------------------------");
        Serial.print("Measurement time: ");
        Serial.print(millis() / 1000);
        Serial.println(" seconds");
        Serial.println("==============================\n");

        // Reset for the next measurement
        lowpulseoccupancy = 0;
        starttime = millis();
    }
}


---

### **2. Python 스크립트로 데이터 수집 및 CSV 저장**
Arduino에서 출력된 데이터를 Python으로 읽어와 CSV 파일에 저장합니다.

#### Python 코드
```python
import serial
import csv
import time

# Arduino가 연결된 시리얼 포트와 보드 속도 설정
SERIAL_PORT = 'COM3'  # Arduino가 연결된 포트를 확인 후 설정 (예: '/dev/ttyUSB0' 또는 'COM3')
BAUD_RATE = 9600
CSV_FILE = "dust_sensor_data.csv"

# CSV 파일 초기화
def initialize_csv(file_path):
    with open(file_path, mode='w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(["Time (s)", "Concentration (ug/m3)", "Air Quality"])  # 헤더 추가

# CSV 파일에 데이터 추가
def append_to_csv(file_path, data):
    with open(file_path, mode='a', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(data)

def main():
    initialize_csv(CSV_FILE)
    try:
        # 시리얼 포트 연결
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1) as ser:
            print("Receiving data from Arduino...")
            while True:
                # 시리얼 데이터 읽기
                line = ser.readline().decode('utf-8').strip()
                if line:
                    print(f"Received: {line}")
                    # CSV 파일에 데이터 추가
                    data = line.split(",")
                    if len(data) == 3:  # 시간, 농도, 대기질 상태
                        append_to_csv(CSV_FILE, data)
    except serial.SerialException as e:
        print(f"Serial error: {e}")
    except KeyboardInterrupt:
        print("Data collection stopped.")

if __name__ == "__main__":
    main()
```

---

### **3. 실행 순서**
1. **Arduino 코드 업로드**:
   - 위 수정된 Arduino 코드를 업로드합니다.
   - 시리얼 모니터에서 데이터가 출력되는지 확인합니다.

2. **Python 스크립트 실행**:
   - Python 스크립트를 실행하여 데이터를 CSV 파일로 저장합니다.
   - `dust_sensor_data.csv` 파일이 생성되며 실시간 데이터가 추가됩니다.

---

### **CSV 파일 예시**
`dust_sensor_data.csv` 내용:
```csv
Time (s),Concentration (ug/m3),Air Quality
30,45.2,Moderate
60,75.8,Unhealthy
90,22.5,Good
```

---

### **추가 팁**
- **포트 확인**: `COM3` 또는 `/dev/ttyUSB0`처럼 연결된 포트를 올바르게 설정해야 합니다.
- **시리얼 모니터 종료**: Arduino IDE의 시리얼 모니터가 열려 있으면 Python에서 포트를 열 수 없으니 닫아야 합니다.
- **데이터 시각화**: 저장된 CSV 파일을 Pandas와 Matplotlib를 사용하여 분석 및 시각화할 수 있습니다.

필요한 추가 구현 사항이 있으면 말씀해주세요! 😊
