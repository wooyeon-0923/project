CM1106 (CO2 센서)와 Grove Dust Sensor 데이터를 측정한 뒤, CSV 파일로 저장할 수 있도록 코드를 수정했습니다. Arduino는 자체적으로 CSV 파일을 저장할 수 없으므로 데이터를 시리얼 통신을 통해 PC로 전송하고, Python을 사용해 저장합니다.

---

### **수정된 Arduino 코드 (CSV 형식 출력)**

```cpp
#include <cm1106_i2c.h>

CM1106_I2C cm1106_i2c;

// Dust Sensor Configuration
int dustPin = 8;
unsigned long duration;
unsigned long starttime;
unsigned long sampletime_ms = 5000;  // Sampling for 5 seconds
unsigned long lowpulseoccupancy = 0;
float ratio = 0;
float dustConcentration = 0;

void setup() {
  // Initialize CM1106
  cm1106_i2c.begin();
  Serial.begin(9600);

  delay(1000);
  cm1106_i2c.read_serial_number();
  delay(1000);
  cm1106_i2c.check_sw_version();
  delay(1000);

  // Initialize Dust Sensor
  pinMode(dustPin, INPUT);
  starttime = millis();

  Serial.println("time,co2,dust_concentration,air_quality"); // CSV Header
}

void loop() {
  // CO2 Measurement
  int co2Concentration = -1;  // Default value if CO2 measurement fails
  uint8_t ret = cm1106_i2c.measure_result();
  if (ret == 0) {
    co2Concentration = cm1106_i2c.co2;
  }

  // Dust Measurement
  duration = pulseIn(dustPin, LOW);
  lowpulseoccupancy += duration;

  if ((millis() - starttime) > sampletime_ms) {
    ratio = lowpulseoccupancy / (sampletime_ms * 10.0);
    dustConcentration = 1.1 * pow(ratio, 3) - 3.8 * pow(ratio, 2) + 520 * ratio + 0.62; // Unit: ug/m3

    // Determine air quality
    String airQuality;
    if (dustConcentration <= 30) {
      airQuality = "Good";
    } else if (dustConcentration <= 80) {
      airQuality = "Moderate";
    } else if (dustConcentration <= 150) {
      airQuality = "Unhealthy";
    } else {
      airQuality = "Very Unhealthy";
    }

    // Print CSV-formatted data
    Serial.print(millis() / 1000); // Time in seconds
    Serial.print(",");
    Serial.print(co2Concentration); // CO2 Concentration
    Serial.print(",");
    Serial.print(dustConcentration); // Dust Concentration
    Serial.print(",");
    Serial.println(airQuality); // Air Quality

    // Reset variables for next measurement
    lowpulseoccupancy = 0;
    starttime = millis();
  }

  // Delay to ensure 5-second interval
  delay(5000 - (millis() - starttime));
}
```

---

### **Python 코드로 CSV 저장**

아래 Python 스크립트는 Arduino에서 시리얼로 출력된 데이터를 읽어와 CSV 파일로 저장합니다.

#### **Python 코드**
```python
import serial
import csv

# Arduino 시리얼 포트 설정
SERIAL_PORT = 'COM3'  # Replace with your Arduino port (e.g., '/dev/ttyUSB0' or 'COM3')
BAUD_RATE = 9600
CSV_FILE = "sensor_data.csv"

# CSV 파일 초기화
def initialize_csv(file_path):
    with open(file_path, mode='w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(["Time (s)", "CO2 (ppm)", "Dust Concentration (ug/m3)", "Air Quality"])  # Header

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
                if line and "," in line:  # CSV 형식인지 확인
                    print(f"Received: {line}")
                    data = line.split(",")  # Split data by comma
                    if len(data) == 4:  # Ensure there are 4 columns (Time, CO2, Dust, Air Quality)
                        append_to_csv(CSV_FILE, data)
    except serial.SerialException as e:
        print(f"Serial error: {e}")
    except KeyboardInterrupt:
        print("Data collection stopped.")

if __name__ == "__main__":
    main()
```

---

### **CSV 출력 예시**

`sensor_data.csv` 파일:
```csv
Time (s),CO2 (ppm),Dust Concentration (ug/m3),Air Quality
5,400,25.3,Good
10,405,50.2,Moderate
15,410,120.1,Unhealthy
20,420,170.5,Very Unhealthy
```

---

### **주의 사항**
1. **포트 확인**: `COM3` 또는 `/dev/ttyUSB0` 등 Arduino가 연결된 포트를 정확히 설정해야 합니다.
2. **시리얼 모니터 닫기**: Arduino IDE의 시리얼 모니터가 열려 있으면 Python에서 포트를 열 수 없습니다.
3. **I2C 주소 확인**: CM1106과 다른 I2C 장치의 주소가 충돌하지 않도록 확인하세요.

추가 질문이 있거나 개선 사항이 필요하면 언제든지 말씀해주세요! 😊
