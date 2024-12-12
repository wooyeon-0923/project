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
  // Measure Dust Sensor
  duration = pulseIn(dustPin, LOW);
  lowpulseoccupancy += duration;

  if ((millis() - starttime) >= sampletime_ms) {
    // Dust concentration calculation
    ratio = lowpulseoccupancy / (sampletime_ms * 10.0);
    dustConcentration = 1.1 * pow(ratio, 3) - 3.8 * pow(ratio, 2) + 520 * ratio + 0.62; // Unit: ug/m3

    // Reset for next measurement
    lowpulseoccupancy = 0;
    starttime = millis();
  }

  // Measure CO2 Sensor
  int co2Concentration = -1;  // Default value if CO2 measurement fails
  uint8_t ret = cm1106_i2c.measure_result();
  if (ret == 0) {
    co2Concentration = cm1106_i2c.co2;
  }

  // Air Quality Determination
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

  // Print Results (CSV format)
  Serial.print(millis() / 1000); // Time in seconds
  Serial.print(",");
  Serial.print(co2Concentration); // CO2 Concentration
  Serial.print(",");
  Serial.print(dustConcentration); // Dust Concentration
  Serial.print(",");
  Serial.println(airQuality); // Air Quality

  // Small delay to ensure timing consistency
  delay(10);
}
