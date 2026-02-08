# 구글 시트 연동 설정 가이드

문의 폼을 구글 시트와 연동하여 모든 문의 내역을 관리할 수 있습니다.

## 1단계: 구글 시트 생성

1. [Google Sheets](https://sheets.google.com)에서 새 스프레드시트 생성
2. 시트 이름을 `AIClab 문의 내역`으로 변경
3. 첫 번째 행(헤더)에 다음 항목 입력:

| A        | B    | C      | D      | E    | F        | G         | H        |
| -------- | ---- | ------ | ------ | ---- | -------- | --------- | -------- |
| 접수일시 | 이름 | 이메일 | 연락처 | 소속 | 문의유형 | 예산/일정 | 상세내용 |

## 2단계: Apps Script 생성

1. 시트 상단 메뉴에서 **확장 프로그램 > Apps Script** 클릭
2. 기존 코드를 모두 지우고 아래 코드를 붙여넣기:

```javascript
function doPost(e) {
  try {
    // 스프레드시트 ID를 현재 시트 ID로 변경
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    // POST 데이터 파싱
    const data = JSON.parse(e.postData.contents);

    // 새 행 추가
    sheet.appendRow([
      data.timestamp,
      data.name,
      data.email,
      data.phone,
      data.company,
      data.type,
      data.budget,
      data.message,
    ]);

    // 이메일 알림 보내기 (선택사항)
    sendEmailNotification(data);

    return ContentService.createTextOutput(
      JSON.stringify({ result: "success" }),
    ).setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(
      JSON.stringify({ result: "error", message: error.toString() }),
    ).setMimeType(ContentService.MimeType.JSON);
  }
}

// 이메일 알림 함수 (선택사항)
function sendEmailNotification(data) {
  const adminEmail = "info@aiclab2020.com"; // 알림 받을 이메일
  const subject = `[AIClab] 새 문의: ${data.type} - ${data.name}`;
  const body = `
새로운 문의가 접수되었습니다.

[문의 정보]
- 접수일시: ${data.timestamp}
- 이름: ${data.name}
- 이메일: ${data.email}
- 연락처: ${data.phone}
- 소속: ${data.company}
- 문의유형: ${data.type}
- 예산/일정: ${data.budget}

[상세 내용]
${data.message}

---
이 메일은 AIClab 웹사이트에서 자동 발송되었습니다.
  `;

  MailApp.sendEmail(adminEmail, subject, body);
}

// 테스트 함수 (GET 요청용)
function doGet(e) {
  return ContentService.createTextOutput(
    "AIClab Contact Form API is running!",
  ).setMimeType(ContentService.MimeType.TEXT);
}
```

3. 파일 이름을 `ContactForm`으로 변경 (왼쪽 상단)
4. **저장** 버튼 클릭 (💾)

## 3단계: 웹 앱으로 배포

1. 상단 **배포 > 새 배포** 클릭
2. 왼쪽 톱니바퀴 아이콘 클릭 > **웹 앱** 선택
3. 설정:
   - **설명**: AIClab Contact Form
   - **실행 사용자**: 나
   - **액세스 권한**: **모든 사용자** (Anyone)
4. **배포** 버튼 클릭
5. 권한 승인 메시지가 나오면 **액세스 허용**
6. **웹 앱 URL** 복사 (예: `https://script.google.com/macros/s/AKfycb.../exec`)

## 4단계: 웹사이트에 URL 적용

1. `contact.html` 파일 열기
2. 스크립트 부분에서 `YOUR_GOOGLE_SCRIPT_URL_HERE`를 복사한 URL로 교체:

```javascript
const GOOGLE_SCRIPT_URL =
  "https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec";
```

3. GitHub에 푸시하여 배포

## 테스트

1. 웹사이트 문의 폼 작성 후 전송
2. 구글 시트에 새 행이 추가되는지 확인
3. 이메일 알림이 오는지 확인 (설정한 경우)

## 문제 해결

### CORS 오류가 발생하는 경우

- Apps Script 배포 설정에서 **액세스 권한**이 "모든 사용자"로 되어있는지 확인

### 데이터가 시트에 저장되지 않는 경우

- Apps Script 실행 로그 확인: **보기 > 실행 로그**
- 스크립트 권한이 제대로 승인되었는지 확인

### 이메일 알림이 오지 않는 경우

- `adminEmail` 주소가 올바른지 확인
- Gmail 스팸함 확인
