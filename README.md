int pin = 8;
unsigned long duration;
unsigned long starttime;
unsigned long sampletime_ms = 30000;  // 30초 동안 샘플링
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

    if ((millis()-starttime) > sampletime_ms)  // 30초마다 측정
    {
        ratio = lowpulseoccupancy/(sampletime_ms*10.0);
        concentration = 1.1*pow(ratio,3)-3.8*pow(ratio,2)+520*ratio+0.62; // ug/m3 단위

        Serial.println("==============================");
        Serial.print("Dust concentration: ");
        Serial.print(concentration);
        Serial.println(" ug/m3");

        // 대기질 상태 표시
        Serial.print("Air quality: ");
        if(concentration <= 30) {
            Serial.println("Good");
        }
        else if(concentration <= 80) {
            Serial.println("Moderate");
        }
        else if(concentration <= 150) {
            Serial.println("Unhealthy");
        }
        else {
            Serial.println("Very Unhealthy");
        }

        Serial.println("------------------------------");
        Serial.print("Measurement time: ");
        Serial.print(millis()/1000);
        Serial.println(" seconds");
        Serial.println("==============================\n");

        // 다음 측정을 위한 초기화
        lowpulseoccupancy = 0;
        starttime = millis();
    }
}
