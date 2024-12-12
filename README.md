아래는 각각의 센서 데이터를 **별도로 측정**하고, **CSV 파일로 저장**하는 방법입니다. 두 개의 Python 스크립트를 사용해 각 센서의 데이터를 독립적으로 관리하면서 측정 시간을 5초로 통일합니다.

---

### **1. Dust Sensor Arduino 코드**

Dust 센서 데이터를 5초마다 출력하며, CSV 포맷으로 출력합니다.

```cpp
int pin = 8;
unsigned long duration;
unsigned long starttime;
unsigned long sampletime_ms = 5000;  // Sampling for 5 seconds
unsigned long lowpulseoccupancy = 0;
float ratio = 0;
float concentration = 0;

void setup() {
    Serial.begin(9600);
    pinMode(pin, INPUT);
    starttime = millis();
    Serial.println("time,dust_concentration,air_quality"); // CSV Header
}

void loop() {
    duration = pulseIn(pin, LOW);
    lowpulseoccupancy = lowpulseoccupancy + duration;

    if ((millis() - starttime) > sampletime_ms) {  // Measure every 5 seconds
        ratio = lowpulseoccupancy / (sampletime_ms * 10.0);
        concentration = 1.1 * pow(ratio, 3) - 3.8 * pow(ratio, 2) + 520 * ratio + 0.62; // Unit: ug/m3

        // Determine air quality
        String airQuality;
        if (concentration <= 30) {
            airQuality = "Good";
        } else if (concentration <= 80) {
            airQuality = "Moderate";
        } else if (concentration <= 150) {
            airQuality = "Unhealthy";
        } else {
            airQuality = "Very Unhealthy";
        }

        // Print CSV-formatted data
        Serial.print(millis() / 1000); // Time in seconds
        Serial.print(",");
        Serial.print(concentration); // Dust concentration
        Serial.print(",");
        Serial.println(airQuality); // Air quality

        // Reset for next measurement
        lowpulseoccupancy = 0;
        starttime = millis();
    }
}
```

---

### **2. CM1106 Arduino 코드**

CM1106 데이터를 5초마다 출력하며, CSV 포맷으로 출력합니다.

```cpp
#include <cm1106_i2c.h>

CM1106_I2C cm1106_i2c;

void setup() {
    cm1106_i2c.begin();
    Serial.begin(9600);
    delay(1000);
    cm1106_i2c.read_serial_number();
    delay(1000);
    cm1106_i2c.check_sw_version();
    delay(1000);

    Serial.println("time,co2_concentration"); // CSV Header
}

void loop() {
    uint8_t ret = cm1106_i2c.measure_result();
    if (ret == 0) {
        // Print CSV-formatted data
        Serial.print(millis() / 1000); // Time in seconds
        Serial.print(",");
        Serial.println(cm1106_i2c.co2); // CO2 concentration
    }
    delay(5000); // Wait 5 seconds for the next measurement
}
```

---

### **3. Python 스크립트로 CSV 저장**

#### **Dust Sensor Python 코드**
```python
import serial
import csv

SERIAL_PORT = 'COM3'  # Replace with your Arduino port
BAUD_RATE = 9600
CSV_FILE = "dust_sensor_data.csv"

def main():
    with open(CSV_FILE, mode='w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(["Time (s)", "Dust Concentration (ug/m3)", "Air Quality"])  # CSV Header

        try:
            with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1) as ser:
                print("Receiving data from Dust Sensor...")
                while True:
                    line = ser.readline().decode('utf-8').strip()
                    if line and "," in line:
                        print(f"Received: {line}")
                        writer.writerow(line.split(","))
        except serial.SerialException as e:
            print(f"Serial error: {e}")
        except KeyboardInterrupt:
            print("Data collection stopped.")

if __name__ == "__main__":
    main()
```

#### **CM1106 Python 코드**
```python
import serial
import csv

SERIAL_PORT = 'COM4'  # Replace with your Arduino port
BAUD_RATE = 9600
CSV_FILE = "cm1106_data.csv"

def main():
    with open(CSV_FILE, mode='w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(["Time (s)", "CO2 Concentration (ppm)"])  # CSV Header

        try:
            with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1) as ser:
                print("Receiving data from CM1106 Sensor...")
                while True:
                    line = ser.readline().decode('utf-8').strip()
                    if line and "," in line:
                        print(f"Received: {line}")
                        writer.writerow(line.split(","))
        except serial.SerialException as e:
            print(f"Serial error: {e}")
        except KeyboardInterrupt:
            print("Data collection stopped.")

if __name__ == "__main__":
    main()
```

---

### **4. 실행 방법**
1. **Dust 센서**와 **CM1106**에 대해 각각의 Python 스크립트를 실행합니다.
2. 두 센서를 Arduino 보드에 연결하고, 각각 다른 포트(COM 포트)를 사용하여 데이터를 출력합니다.
3. 각각의 Python 파일을 별도로 실행하여 두 센서의 데이터를 개별적으로 저장합니다.

---

### **CSV 파일 예시**
#### **dust_sensor_data.csv**
```csv
Time (s),Dust Concentration (ug/m3),Air Quality
5,25.3,Good
10,50.2,Moderate
15,120.1,Unhealthy
20,170.5,Very Unhealthy
```

#### **cm1106_data.csv**
```csv
Time (s),CO2 Concentration (ppm)
5,400
10,405
15,410
20,420
```

---

### **추가 사항**
- **포트 충돌 방지**: Dust 센서와 CM1106 센서를 각각 다른 COM 포트에 연결해야 합니다.
- **시리얼 모니터 종료**: Arduino IDE의 시리얼 모니터를 닫아야 Python 스크립트에서 포트를 열 수 있습니다.
- **Python 병렬 실행**: 두 Python 스크립트를 각각의 터미널 창에서 실행합니다.

궁금한 점이 있다면 언제든지 말씀해주세요! 😊
