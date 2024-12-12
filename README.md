import pandas as pd
from urllib.request import urlopen

# URL 설정
url = 'https://apihub.kma.go.kr/api/typ01/url/kma_pm10.php?tm1=202412112020&tm2=202412122020&authKey=8diTRQm4TEKYk0UJuOxCsg'

# URL 호출 및 데이터 읽기
with urlopen(url) as f:
    html = f.read().decode('euc-kr')  # euc-kr로 디코딩 (한글 포함 데이터 처리)

# 데이터를 행 단위로 분리
lines = html.split('\n')

# stn이 132인 행만 필터링
filtered_lines = [line for line in lines if ',   132,' in line]

# DataFrame으로 변환
# 각 행의 데이터를 쉼표로 분리하여 DataFrame으로 변환
columns = ["TM", "STN", "PM10", "FLAG", " ", "MQC"]  # 컬럼명 추가

data = []
for line in filtered_lines:
    values = line.split(',')
    data.append([value.strip() for value in values])  # 공백 제거

# DataFrame 생성
df = pd.DataFrame(data, columns=columns[:len(data[0])])  # 데이터 길이에 맞게 컬럼 조정

# 불필요한 열 삭제
df.drop(['STN', 'FLAG', ' ', 'MQC'], axis=1, inplace=True)

# PM10 열의 마지막 값 출력
last_pm10_value = df['PM10'].iloc[-1]
# print("PM10 열의 마지막 값:", last_pm10_value)

# # DataFrame 출력
# print(df)

# 필요 시 CSV로 저장 가능
# df.to_csv('filtered_stn_132.csv', index=False)
