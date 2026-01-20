# Tensorflowjs/Tensorflow.js

## Tensorflowjs
* Python 모듈이며 Tensorflow 모델을 Javascript에서 사용할 수 있는 모델로 변환
* 변환된 모델은 model.json, group1-shard1of1.bin 파일을 포함

## Tensorflow.js
* Javascritp 라이브러리리이며 자바스크립트 버전으로 변환된 Tensorflow 모델을 사용하는 라이브러리
* 크롬 확장프로그램에서 실행되는 Javascript에 model.json, group1-shard1of1.bin을 포함시키면 브라우저 기반에서 모델을 사용할 수 있음

## 크롬 브라우저에서 Hello World 확장프로그램 (Chrome Extension) 작성하기
### 1. 폴더 및 파일 생성
```text
hello-world-extension/
├── manifest.json
├── content.js
└── icon.png (선택사항)
```
#### 1-1. manifest.json
```json
{
  "manifest_version": 3,
  "name": "Hello World Extension",
  "version": "1.0.0",
  "description": "My first Chrome extension",
  
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ]
}
```
#### 1-2. contents.js
```javascript
// 페이지 로드 시 "Hello World" 배너 표시
(function() {
    console.log('Hello World Extension loaded!');
    
    // Hello World 메시지 박스 생성
    const helloBox = document.createElement('div');
    helloBox.textContent = 'Hello World!';
    helloBox.style.cssText = `
        position: fixed;
        top: 20px;
        right: 20px;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        padding: 20px 30px;
        border-radius: 10px;
        font-size: 24px;
        font-weight: bold;
        font-family: Arial, sans-serif;
        z-index: 999999;
        box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
        cursor: pointer;
        animation: slideIn 0.5s ease-out;
    `;
    
    // 애니메이션 추가
    const style = document.createElement('style');
    style.textContent = `
        @keyframes slideIn {
            from {
                transform: translateX(400px);
                opacity: 0;
            }
            to {
                transform: translateX(0);
                opacity: 1;
            }
        }
    `;
    document.head.appendChild(style);
    
    // 클릭하면 사라지게
    helloBox.addEventListener('click', () => {
        helloBox.remove();
    });
    
    // 페이지에 추가
    document.body.appendChild(helloBox);
    
    // 5초 후 자동으로 사라지게 (선택사항)
    setTimeout(() => {
        if (helloBox.parentNode) {
            helloBox.style.animation = 'slideIn 0.5s ease-out reverse';
            setTimeout(() => helloBox.remove(), 500);
        }
    }, 5000);
})();
```
### 2. Chrome에 설치하기
#### 1. Chrome 열기
#### 2. 확장 프로그램 페이지로 이동
* 주소창에 chrome://extensions/ 입력
* 또는 메뉴(3점 아이콘) -> 도구 더보기 -> 확장 프로그램
#### 3. 개발자 모드 활성화
* 우측 상다의 "개발자 모드" 토글 ON
#### 4. 확장 프로그램 로드
* [압축해제된 확장프로그램 로드] 버튼 클릭
* hello-world-extension 폴더 선택
#### 5. 완료
* "확장 프로그램 로드됨" 메시지가 브라우저 하단에 잠시 표시됨
* 확장 프로그램이 목록에 표시됩니다

### 3. 테스트하기
#### 1. 아무 웹사이트나 방문한다(예: google.com)
#### 2. 페이지 우측 상단에 "Hello World!" 메시지가 나타나는지 확인
#### 3. 메시지를 클릭하면 사라짐

## Python기반에서 Tensorflow 모델 생성
* 파이썬 가상환경 생성
```bash
# 1. 새로운 가상환경 생성 (파이썬 3.11 지정)
conda create -n tfjs_fix python=3.11 -y

# 2. 가상환경 활성화
conda activate tfjs_fix

# 3. 필수 패키지만 깔끔하게 설치
pip install tensorflow==2.15.0 tensorflowjs==4.17.0
```
* 문제의 원인인 Decision Forests 모듈 제거
```bash
# 1. 활성화된 가상환경인지 확인 (tfjs_fix)
# 2. 오류를 일으키는 Decision Forests 모듈 제거
pip uninstall tensorflow-decision-forests -y
```

* y = 2x + 1 직선의 방정식에 노이즈를 추가한 데이터 회귀모델 생성
```python
import numpy as np
import tensorflow as tf
from tensorflow import keras

# 1. 데이터 생성 (y = 2x + 1)
# -10부터 10까지 100개의 데이터를 생성합니다.
X = np.linspace(-10, 10, 100)
# 실제 수식에 아주 약간의 무작위 노이즈를 더해 학습 효율을 높입니다.
y = 2 * X + 1 + np.random.normal(0, 0.1, X.shape)

# 2. 모델 정의
model = keras.Sequential([
    # 입력 1개, 출력 1개인 가장 단순한 선형 레이어
    keras.layers.Dense(units=1, input_shape=(1,), name='linear_layer'),
    keras.layers.Dense(units=1, name='output_layer')
])

# 3. 컴파일 및 학습
# 선형 회귀에는 'adam' 옵티마이저와 'mse' 손실함수가 가장 적합합니다.
model.compile(optimizer=keras.optimizers.Adam(0.1), loss='mse')
print("학습 시작...")
model.fit(X, y, epochs=200, verbose=0)

# 4. 검증 (입력 10일 때 결과는 약 21이 나와야 함)
test_val = np.array([[10.0]])
prediction = model.predict(test_val)
print(f"예측 결과: 입력 10.0 -> 출력 {prediction[0][0]:.4f} (정답: 21.0)")

# 5. 세 가지 방식으로 저장 (확장 프로그램 변환 대비)
model.save('linear_reg_model.keras')          # Keras V3
model.save('linear_reg_model.h5')             # Legacy H5
model.save_weights('linear_reg_weights.weights.h5') # 가중치 전용

print("저장 완료: linear_reg_model.keras, .h5, .weights.h5")
```

## Tensorflow 모델을 Javascript기반에서 실행할 수 있도록 변환하기
* 파이썬 기반에서 Tensorflowjs 모듈을 사용하여 모델 변환
  + 아래의 툴은 버전 차이로 오류를 발생할 경우가 많으므로 아래의 변환용 코드를 사용하는 것을 고려해야 함
```python
# tensorflowjs_converter --input_format=keras [모델경로] [저장될디렉토리]
tensorflowjs_converter --input_format=keras ./odd_even_model.keras ./tfjs_model
```
* 변환용 코드를 사용하여 변환하기(*.keras, *.h5 모두 생성해서 테스트한다 (h5 포맷이 호환성이 더 높음))
* 아래의 코드에서 사용된 가중치만 로드하는 방식은 파일의 구조에 의존하지 않으므로 호환성이 더 높음
```python
import sys
from types import ModuleType

# 1. Windows 호환성 가짜 모듈 주입
dummy_tfdf = ModuleType('tensorflow_decision_forests')
sys.modules['tensorflow_decision_forests'] = dummy_tfdf
sys.modules['tensorflow_decision_forests.keras'] = ModuleType('keras')

import tensorflow as tf
import tensorflowjs as tfjs

print("최종 해결책: load_weights 방식 시작")

# 2. 새 모델 정의 (학습 시 모델 구조와 정확히 동일해야 함)
new_model = tf.keras.Sequential([
    # 입력 1개, 출력 1개인 가장 단순한 선형 레이어
    tf.keras.layers.Dense(units=1, input_shape=(1,), name='linear_layer'),
    tf.keras.layers.Dense(units=1, name='output_layer')
])

try:
    # 3. 가중치 파일 로드 (.weights.h5 파일을 사용하면 메타데이터 충돌이 없습니다)
    # 만약 odd_even_weights.weights.h5가 없다면 odd_even_model.h5를 넣어보세요.
    try:
        new_model.load_weights('linear_reg_weights.weights.h5')  # linear_reg_weights.weights.h5
        print("성공: 전용 가중치 파일에서 로드 완료")
    except:
        new_model.load_weights('linear_reg_model.h5')
        print("성공: 전체 모델 파일에서 가중치 로드 완료")

    # 4. TFJS 변환
    tfjs.converters.save_keras_model(new_model, 'tfjs_model')
    print("\n" + "="*50)
    print("변환 성공! ./tfjs_model 폴더를 확인하세요.")
    print("="*50)

except Exception as e:
    print(f"오류 발생: {e}")
```

* 변환된 모델 구성파일 확인 (위의 명령에서 사용한 tfjs_model 디렉토리 확인)
```text
model.json: 모델의 구조(Layer, 가중치 연결 등)가 정의된 파일.
group1-shard1of1.bin: 실제 학습된 가중치(Weight) 데이터가 담긴 이진 파일
```

## 크롬 확장프로그램에 변환된 모델 추가하기
* model.json, group1-shard1of1.bin 파일 내용을 Javascript코드에 포함
* Javascript기반에서 사용되는 tensorflow.js 라이브러리 사용하여 모델 실행
* 브라우저 화면에 분류 결과 표시
### 1단계: 확장 프로그램 폴더 구조 만들기
* my_extension/
  + manifest.json (확장 프로그램 설정)
  + popup.html (사용자 화면)
  + popup.js (모델 로드 및 로직)
  + tf.min.js (TensorFlow.js 라이브러리)
  + tf-core.min.js
  + tf-backend-cpu.min.js
  + tf-layers.min.js
  + model/ (변환된 폴더)
    - model.json
    - group1-shard1of1.bin
### 2단계: TensorFlow.js 라이브러리 다운로드
* TensorFlow.js 공식 GitHub에서 tf.min.js 등을 다운로드하여 폴더에 저장(웹 검색)

### 3단계: manifest.json 작성
* 확장프로그램의 정보 정의 (web_accessible_resources 설정으로 브라우저가 .bin가중치 파일에 접근 가능함)
```json
{
  "manifest_version": 3,
  "name": "y = 2x + 1 회귀 모델",
  "version": "1.0",
  "action": { "default_popup": "popup.html" },
  "web_accessible_resources": [
    {
      "resources": ["model/*", "*.wasm"],
      "matches": ["<all_urls>"]
    }
  ]
}
```
### 4단계: popup.html 작성
* 숫자를 입력하여 확장 프로그램에 입력할 웹 페이지
* https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@2.0.0/dist/tf.min.js 에 접속하여 tf.min.js 파일을 다운로드하거나 아래처럼...
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>y = 2x + 1 회귀 모델</title>
</head>
<body style="width: 200px; padding: 10px;">
  <h3>y = 2x + 1 회귀 모델</h3>
  <input type="number" id="numberInput" placeholder="숫자 입력">
  <button id="predictBtn">예측하기</button>
  <p id="resultText">모델 로딩 중...</p>

    <!-- popup.html -->
    <script src="tf-core.min.js"></script>
    <script src="tf-backend-cpu.min.js"></script>
    <script src="tf-layers.min.js"></script>
    <script src="popup.js"></script>

</body>
</html>
```

### 5단계: popup.js 작성
* 모델을 불러오고 예측을 실행하는 핵심 로직
```javascript
// popup.js

// 전역 객체 확보 (tf-core 로드 확인)
const tfEngine = window.tf;

async function loadModel() {
  try {
    // 1. CPU 백엔드 명시적 설정
    await tfEngine.setBackend('cpu');
    console.log("CPU Backend 활성화");

    // 2. 모델 로드
    const modelUrl = chrome.runtime.getURL('model/model.json');
    
    // tf-layers가 로드되면 tf.loadLayersModel이 활성화됩니다.
    model = await tfEngine.loadLayersModel(modelUrl);
    
    document.getElementById('resultText').innerText = "AI 준비 완료!";
  } catch (error) {
    document.getElementById('resultText').innerText = "로드 실패: " + error.message;
    console.error("Detail Error:", error);
  }
}

// 예측 함수 내에서도 tfEngine(또는 window.tf)을 사용하세요.
async function predict() {
  const inputVal = document.getElementById('numberInput').value;
  if (!inputVal || !model) return;

  const num = parseFloat(inputVal);
  const inputTensor = tfEngine.tensor2d([[num]]);
  const prediction = model.predict(inputTensor);
  const y_val = (await prediction.data())[0];

  document.getElementById('resultText').innerText = `예측 결과: ${y_val}`;
}

document.getElementById('predictBtn').addEventListener('click', predict);
loadModel();
```
### 6단계: 크롬에 설치하기
#### 1. 크롬 브라우저 주소창에 chrome://extensions/를 입력하여 접속.
#### 2. 오른쪽 상단의 '개발자 모드'를 켭니다.
#### 3. 왼쪽 상단의 '압축해제된 확장 프로그램을 로드' 버튼을 클릭.
#### 4. 작업한 my_extension 폴더를 선택.
#### 5. 크롬의 일반 웹브라우저 창으로 나가서 주소창 옆 퍼즐 아이콘 -> 방금 등록한 확장 프로그램 클릭 -> 고정핀 아이콘 클릭(항상 보이게 됨) -> 누르면 확장 프로그램이 실행
#### 5. 확장 프로그램 아이콘을 클릭하여 숫자를 입력하고 결과를 확인

## 3. 크롬 브라우저에서 DevTools Extension 생성하기
* 일반 확장 프로그램과 달리 크롬의 개발자 도구를 사용해야 함
* 웹 응답 데이터에 대한 완전한 접근 허용
  + ✔ 압축 해제된 실제 응답 본문
  + ✔ JS / HTML / JSON / Text
  + ✔ 악성코드 분석에 필요한 원본 데이터
* 보안 분석 도구, 기업용 검사 시스템은 거의 전부 이 구조를 사용

### 3-1 DevTools Extension 구조 개요
```markdown
Chrome
 └─ DevTools
     └─ Network Panel
         └─ devtools_page (확장)
             └─ JS로 네트워크 요청/응답 접근
```
