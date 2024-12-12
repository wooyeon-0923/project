앞으로의 농도를 예측하려면 **시간 시계열 데이터**를 기반으로 **시계열 분석**이나 **회귀 모델**을 사용하여 미래의 Dust 농도와 CO2 농도를 예측해야 합니다. 이를 위해 ARIMA, LSTM, 또는 Random Forest Regressor 같은 모델을 활용할 수 있습니다.

아래는 시간에 따른 농도를 예측하는 전체적인 단계와 코드 예시입니다.

---

### **1. 데이터 준비**
먼저, `sensor_data.csv`가 시간 시계열 데이터를 포함하도록 준비되어야 합니다.

#### 예시: `sensor_data.csv`
```csv
Time (s),Dust Concentration (ug/m3),CO2 Concentration (ppm)
0,25.3,400
5,30.2,405
10,28.1,410
15,32.0,420
20,40.0,430
...
```

---

### **2. 시간 시계열 분석**
시간별로 수집된 데이터를 기반으로 미래 농도를 예측합니다.

#### **코드 예시: ARIMA 모델 사용**
ARIMA는 시간 시계열 데이터를 분석하고 미래 값을 예측하는 데 적합합니다.

```python
import pandas as pd
from statsmodels.tsa.arima.model import ARIMA
import matplotlib.pyplot as plt

# 데이터 로드
df = pd.read_csv("sensor_data.csv")

# Time을 인덱스로 설정 (필요하면 datetime 형식으로 변환)
df['Time (s)'] = pd.to_datetime(df['Time (s)'], unit='s')  # 초 단위 시간을 datetime으로 변환
df.set_index('Time (s)', inplace=True)

# Dust 농도 예측
dust_series = df['Dust Concentration (ug/m3)']

# ARIMA 모델 생성 및 학습
model = ARIMA(dust_series, order=(5, 1, 0))  # ARIMA(p, d, q)
model_fit = model.fit()

# 미래 10개의 농도 예측
forecast = model_fit.forecast(steps=10)

# 결과 출력
print("Future Dust Concentration Predictions:")
print(forecast)

# 시각화
plt.figure(figsize=(10, 6))
plt.plot(dust_series, label='Observed')
plt.plot(forecast, label='Forecast', color='red')
plt.xlabel('Time')
plt.ylabel('Dust Concentration (ug/m3)')
plt.title('Dust Concentration Forecast')
plt.legend()
plt.show()
```

#### **출력 예시**:
```
Future Dust Concentration Predictions:
2024-12-12 12:10:00    35.0
2024-12-12 12:15:00    36.5
2024-12-12 12:20:00    37.2
...
```

---

### **3. 다변량 회귀 모델**
Dust와 CO2 농도의 상관성을 활용해 다변량 회귀 모델로 미래 값을 예측할 수도 있습니다.

#### **코드 예시: Random Forest Regressor 사용**
```python
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error
import numpy as np

# 독립 변수와 종속 변수 정의
X = df[['Dust Concentration (ug/m3)']].shift(1).dropna()  # 이전 값을 사용
y = df['CO2 Concentration (ppm)'][1:]  # 현재 CO2 농도

# 데이터 분할
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 모델 학습
model = RandomForestRegressor(random_state=42)
model.fit(X_train, y_train)

# 미래 값 예측
future_dust = np.array([30, 35, 40]).reshape(-1, 1)  # 미래 Dust 농도 가정
future_co2 = model.predict(future_dust)

# 결과 출력
for dust, co2 in zip(future_dust.flatten(), future_co2):
    print(f"Future Dust: {dust} ug/m3 -> Predicted CO2: {co2:.2f} ppm")
```

---

### **4. LSTM (딥러닝 기반 시계열 예측)**
LSTM은 장기 의존성을 가진 시계열 데이터를 처리하는 데 유리합니다. 아래는 LSTM을 활용한 예제입니다.

#### **코드 예시: LSTM 모델**
```python
import pandas as pd
import numpy as np
from sklearn.preprocessing import MinMaxScaler
from keras.models import Sequential
from keras.layers import LSTM, Dense

# 데이터 로드 및 전처리
df = pd.read_csv("sensor_data.csv")
scaler = MinMaxScaler()
scaled_data = scaler.fit_transform(df[['Dust Concentration (ug/m3)', 'CO2 Concentration (ppm)']])

# 데이터 준비 (LSTM 입력)
X, y = [], []
time_steps = 5
for i in range(len(scaled_data) - time_steps):
    X.append(scaled_data[i:i + time_steps, 0])  # Dust 농도
    y.append(scaled_data[i + time_steps, 1])   # CO2 농도
X, y = np.array(X), np.array(y)

# LSTM 모델 정의
model = Sequential()
model.add(LSTM(50, return_sequences=True, input_shape=(X.shape[1], 1)))
model.add(LSTM(50, return_sequences=False))
model.add(Dense(25))
model.add(Dense(1))

# 모델 컴파일 및 학습
model.compile(optimizer='adam', loss='mean_squared_error')
model.fit(X, y, batch_size=1, epochs=10)

# 미래 값 예측
future_input = scaled_data[-time_steps:, 0].reshape(1, time_steps, 1)  # 최근 데이터를 기반으로 예측
future_prediction = model.predict(future_input)
future_prediction = scaler.inverse_transform(future_prediction)  # 원래 값으로 변환

print("Predicted Future CO2 Concentration:", future_prediction.flatten()[0])
```

---

### **5. 선택 기준**
- **ARIMA**: 단일 변수의 시간 시계열 분석. Dust 농도나 CO2 농도 각각을 독립적으로 예측.
- **Random Forest Regressor**: 다변량 관계를 고려한 미래 값 예측.
- **LSTM**: 시계열 데이터에 딥러닝을 적용하여 장기적인 미래 값 예측.

---

### **6. 추천 시각화**
미래 예측 값을 기존 데이터와 함께 시각화하면 결과를 직관적으로 확인할 수 있습니다.

```python
plt.plot(df.index, df['Dust Concentration (ug/m3)'], label='Observed')
plt.plot(forecast.index, forecast, label='Forecast', color='red')
plt.xlabel('Time')
plt.ylabel('Dust Concentration (ug/m3)')
plt.legend()
plt.show()
```

---

### **결론**
1. Dust 농도와 CO2 농도의 미래 값을 예측하려면 시계열 분석이나 회귀 모델을 사용하세요.
2. 데이터의 성격에 따라 ARIMA, Random Forest, 또는 LSTM을 선택합니다.
3. 예측 결과를 시각화하거나 CSV로 저장하여 활용성을 높입니다.

필요하면 특정 모델 구현에 대해 더 도와드릴게요! 😊
