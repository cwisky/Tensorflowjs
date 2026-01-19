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
* 홀수/짝수 분류모델 생성

## Tensorflow 모델을 Javascript기반에서 실행할 수 있도록 변환하기
* 파이썬 기반에서 Tensorflowjs 모듈을 사용하여 모델 변환
* 변환된 모델 구성파일 확인

## 크롬 확장프로그램에 변환된 모델 추가하기
* model.json, BIN 파일 내용을 Javascript코드에 포함
* Javascript기반에서 사용되는 tensorflow.js 라이브러리 사용하여 모델 실행
* 브라우저 화면에 분류 결과 표시
