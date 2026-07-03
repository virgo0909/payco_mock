# PAYCO 목업 (payco_bundle)

실제 Payco 앱 화면을 **HTML/CSS/JS 단일 페이지**로 재현한 개인 학습·포트폴리오용 UI 클론.
iPhone Safari "홈 화면에 추가"로 standalone(전체화면) 실행하는 것이 목표.
프레임워크·번들러 없이 바닐라 JS + CSS 변수로 구현. 프로젝트 공통 철칙은 상위 `../CLAUDE.md` 참조.

> ⚠️ `file://`(파일 더블클릭)로 열면 `fetch('data.json')`이 브라우저 보안에 막혀 **스플래시에서 멈춘다.**
> 반드시 **로컬 서버** 또는 **GitHub Pages(http/https)** 로 열 것.
> ```
> python -m http.server 8137     # → http://localhost:8137/
> ```

## 화면 (단일 index.html, 화면 스택 슬라이드)
1. **스플래시** — 흰 배경 + 빨간 PAYCO, ~1.1초 후 홈으로 페이드.
2. **혜택 홈** — 검색바·카테고리 탭·프로모 배너·추천 혜택 브랜드 그리드(`benefits`), 하단 탭바(혜택/포인트/결제/금융/전체).
3. **MY PAYCO** — VIP★GOLD, 기업복지(넥슨게임즈) 진입, PAYCO Point, 카드/계좌·멤버십·전자문서.
4. **기업복지** — 복지 포인트 총 보유(계산 표시), 바코드 사용(흉내), 복지포인트 카드, 메뉴(결제내역→결제내역 화면).
5. **결제 내역**(핵심) — 상단 **월 드롭다운**, 월 총 결제금액(계산), 날짜별 그룹 리스트. 우상단 이용리포트.
6. **이용리포트** — 월 누적·전월대비·최근 3일 일평균·주별/월별 비교 바차트(전부 계산).

씬 전환 시 상단에 서버풍 로딩바가 잠깐 뜬 뒤 슬라이드. `prefers-reduced-motion` 대응.

## 데이터 (`data.json`) — 모든 숫자의 원천
표시되는 모든 금액은 `transactions[]`에서 코드가 **실시간 계산**한다(하드코딩 없음).

- **거래 71건 / 2025-07 ~ 2026-06 / 결제수단 = 복지포인트 단일 / 취소 2건.**
- 월 총 결제금액은 해당 달력월의 `결제완료` 합. 예: **2026-06 = 417,180원**, 2026-05 = 387,800원.
- 이용리포트의 "오늘" 기준 = `config.today` = **2026-07-03**(스크린샷 촬영 시점, 실제 시스템 날짜 미사용).

### ⚠️ 스크린샷과 헤드라인 숫자가 다른 이유
원본 export(`PAYCO_Payment_20260703.xlsx`)가 **부분 export**라, 요약행(84건/4,354,182원)보다 상세행이 적다(71건).
그래서 앱의 6월 총액은 **417,180원**으로, 스크린샷 헤드라인(473,180원)과 다르다. 이는 **의도된 것**(정직하게
원본 상세행 그대로 사용, 없는 거래를 지어내지 않음). 가맹점명도 원본 그대로("PAYCO 복지포인트 카드" 제네릭 포함).

### config 값
| 키 | 값 |
|---|---|
| user | 김학중 |
| grade | VIP GOLD (화면엔 VIP★GOLD로 표기) |
| company | 넥슨게임즈 / empNo 89799 / email 70****@naver.com |
| paycoPoint | 2,113 P |
| welfarePoint | 1,021,984 P |
| welfareCardNo | 9491-07\*\*-\*\*\*\*-0345 |
| today | 2026-07-03 |

### 거래 스키마
```json
{ "id":"...", "date":"2026-06-28", "time":"19:28", "merchant":"농협하나로마트(양재점)",
  "order":"...", "amount":8000, "supply":7273, "vat":727,
  "method":"복지포인트", "methodTail":"0345", "status":"결제완료|취소완료" }
```

## 새 데이터로 갱신
원본 xlsx(시트 `NHN_PAYCO_1`)를 받으면 변환기로 `data.json`을 다시 만든다(로컬 전용 도구, 배포 제외):
```
python export_payco.py "PAYCO_Payment_YYYYMMDD.xlsx" -o data.json
```
변환 후 `python`으로 UTF-8 재확인(터미널 한글은 콘솔에서 깨져 보여도 파일은 정상).

## 커스터마이즈
- 색: `index.html`의 `:root` 변수(`--red` = Payco 레드 `#F13C41` 등).
- 애니메이션 속도: `:root{--dur}`.
- 홈 브랜드 그리드: `data.json`의 `benefits[]`.
- 에셋: `assets/`의 `splash-logo.png`·`app-icon.png`·`nexon-logo.png`·`welfare-card.png`(없으면 텍스트/색 폴백).

## 설계 결정 메모
- **결제내역 필터 UI는 의도적으로 제외**(결제수단 드롭다운·기간/이용구분/이용처 칩·필터 바텀시트 없음).
  월 전환은 **상단 월 드롭다운**만으로 한다.
- 자격증명·결제·개인정보 입력(충전·바코드·검색·로그인 등)은 **동작 흉내만** 낸다(실제 처리 없음).

## 배포 (GitHub Pages)
- 공개 대상: `index.html`, `data.json`, `manifest.json`, `assets/`, `.gitignore`, `README.md`.
- 제외(로컬 유지): `PAYCO_Payment_*.xlsx`, `export_payco.py`, `crop_assets.py`, `../Payco_Mock/`.
- Settings → Pages → Deploy from a branch → `main` / `/(root)` → Save → `https://<user>.github.io/<repo>/`.
- iPhone Safari로 접속 → 공유 → 홈 화면에 추가 → standalone 실행.
- ⚠️ public 저장소는 누구나 열람 → `data.json`의 실제 가맹점·금액이 공개됨.
