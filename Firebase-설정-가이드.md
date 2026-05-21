# Firebase 설정 가이드 — ID 로그인 / 클라우드 저장

NEON HOLDEM v55 — ID+PIN 클라우드 동기화를 켜는 방법

이 가이드를 따라 **5~10분**이면 끝납니다. 무료입니다 (Firebase Spark 플랜).

---

## 왜 필요한가

지금 게임은 진행이 **브라우저 localStorage**에만 저장돼요. 그래서:
- 폰에서 한 진행 ↔ PC에서 안 이어짐
- 브라우저 캐시 지우면 날아감

Firebase를 연결하면 **ID + PIN으로 로그인**해서 어느 기기에서든 같은 진행을 이어갈 수 있어요.

---

## 1단계 — Firebase 프로젝트 만들기

1. https://console.firebase.google.com 접속 (구글 계정 로그인)
2. **"프로젝트 추가"** 클릭
3. 프로젝트 이름: `neon-holdem` (아무거나 OK)
4. Google 애널리틱스 — **사용 안 함** 선택 (필요 없음)
5. **"프로젝트 만들기"** → 1분 대기

---

## 2단계 — 웹 앱 등록 + 설정값 복사

1. 프로젝트 대시보드에서 **`</>`** (웹) 아이콘 클릭
2. 앱 닉네임: `neon-holdem-web` 입력 → **"앱 등록"**
3. 다음 화면에 나오는 `firebaseConfig` 코드 블록을 **복사**:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...............",
  authDomain: "neon-holdem.firebaseapp.com",
  projectId: "neon-holdem",
  storageBucket: "neon-holdem.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

---

## 3단계 — Firestore 데이터베이스 만들기

1. 왼쪽 메뉴 → **"빌드" → "Firestore Database"**
2. **"데이터베이스 만들기"** 클릭
3. 위치: `asia-northeast3 (서울)` 선택 (한국이면 가장 빠름)
4. 보안 규칙 모드: **"테스트 모드로 시작"** 선택 → 만들기

### ⚠️ 보안 규칙 설정 (중요)

테스트 모드는 30일 후 막혀요. **규칙** 탭에서 아래로 교체하세요:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 로그인(익명auth)된 사용자만 careers 읽기/쓰기 가능
    match /careers/{playerId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

**"게시"** 클릭.

> 참고: 이 규칙은 "로그인한 사람은 누구나 careers를 읽고 쓸 수 있음"이에요. PIN은 클라이언트에서 검증합니다. 캐주얼 게임에는 충분하지만, 민감 데이터는 저장하지 마세요 (게임 진행만 저장됨).

---

## 4단계 — 익명 인증 켜기

1. 왼쪽 메뉴 → **"빌드" → "Authentication"**
2. **"시작하기"** 클릭
3. **"Sign-in method"** 탭 → **"익명"** 선택 → **사용 설정** → 저장

> 익명 인증은 Firestore 접근 권한용입니다. 실제 로그인은 게임 안의 ID+PIN으로 합니다.

---

## 5단계 — 게임에 설정값 넣기

1. `index.html` 파일을 텍스트 에디터로 엽니다
2. `FIREBASE_CONFIG` 를 검색 (Ctrl+F)
3. 빈 칸에 2단계에서 복사한 값을 채웁니다:

**찾을 부분 (현재 — 비어있음):**
```js
const FIREBASE_CONFIG = {
  apiKey: "",
  authDomain: "",
  projectId: "",
  storageBucket: "",
  messagingSenderId: "",
  appId: ""
};
```

**이렇게 채우기 (본인 값으로):**
```js
const FIREBASE_CONFIG = {
  apiKey: "AIzaSy...............",
  authDomain: "neon-holdem.firebaseapp.com",
  projectId: "neon-holdem",
  storageBucket: "neon-holdem.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

4. 저장 → GitHub Desktop에서 **Commit + Push**
5. 1~2분 후 사이트 새로고침

---

## 6단계 — 게임에서 로그인

1. 로비 → 메뉴 → **🔑 "ID 로그인 / 계정 만들기"**
2. **"새 계정"** 탭:
   - ID: 영문/숫자 3~20자 (예: `rlawnaud3`)
   - PIN: 숫자 4~6자리 (예: `1234`)
   - **"계정 만들기"** → 현재 진행이 클라우드에 올라감
3. 다른 기기에서:
   - **"로그인"** 탭 → 같은 ID + PIN → 진행 이어받기

---

## 동작 방식

| 상태 | 저장 |
|---|---|
| 게스트 (로그인 X) | 로컬(브라우저)만 |
| ID 로그인 | 로컬 + **클라우드 자동 동기화** (5초 디바운스) |

- 로그인하면 진행할 때마다 자동으로 클라우드 저장
- 메뉴 → "수동 클라우드 저장"으로 즉시 저장도 가능
- 로그아웃하면 클라우드 진행은 남고, 이 기기는 로컬 모드로

---

## 주의사항

- **PIN을 잊으면 복구 불가** — 꼭 기억하세요 (PIN은 해시로만 저장돼서 서버에서도 못 봄)
- ID는 **소문자로 통일** 저장됨 (대소문자 구분 안 함)
- 무료 플랜(Spark): 하루 읽기 5만 / 쓰기 2만 — 개인 플레이엔 차고 넘침
- 클라우드 vs 로컬 충돌 시: **핸드 수가 더 많은 쪽**을 자동 선택

---

## 문제 해결

| 증상 | 해결 |
|---|---|
| "Firebase 설정 필요" | 5단계 config 값 확인, Push 했는지 |
| "Firebase 연결 중..." 안 끝남 | 익명 인증(4단계) 켰는지 확인 |
| "PIN이 틀렸습니다" | PIN 재확인 (숫자만) |
| 저장 안 됨 | Firestore 보안 규칙(3단계) 게시했는지 |
| 콘솔 빨간 에러 | F12 → Console에서 메시지 확인 → 알려주세요 |

---

*v55 · ID+PIN 클라우드 로그인*
