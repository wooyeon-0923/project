# 💨 Air Quality Measurement and Chatbot Project

## 📖 Project Overview
This project utilizes the **Jetson Nano** and **PM10 API** to measure fine dust concentrations in real time and allows users to easily check air quality status via a chatbot interface.  
By incorporating Gradio UI and OpenAI GPT-4, we provide data-driven suggestions and advice.

---
## Air Pollution Standards in Korean Schools (School Health Act)

For **Particulate Matter (PM10)**, the standard is **75 μg/m³ or lower** in classrooms and cafeterias, and **150 μg/m³ or lower** in gyms and auditoriums.  
For **Fine Particulate Matter (PM2.5)**, it is **35 μg/m³ or lower** in classrooms and cafeterias.  
For **Carbon Dioxide (CO₂)**, it is **1,000 ppm or lower** in classrooms and cafeterias; for mechanical ventilation systems, **1,500 ppm or lower** is allowed.

Source: Standards for environmental hygiene in school facilities (Article 3, Paragraph 1, Item 1, 3, and 3-2 of the Enforcement Rules of the School Health Act)

---
## Impact of Air Pollutants on Learning Efficiency
According to a referenced study, when indoor CO₂ concentration exceeds 1,000 ppm, concentration levels drop, and learning efficiency decreases. If it goes beyond 2,000 ppm, symptoms such as headaches and fatigue occur, having a negative effect on academic performance.  
Additionally, the EPA (United States Environmental Protection Agency) states that poor air quality can reduce concentration, trigger or worsen respiratory diseases, and that students in schools with proper indoor air quality show a **5% increase** in test scores.

---
## Project Direction
**Goal 1**: Measure harmful air pollutants in the classroom using Arduino and Jetson Nano  
**Goal 2**: Create a chatbot that learns about air quality using Function Calling

**Expected Outcome 1**: Improve learning efficiency by purifying the air when the air pollutant levels exceed acceptable standards  
**Expected Outcome 2**: Provide solutions to tackle poor air quality through a chatbot interface

---

## 🎯 Key Features
1. **Comparison of Sensor Data and API Data**  
   - Compare Jetson Nano’s Dust Sensor data with PM10 API data to recommend ventilation or use of air purifiers.

2. **Gradio UI-Based Chatbot**  
   - Analyze real-time data to provide appropriate responses to user queries.

3. **Integration with OpenAI GPT-4**  
   - Leverage the OpenAI GPT-4 API to handle complex questions.

---

## ⚙️ System Configuration

### 🖥️ Hardware Setup

- **Grove Dust Sensor**  
  <img src="https://github.com/user-attachments/assets/eb91cb1b-1dd8-4e63-b716-f5217ab44a32" width="300"/>

- **CM1106 CO2 Sensor**  
  <img src="https://github.com/user-attachments/assets/a93a7a01-6cdf-440f-ad10-5900f2303ccf" width="300"/>

- **Jetson Nano**  
  <img src="https://github.com/user-attachments/assets/7edb789c-5baa-40a0-8204-94ecf3373c9c" width="300"/>

- **Arduino**  
  <img src="https://github.com/user-attachments/assets/0a5712f2-cf6c-4947-a56b-d99ff239aee5" width="300"/>

### 🛠️ Software Setup
- **Python**
  - Data processing and machine learning model building.
- **Arduino**
  - Sensor data acquisition and serial communication.
- **OpenAI API**
  - Implementing a chatbot using Function Calling.

---

## 🛠️ Project Structure

### 1. Technologies Used
- **Jetson Nano**: Processes dust sensor data.  
- **Dust Sensor**: Measures indoor particulate matter (PM) in µg/m³.  
- **PM10 API**: Fetches real-time PM10 data from an external API.  
- **Python Libraries**:
  - `gradio`, `pandas`, `openai`, `urllib`.

### 2. Requirements
- Python 3.8 or higher
- Internet connection (required for PM10 API and GPT-4 API calls)

---

## 💾 Installation & Execution

1. **Set up Python environment**  
   Install the required libraries with the following command:
   ```bash
   pip install gradio pandas openai
   ```

2. **Configure OpenAI API Key**  
   Set the `OPENAI_API_KEY` environment variable or provide it directly in the code:
   ```python
   os.environ['OPENAI_API_KEY'] = 'YOUR_OPENAI_API_KEY'
   ```

3. **Prepare Dust Sensor Data**  
   The dust data collected by Jetson Nano should be stored in CSV format. For example:
   ```csv
   Timestamp,Dust Concentration (ug/m3)
   2024-12-01 10:00:00,35.2
   2024-12-01 11:00:00,40.5
   ```

4. **Run the Code**  
   Execute the script with:
   ```bash
   python app.py
   ```

---

## 💻 Main Code Explanation

### 1. Fetching PM10 API Data
Retrieve data from the PM10 API and convert it into a DataFrame:
```python
url = 'https://apihub.kma.go.kr/api/typ01/url/kma_pm10.php?tm1=202412112020&tm2=202412122020&authKey=8diTRQm4TEKYk0UJuOxCsg'
with urlopen(url) as f:
    html = f.read().decode('euc-kr')

lines = html.split('\n')
filtered_lines = [line for line in lines if ',   132,' in line]

columns = ["TM", "STN", "PM10", "FLAG", " ", "MQC"]
data = []
for line in filtered_lines:
    values = line.split(',')
    data.append([value.strip() for value in values])

df = pd.DataFrame(data, columns=columns[:len(data[0])])
df.drop(['STN', 'FLAG', ' ', 'MQC'], axis=1, inplace=True)
last_API_value = float(df['PM10'].iloc[-1])
```

The preprocessing of the code above (shown in images):  
<img src="https://github.com/user-attachments/assets/4c89c75b-9503-4b9b-af3e-43ec1735f4ca" width="300"/>  
<img src="https://github.com/user-attachments/assets/e9f1818f-cbe1-4d1e-8300-5ba421c75b68" width="300"/>

### 2. Reading Jetson Nano Sensor Data
Reads the latest Dust Sensor data from a CSV file:
```python
df_dust = pd.read_csv('dust_sensor_data.csv')
last_sensor_value = float(df_dust['Dust Concentration (ug/m3)'].iloc[-1])
```

### 3. Integrating OpenAI API and Gradio UI
Through the Gradio UI, users can input questions; the OpenAI API is then called to generate answers:
```python
with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="Chat Window")
    user_textbox = gr.Textbox(label="Input")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```

### 4. Complete Code
Below is the entire Python code.  
At this point, we named the chatbot that suggests solutions based on the collected air quality—especially using Function Calling—**"청정이"** ("Cheongjeongi").  
Main responses of Cheongjeongi:  
- If the external fine dust value (API) is higher than the internal value, it recommends using an air purifier.  
- If the external air quality is normal or lower, it recommends ventilating.  
- If the internal fine dust is higher than the external, it recommends ventilation.  
- If CO₂ levels are over 1000 ppm, it recommends ventilation.

```python
import gradio as gr
import random
import os
from openai import OpenAI
import pandas as pd
from urllib.request import urlopen

# Set OpenAI Key
os.environ['OPENAI_API_KEY'] = 'YOUR_OPENAI_API_KEY'
OpenAI.api_key = os.getenv("OPENAI_API_KEY")

# Retrieve API data
url = 'https://apihub.kma.go.kr/api/typ01/url/kma_pm10.php?tm1=202412112020&tm2=202412122020&authKey=8diTRQm4TEKYk0UJuOxCsg'
with urlopen(url) as f:
    html = f.read().decode('euc-kr')

lines = html.split('\n')
filtered_lines = [line for line in lines if ',   132,' in line]

columns = ["TM", "STN", "PM10", "FLAG", " ", "MQC"]
data = []
for line in filtered_lines:
    values = line.split(',')
    data.append([value.strip() for value in values])

df = pd.DataFrame(data, columns=columns[:len(data[0])])
df.drop(['STN', 'FLAG', ' ', 'MQC'], axis=1, inplace=True)
last_API_value = float(df['PM10'].iloc[-1])

# Retrieve Jetson Nano sensor data
df_dust = pd.read_csv('dust_sensor_data.csv')
last_sensor_value = float(df_dust['Dust Concentration (ug/m3)'].iloc[-1])

# Define OpenAI functions
def get_last_sensor_value():
    return {"last_sensor_value": last_sensor_value}

def get_last_api_value():
    return {"last_api_value": last_API_value}

functions = [
    {
        "name": "get_last_sensor_value",
        "description": "Returns the latest dust concentration value from the Jetson Nano sensor",
        "parameters": {"type": "object", "properties": {}, "required": []}
    },
    {
        "name": "get_last_api_value",
        "description": "Returns the latest PM10 value from the API",
        "parameters": {"type": "object", "properties": {}, "required": []}
    }
]

def process(user_message, chat_history):
    if "대전 날씨" in user_message:
        if last_sensor_value > last_API_value:
            ai_message = (f"현재 젯슨 나노 측정 값({last_sensor_value} ug/m3)가 "
                          f"API PM10 값({last_API_value} ug/m3)보다 높습니다. 환기를 시키세요.")
        else:
            ai_message = (f"현재 젯슨 나노 측정 값({last_sensor_value} ug/m3)가 "
                          f"API PM10 값({last_API_value} ug/m3)보다 낮거나 같습니다. 공기청정기를 사용하세요.")
        chat_history.append((user_message, ai_message))
        return "", chat_history

    client = OpenAI()
    completion = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "system", "content": "You are a helpful assistant."},
                  {"role": "user", "content": user_message}],
        functions=functions,
        function_call="auto"
    )
    response = completion.choices[0].message

    if "function_call" in response:
        func_name = response["function_call"]["name"]
        func_result = (get_last_sensor_value() if func_name == "get_last_sensor_value" else get_last_api_value())
        completion = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "system", "content": "You are a helpful assistant."},
                      {"role": "user", "content": user_message},
                      {"role": "function", "name": func_name, "content": str(func_result)}]
        )
        ai_message = completion.choices[0].message.content
    else:
        ai_message = response["content"]

    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="Chat Window")
    user_textbox = gr.Textbox(label="Input")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```

---
## 💻 Practical Results

https://github.com/user-attachments/assets/8bf758d6-a6f7-4aa3-8641-e2c43969a85d

---

## 📈 Expected Effects
1. **Real-Time Sensor Monitoring**  
   - Check particulate matter and CO₂ levels in real time.

2. **Future Concentration Prediction**  
   - Predict future concentration changes using machine learning models.

3. **User-Friendly Interface**  
   - Provide a conversational way of accessing air quality data and predictions via a chatbot.

---

## 🛠️ Future Improvements
1. **Adding More Sensors**  
   - 1. VOCs Sensor: C304 <img src="https://github.com/user-attachments/assets/335817fc-f9b6-4d26-b1f8-2881952a8221" width="300"/>  
     2. Formaldehyde Sensor: C303 <img src="https://github.com/user-attachments/assets/21af43b0-975f-475a-8853-31ae398cbc13" width="300"/>  
     3. Ozone Sensor: C401 <img src="https://github.com/user-attachments/assets/c550c479-00aa-47cf-bdff-7110d2fd7afc" width="300"/>

   - Collect additional environmental data such as temperature and humidity.

2. **STT/TTS Integration**  
   - Interact with the chatbot via voice.

3. **Integration with IoT Platforms**  
   - Cloud-based data storage and remote monitoring.  
   - Automatically open windows remotely if pollutant levels exceed recommended thresholds.

4. **Applying Machine Learning Techniques**  
   - Build an algorithm that predicts future concentrations based on collected data for proactive measures.

---

## 📢 **Contributors**

- Song Hyun-gon  
- Kwak Won-jun  
- Kim Kang-in  
- Cho Woo-yeon  

---

## review

**Cho Woo-yeon**:  
“As someone deeply involved in orchestrating the end-to-end workflow—from sensor data collection and processing to the final chatbot interface. I gained invaluable insights into comprehensive solution development. This project let me tackle complex challenges such as filtering noisy data, integrating advanced AI models, and ensuring user-friendly communication. I’m especially proud that our platform can monitor air quality while also suggesting practical measures for improving indoor environments. Overall, it’s been a transformative experience, and I’m eager to continue innovating in the sphere of smart, data-driven solutions.”
