# Tensorflowjs
* Python 기반에서 생성된 머신러닝 모델을 Javascript기반에서 실행할 수 있도록 변환해주는 파이썬 모듈
* 변환된 모델은 model.JSON, BIN 파일을 포함
* Javascript에서는 Tensorflow.js 라이브러리를 사용하여 model.json을 로드하고 사용함
* 크롬 확장프로그램에서 실행되는 Javascript에 model.json, BIN을 포함시키면 브라우저 기반에서 모델을 사용할 수 있음

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

* 홀수/짝수 분류모델 생성
```python
# simple_odd_even.py
import numpy as np
import tensorflow as tf
from tensorflow import keras

# 데이터 생성
X = np.arange(0, 1001).reshape(-1, 1).astype(np.float32)
y = (X.flatten() % 2).astype(np.float32)

# 모델 정의
model = keras.Sequential([
    keras.layers.Dense(16, activation='relu', input_shape=(1,), name='hidden_layer_1'),
    keras.layers.Dense(8, activation='relu', name='hidden_layer_2'),
    keras.layers.Dense(1, activation='sigmoid', name='output_layer')
])

# 컴파일
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])

# 학습
model.fit(X, y, epochs=50, batch_size=32, validation_split=0.2, verbose=1)

# 저장
model.save('odd_even_model.keras')
print("\n모델 저장 완료: odd_even_model.keras")

model.save('odd_even_model.h5')
print("✓ H5 모델 저장: odd_even_model.h5")

# 가중치만 별도로 저장 (가장 권장 - 구조적 오류 회피용)
model.save_weights('odd_even_weights.weights.h5') 
print("가중치 저장 완료: odd_even_weights.weights.h5")

# 테스트
test_numbers = [10, 11, 100, 101]
for num in test_numbers:
    pred = model.predict(np.array([[num]]), verbose=0)[0][0]
    result = "홀수" if pred > 0.5 else "짝수"
    print(f"{num} → {result}")
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
    tf.keras.layers.Dense(16, activation='relu', input_shape=(1,), name='hidden_layer_1'),
    tf.keras.layers.Dense(8, activation='relu', name='hidden_layer_2'),
    tf.keras.layers.Dense(1, activation='sigmoid', name='output_layer')
])

try:
    # 3. 가중치 파일 로드 (.weights.h5 파일을 사용하면 메타데이터 충돌이 없습니다)
    # 만약 odd_even_weights.weights.h5가 없다면 odd_even_model.h5를 넣어보세요.
    try:
        new_model.load_weights('odd_even_weights.weights.h5')
        print("성공: 전용 가중치 파일에서 로드 완료")
    except:
        new_model.load_weights('odd_even_model.h5')
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
* model.json, BIN 파일 내용을 Javascript코드에 포함
* Javascript기반에서 사용되는 tensorflow.js 라이브러리 사용하여 모델 실행
* 브라우저 화면에 분류 결과 표시
