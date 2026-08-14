# 벤처인증 랜딩페이지

박세준 변리사 · 마커블 벤처인증 상담 신청 랜딩페이지입니다.
빌드 과정이 없는 정적 사이트로, GitHub → Vercel 자동 배포로 운영합니다.

- **운영 주소** https://벤처기업인증.kr (퓨니코드 `xn--ok0b27ym0gzvbp4dy9f.kr`)
- **호스팅** Vercel
- **리드 처리** Make 웹훅 → 구글 시트 · 슬랙 · 이메일

---

## 파일 구조

```
.
├── index.html            랜딩페이지 (전부 여기 들어 있음)
├── thanks.html           신청 완료 페이지
├── vercel.json           배포 설정 — 반드시 함께 올려야 함
├── robots.txt            검색·AI 크롤러 안내
├── sitemap.xml           사이트맵
├── favicon.ico           브라우저 탭 아이콘
├── favicon-32.png        고해상도 탭 아이콘
├── apple-touch-icon.png  iOS 홈 화면 아이콘
├── .gitignore
└── images/
    ├── og.jpg                  링크 공유 미리보기 이미지 (실제 사용)
    ├── park-sejun.jpg          변리사 사진 원본 (보관용)
    └── park-sejun-circle.png   변리사 사진 원형 크롭 (보관용)
```

### 반드시 함께 배포해야 하는 세 파일

`index.html` · `thanks.html` · `vercel.json`

`vercel.json` 에는 두 가지가 걸려 있습니다.

- **`Referrer-Policy` 헤더** — 없으면 유튜브 임베드가 **오류 153** 으로 재생되지 않습니다
- **`cleanUrls`** — 폼 제출 후 이동하는 `/thanks` 주소를 처리합니다

나머지 파일은 없어도 페이지는 뜨지만, 아이콘·공유 미리보기·검색 노출이 빠집니다.

### 사진에 대하여

변리사 사진 두 장은 **HTML 안에 직접 담겨 있습니다**(base64). `images/` 의 원본은
나중에 다른 곳에 쓰기 위한 보관용이며, 페이지는 이 파일들이 없어도 정상 작동합니다.

파일 방식으로 되돌리려면 각 HTML 맨 아래의 사진 삽입 스크립트를 지우고
`<img>` 의 `src` 를 `images/park-sejun.jpg` · `images/park-sejun-circle.png` 로 바꾸면 됩니다.

---

## 설정값이 들어 있는 곳

### index.html — 스크립트 상단

| 값 | 용도 |
|---|---|
| `ENDPOINT` | Make 웹훅 주소. 폼 데이터가 이리로 갑니다 |
| `THANKS_URL` | 제출 후 이동할 주소 (`/thanks`) |
| `CONSENT_VERSION` | 개인정보 동의문 버전. **문구를 고치면 이 날짜도 바꿀 것** |
| `FORM_TOKEN` | 폼에서 온 요청임을 표시하는 값. Make 필터가 이걸로 봇을 걸러냅니다 |

### thanks.html — 스크립트 상단

| 값 | 용도 |
|---|---|
| `NAVER_BOOKING` | 네이버 예약 주소. 비우면 예약 카드가 자동으로 숨겨집니다 |

날짜(`startDate`)는 접속 시점을 계산해 자동으로 붙으므로 주소에 넣지 마세요.

### 메타 픽셀

두 페이지 `<head>` 에 설치되어 있습니다. 전환(`Lead`)은 **완료 페이지에서만**
한 번 발생하며, 어느 서비스에서 온 전환인지도 함께 기록됩니다.

---

## Make 로 전송되는 데이터

```json
{
  "name": "홍길동",
  "phone": "010-1234-5678",
  "email": "hong@company.com",
  "plan": "patent-first",
  "planLabel": "기술 근거 설계부터 벤처인증까지",
  "source": "plan-card",
  "sourceLabel": "상품 카드",
  "page": "벤처인증 랜딩",
  "token": "vclanding",
  "consent": true,
  "consentAt": "2026-08-14T14:32:05+09:00",
  "consentVersion": "2026-08-12",
  "submittedAt": "2026-08-14T14:33:11+09:00"
}
```

**사람이 보는 자리에는 `planLabel`, 조건 분기에는 `plan`** 을 쓰세요.
문구를 바꿔도 분기 조건은 그대로 살아 있습니다.

| `plan` | `planLabel` |
|---|---|
| `consulting` | 사업계획서 지원 |
| `full` | 벤처인증 전 과정 대행 |
| `patent-first` | 기술 근거 설계부터 벤처인증까지 |

| `source` | `sourceLabel` |
|---|---|
| `hero` | 첫 화면 폼 |
| `plan-card` | 상품 카드 |
| `nav` | 상단 버튼 |
| `final` | 마지막 폼 |

`consentAt` 은 **동의 체크박스를 누른 시각**이고, `submittedAt` 은 제출 버튼을 누른
시각입니다. 동의 시점을 증명해야 할 때는 `consentAt` 을 보면 됩니다.

### Make 필터 (필수)

웹훅 주소는 페이지 소스에 노출되므로 자동 스캐너가 빈 요청을 보냅니다.
웹훅 바로 다음에 필터를 걸어 막아야 합니다.

```
name    →  Exists
token   →  Equal to  →  vclanding
consent →  Exists
```

웹훅에서 선이 여러 갈래로 갈라져 있으면 **갈래마다** 필터가 필요합니다.

---

## 수정할 때 알아둘 것

### 문단 여백이 안 먹을 때

기본 규칙이 `.vc :where(p)` 로 되어 있어 클래스로 준 여백이 항상 이깁니다.
그래도 안 먹으면 `.vc p.클래스명` 형태로 특이도를 올리세요.

### 모바일에서만 줄바꿈

`<br class="vc-brm" />` 를 쓰면 좁은 화면에서만 줄이 바뀝니다.
반대로 PC에서만 보이는 문장 사이 여백은 `<span class="vc-sp"></span>` 입니다.

### 화면 크기별 스타일

CSS 아래쪽에 구간이 나뉘어 있습니다.

```
@media (max-width: 1000px)   태블릿 이하
@media (max-width:  720px)   휴대폰
```

한쪽만 고치고 싶을 때 해당 구간 안에서 작업하면 됩니다.

### 서비스 3종을 바꿀 때 같이 손봐야 하는 곳

1. 서비스 카드 (제목 · 상황 문구 · 항목)
2. 폼 선택지 2곳 (첫 화면 · 마지막)
3. `PLAN_LABEL` (Make 로 가는 한글 라벨)
4. `thanks.html` 의 서비스별 안내 문구
5. 푸터 서비스 목록
6. 구조화 데이터(`ld+json`)의 `hasOfferCatalog`

`plan` 코드값(`consulting` `full` `patent-first`)은 바꾸지 마세요.
Make 필터와 이미 쌓인 시트 데이터가 이 값을 씁니다.

---

## 남은 일

- [ ] 개인정보처리방침에 **위탁·국외이전 조항** 추가
      (Make → EU, 구글 시트 → 미국, 메타 픽셀 → 미국)
- [ ] 알림톡 템플릿 심사 신청 (`알림톡_템플릿.md` 참고)
- [ ] 통신판매업 신고 주소와 실제 사업장 주소 일치 여부 확인
      (신고는 금천구, 표기는 강남구)
- [ ] 웹훅 주소 노출 해소 — 본격 운영 시 `/api/lead` 프록시로 전환 검토
