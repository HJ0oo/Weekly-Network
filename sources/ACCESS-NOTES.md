# 소스 접근 노트

매주 루틴이 **시작할 때 읽고, 끝날 때 갱신하는** 파일이다.
목적은 하나: **지난주에 이미 실패한 방법을 이번 주에 다시 시도하느라 시간을 낭비하지 않는 것.**

최종 확인: 2026-09-21

---

## 요약표

| 소스 | 상태 | 쓸 방법 |
|---|---|---|
| TTA 목록 (committee.tta.or.kr) | 동작 | curl + `iconv //IGNORE` |
| TTA 본문 PDF (weekly.tta.or.kr) | **차단(403)** | 제목만 쓰고 웹검색으로 보완 |
| arXiv API (export.arxiv.org) | 동작(2026-09-21엔 429 없이 성공) | 그래도 1회 시도 후 안 되면 목록 페이지로 전환 |
| arXiv 목록·초록 페이지 (arxiv.org) | 동작 | WebFetch로 `/list/...?skip=N`, `/abs/ID` |
| 3GPP (www.3gpp.org) | **차단** | 웹검색으로 대체 |
| IITP (itfind.or.kr) | 동작 | WebFetch로 weekly list.do |
| IITP (iitp.kr) | 목록 안 보임(JS) | itfind 쪽을 쓸 것 |
| IEEE (ieeexplore / comsoc / spectrum) | **차단(4주 연속)** | 1회만 시도, 안 되면 포기 |
| 국내 언론 일부 (edaily, boannews 등) | **본문 차단** | 검색 스니펫으로 확인 |
| 한국경제(hankyung.com) | **본문 차단(신규 확인, 09-21)** | 검색 스니펫으로 확인 |
| 이포커스(e-focus.co.kr) | **본문 차단(신규 확인, 09-21)** | 검색 스니펫으로 확인 |
| 뉴스1(news1.kr) | **본문 차단(신규 확인, 09-21)** | 검색 스니펫으로 확인 |
| 네이트뉴스(m.news.nate.com) | **본문 차단(신규 확인, 09-21)** | 검색 스니펫으로 확인, 링크만 인용 |
| IEEE ComSoc 기술블로그(techblog.comsoc.org) | **본문 차단(신규 확인, 09-21)** | 검색 스니펫으로 확인 |

---

## 상세

### TTA ICT Standard Weekly
- 목록은 EUC-KR 인코딩이다. **`iconv -f euc-kr -t utf-8//IGNORE`를 써야 한다.**
  `//IGNORE` 없이 쓰면 깨진 바이트에서 멈춰 **아무 출력 없이 조용히 실패**한다. (실제로 겪음)
- 본문 페이지(`weekly_view.jsp`)의 내용 영역(`div.con`)은 비어 있다. 진짜 본문은 첨부 PDF다.
- 그 PDF 호스트(`weekly.tta.or.kr`)는 curl에서 403, WebFetch에서 차단이다. **2026-09-01·09-15 모두 실패.**
- 목록 페이지는 최신 1건만 노출한다. `nowPage` 파라미터를 바꿔도 과거 호가 순서대로 나오지 않는다. 지난 호 추적에 시간 쓰지 말 것.

### arXiv
- `http://export.arxiv.org/api/query?...` 는 `-L` 없이는 301만 돌아온다.
- `-L`을 붙여도 **429(요청 제한)가 자주** 난다. https 쪽은 연결이 걸려 타임아웃 나기도 한다.
- 긴 `sleep` 후 재시도는 **하네스가 차단**한다. 기다리지 말고 경로를 바꿀 것.
- 검증된 우회 경로 (2026-09-15 성공):
  1. WebFetch → `https://arxiv.org/list/cs.NI/2026-09?skip=75` — skip 값을 키우면 최신 구간이 나온다
  2. WebFetch → `https://arxiv.org/abs/2609.11843` — 제목·제출일·초록 확보
  - `?show=75` 를 붙이면 **400 오류**가 난다. skip만 쓸 것.
- arXiv 번호 뒷자리는 그 달 전체 제출 순번이다. 하루 약 850편 기준으로 날짜를 역산할 수 있다.

### 3GPP / ITU
- `www.3gpp.org`는 이 환경의 네트워크 정책으로 차단(EGRESS_BLOCKED).
- 웹검색으로 충분히 대체된다. 실제로 잘 잡힌 소스: `6gfutures.substack.com`, `ericsson.com/blog`, `techblog.comsoc.org`, `firstnet.gov`.
- ITU 쪽은 새 소식이 몇 달에 한 번뿐이다. 없으면 "이번 주 신규 소식 없음"으로 짧게 적고 넘어갈 것.

### IITP 주간기술동향
- 검증된 경로 (2026-09-15 성공): WebFetch → `https://www.itfind.or.kr/publication/regular/weeklytrend/weekly/list.do`
  → 최신 호수·발행일·기사 제목이 바로 나온다.
- `iitp.kr` 의 목록 페이지는 자바스크립트 렌더링이라 내용이 보이지 않는다.
- 발행 주기가 매주가 아닐 수 있다. 최신호가 지난주 것이면 그대로 표기.

### IEEE 계열
- `ieeexplore.ieee.org`, `www.comsoc.org`, `spectrum.ieee.org` 모두 EGRESS_BLOCKED.
- 2026-08-25, 09-01, 09-15, 09-21 **4주 연속 실패.** 매주 같은 시도를 반복하지 말 것.
- `techblog.comsoc.org`도 2026-09-21에 EGRESS_BLOCKED로 신규 확인됨. 웹검색 스니펫으로만 대체 가능.
- 대안: 같은 연구의 arXiv 공개본을 찾거나, 그냥 그 주는 생략한다.

### 이번 주(09-21) 새로 확인된 차단 도메인
- `www.hankyung.com`, `www.e-focus.co.kr`, `www.news1.kr`, `m.news.nate.com` 모두 WebFetch에서 EGRESS_BLOCKED.
- 이 도메인들은 WebSearch 결과 스니펫만으로 날짜·내용을 확인하고, 링크는 인용하되 직접 열람은 못 한다고 가정할 것.
- TTA·IITP 모두 이번 주 **신규 호가 없었다** (지난주와 동일한 제1307호·제2220호). 매주 목록을 확인하되, 같은 번호면 새 기사로 쓰지 말 것.

### 국내 언론
- `edaily.co.kr`, `m.boannews.com` 등 여러 매체가 본문 접근 차단이다. WebSearch 결과 스니펫으로 내용을 확인하라.
- **날짜 함정 주의**: 검색 결과에 2025년 기사가 최신인 것처럼 섞여 나온다.
  실제 사례 — "LG유플러스 금오공대 오픈랜 실증단지"(2025-10), "쿤텍·ETRI 오픈랜 보안"(2025-11)이 2026-09 검색 결과 상단에 나왔다.
  **발행일을 확인하지 못한 기사는 절대 리포트에 넣지 말 것.**

---

## 갱신 규칙
- 차단됐던 소스가 다시 열리거나, 되던 소스가 막히면 **그 주에 바로 이 파일을 고칠 것.**
- 새로 찾은 우회 경로는 성공한 날짜와 함께 기록할 것.
