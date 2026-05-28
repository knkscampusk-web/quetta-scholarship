[README.md](https://github.com/user-attachments/files/28329804/README.md)
# QUETTA 장학 관리 시스템

강남대성 QUETTA 학원의 **월별 모의고사 장학생 선발 자동화** 시스템.
기존 Google Sheets 수동 작업을 대체하는 단일 HTML 웹앱.

🔗 **운영 URL**: https://knkscampusk-web.github.io/quetta-scholarship/

---

## 배포 방법 (간단)

이 레포는 **GitHub Pages**로 자동 배포됩니다. 별도 빌드 도구나 터미널 작업 없이 깃허브 웹에서 모든 작업이 끝납니다.

### 새 버전 올리기 (가장 흔한 작업)
1. 이 레포 메인 페이지에서 **Add file → Upload files** 클릭
2. 새 `index.html` 드래그앤드롭 (같은 이름이라 자동 덮어쓰기됨)
3. 페이지 아래 **Commit changes** 버튼 클릭
4. 1-2분 기다리기 — `Actions` 탭에서 "pages build and deployment" 워크플로우가 ✅ 초록 체크되면 사이트에 반영됨

> 💡 GitHub Pages 재배포는 Firestore 데이터에 영향이 없습니다. 자유롭게 iterate 가능.

### 이전 버전으로 되돌리기
1. 레포 페이지에서 `index.html` 클릭
2. 오른쪽 위 **History** 버튼
3. 되돌리고 싶은 시점의 커밋 클릭
4. 그 시점의 파일 내용 복사해서 다시 업로드 → commit

---

## ⚠️ 보안 주의 — 반드시 읽어보세요

### 1. 학생 개인정보 (가장 중요)
- **학생 명단/성적 엑셀 파일은 절대 이 저장소에 커밋하지 마세요.**
- `.gitignore`에 `*.xlsx`, `*.xls`, `*.csv` 가 등록되어 있어 기본적으로 무시됩니다.
- 테스트용 파일을 두려면 별도 폴더에 두고 그 폴더도 `.gitignore`에 추가하세요.

### 2. Firebase API 키
- `index.html` 안에 Firebase Web API 키가 하드코딩되어 있습니다.
- **이 키 자체로는 데이터에 접근할 수 없습니다** — Firebase Web API 키는 원래 클라이언트(브라우저)에 노출되도록 설계됨.
- 실질 보안은 **Firestore Security Rules**와 **Firebase Authentication**이 담당.
- GitHub Secret Scanning이 자동 경고할 수 있지만, 일반 Firebase Web 키는 무시 OK.

### 3. Firestore Security Rules
Firebase Console → Firestore → 규칙 탭에서 다음과 같이 설정되어 있어야 합니다:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```
**`allow read, write: if true` 같은 설정은 절대 사용 금지** — 누구나 데이터를 읽고 쓸 수 있게 됩니다.

### 4. Firebase Authorized Domains
Firebase Console → Authentication → 설정 → 승인된 도메인 에 다음만 등록되어 있어야 합니다:
- `localhost` (개발용)
- `knkscampusk-web.github.io` (운영)

불필요한 도메인이 있다면 제거.

### 5. 로그인 계정
- 이메일: `campusk@dshw.co.kr` (코드에 고정)
- 비밀번호: Firebase Console → Authentication 에서 관리, 정기 변경 권장

---

## 기술 스택

- **프론트엔드**: 단일 HTML 파일 (Vanilla JS)
- **인증**: Firebase Authentication (Email/Password)
- **데이터베이스**: Firestore
- **엑셀 파싱**: SheetJS (xlsx.full.min.js) — CDN
- **호스팅**: GitHub Pages (이 레포에서 자동 배포)

---

## 파일 구조

```
.
├── index.html              # 메인 앱 (단일 파일, 모든 로직 포함)
├── README.md               # 이 문서
└── .gitignore              # 깃 제외 규칙 (학생 데이터 차단)
```

---

## 주요 기능

### 자동산출 (엑셀 업로드 시 자동 계산)
| 장학명 | 조건 | 혜택 |
|---|---|---|
| 다이닝 장학 | 전체 환산표점 1위 | 식비 100% |
| 다이닝 하프 장학 | 전체 환산표점 2위 | 식비 50% |
| 퀀텀 장학 | 전체 환산표점 3위 | 독서실비 100% |
| 수학반 1등 | 반별 환산표점 1위 | 독서실비 100% |
| 수학백분위 최고향상 | 수학 백분위 최대 향상자 | 독서실비 100% |
| 백분위총합 최고향상 | 4과목 백분위 합 최대 향상자 | 독서실비 100% |

**환산표점** = 국어 표준 + 수학 표준 + 탐1 표준 + 탐2 표준 − 영어 감점

**영어 감점표**
| 등급 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|
| 감점 | 0 | 0.5 | 2 | 4 | 6 | 8 | 10 | 12 | 14 |

**동점 처리**: 수학표준 → 국어표준 → 탐구표준 순 우선

### 수기입력 활성 항목
- 대성학원 전체 1등 (인문/자연) — 교습비 100%
- 성적우수자 (인문/자연) — 매점상품권 5만원

---

## 탭 구조

| 탭 | 기능 |
|---|---|
| 자동산출 | 엑셀 업로드 → 자동 장학 계산 + 이전달 비교 |
| 수기입력 | 학생 검색/필터/다중선택 → 장학 종류별 수동 추가 |
| 최종명단 | 자동+수기 통합, 제외(줄긋기), 기존장학 표시, 이름 클릭 시 상세 모달, 엑셀 다운로드 |
| 전체명단 | 월별 전체 학생 성적 + 이전달 대비 향상 점수 + 영어감점 + 기존장학 |
| 통계&누적 | 반별 환산 평균(좌)·그래프(우) 좌우 배치, 빌보드/장학종류별 셀 클릭 시 학생 모달, 학생별 수상이력 정렬 가능 |
| 설정 | 장학 이름·혜택 수정, Firestore 영구 저장 |

---

## 데이터 모델 (Firestore)

```
quetta_data / {YYYY-MM} / {
  label: "2026년 5월",
  시험종류: "더프",
  winners: [
    { lbl, bc, 혜택, 반, 번호, 이름, 계열, val, type, 기존장학, excluded, 비고 },
    ...
  ],
  classStats: [...],
  overallAvg: 246.3,
  manual: { dasung_in: [...], honor_in: [...], ... },
  excluded: ["이름::장학명", ...],
  remarks: { "이름::장학명": "비고...", ... },
  allStudents: [ /* parseScores 출력물 전체 */ ],
  savedAt: "2026-05-28T01:00:00Z"
}

quetta_settings / sconf / {
  auto: { d1: {lbl,혜택,기준}, ... },
  manual: { dasung_in: {lbl,혜택}, ... }
}
```

### 학생 객체 (allStudents 항목)
```js
{
  반, 번호, 이름, 계열,
  국어표준, 국어백분, 수학표준, 수학백분,
  영어등급, 영어결시,  // 영어결시: 영어 점수 빈 값/0일 때 true
  탐1표준, 탐1백분, 탐2표준, 탐2백분,
  환산, 백분위합
}
```

---

## 평균 계산 기준

통계 탭의 "반별 환산표점 평균"은 **국·수·탐1·탐2 표준점수가 모두 0보다 크고 영어 결시가 아닌 학생만** 집계합니다.

기존 저장본은 영어 결시 플래그가 없어서 결시자가 포함될 수 있습니다. 정확한 평균이 필요하면 해당 월의 성적 파일을 자동산출 탭에서 다시 업로드 후 "자동산출 저장"을 누르세요. Firestore에 새 기준으로 갱신됩니다.

---

## 엑셀 파싱 구조 (참고)

성적 파일 컬럼 위치 (0-based index):

| 항목 | 열 | 엑셀 열명 |
|---|---|---|
| 반 | 2 | C |
| 번호(학번) | 3 | D |
| 이름 | 4 | E |
| 계열(인문/자연) | 5 | F |
| 국어 표준점수 | 12 | M |
| 국어 백분위 | 13 | N |
| 수학 표준점수 | 26 | AA |
| 수학 백분위 | 27 | AB |
| 영어 등급 | 36 | AK |
| 탐1 표준점수 | 55 | BD |
| 탐1 백분위 | 56 | BE |
| 탐2 표준점수 | 66 | BO |
| 탐2 백분위 | 67 | BP |

- 시트 이름 우선순위: "성적" 시트 > 첫 번째 시트
- 데이터 시작행 자동 탐지 (이름 + 학번 모두 유효한 첫 행)

### 기존장학(직방계) 파일 컬럼:
| 항목 | 열 |
|---|---|
| 장학명 | 3 |
| 반 | 8 |
| 학번 | 9 (매칭 기준) |
| 이름 | 10 |

---

## 트러블슈팅

| 증상 | 원인 / 해결 |
|---|---|
| 엑셀 업로드 후 학생 0명 | 시트 이름이 "성적"이 아니거나, 데이터 시작행이 4행이 아님. 엑셀 형식 확인 |
| 향상 장학 산출 안됨 | 이전달 성적 파일 미업로드 또는 학번 매칭 실패 |
| 저장본 안 보임 | Firestore Rules 확인, 로그인 상태 확인 |
| 배포 안 됨 | Actions 탭에서 "pages build and deployment" 가 ❌ 빨간색이면 클릭해서 에러 확인 |
| 변경했는데 사이트 그대로 | Actions 워크플로우 1-2분 기다리기, 또는 브라우저 새로고침 (Ctrl+Shift+R) |

---

## 작업 메모

- 새 작업 시 현재 `index.html` 전체 내용을 다른 Claude 대화에 첨부 (대화 간 컨텍스트 공유 안됨)
- 인수인계 문서도 같이 첨부하면 더 빠른 컨텍스트 파악 가능
- Firebase Console: https://console.firebase.google.com → `grades-of-test` 프로젝트
- 깃허브 레포: https://github.com/knkscampusk-web/quetta-scholarship
