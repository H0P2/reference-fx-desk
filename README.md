# Reference FX Desk

공개 ECB 기준환율로 `round4(EUR/KRW ÷ EUR/USD)`를 계산하는 정보판. 키 없는 호출이며 거래 체결가격이나 수익 보장이 아닙니다. 기존 개인용 CRWD 프로젝트는 별도로 보존합니다.

## 출처와 권한
- 최신 XML: https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml
- 과거 XML: https://www.ecb.europa.eu/stats/eurofxref/eurofxref-hist-90d.xml
- 재사용 근거: https://www.ecb.europa.eu/services/using-our-site/disclaimer/html/index.en.html
- ECB 출처 표기와 교차환율 계산 사실을 명시합니다.
- `source_observed_at`은 실제 원천의 기준일, 정밀도는 date. 정확한 관측 시각은 미제공이므로 `reading.source_time`은 null입니다. HTTP Last-Modified는 별도 파일 갱신 시각이며 관측 시각으로 사용하지 않습니다.

## 저장과 호출
D1 일별 고유키는 signal_id + KST 조회 날짜. 동일 날짜의 성공은 최신 정상값으로 갱신하며 최초 조회 시각은 보존합니다. 오류는 마지막 정상값을 지우지 않습니다. 4시간 내 정상 조회는 캐시로 유지하되 새로운 KST 날짜에는 실제 호출합니다. 공유 호출 간격은 최소 60초입니다. 자동 금융거래를 실행하지 않습니다.

## 실행
Node 22.13 이상. `npm ci`, `npm run db:generate`, `npm run build`. 로컬 실제 원천 수집은 `node capture-once.mjs`. 실제 조회가 가능한 로컬 서버는 `node server-local.mjs` 또는 `start-local.ps1`로 실행합니다(127.0.0.1:8897). 도구 내 8896 미리보기는 보관 기록 열람 전용입니다. 로컬 SQLite data/는 공개 소스와 배포에서 제외합니다. Cloudflare D1에는 배포 후 실제 조회로 기록합니다. schema-only Drizzle migrations는 배포 전에 적용합니다. 화면은 React로 작성하고 esbuild로 브라우저 코드와 Worker를 묶습니다.

## 검증 경계
tests/contract.test.mjs: 제공된 17개 파일의 SHA-256과 크기, 다섯 합성 실패, 복구, 날짜 중복 방지.
tests/ecb.test.mjs: XML 형식, 환율 계산, HTTP 실패 분리, D1 저장 경로, 실제 로컬 캡처 대조. 마지막 실제 캡처 검사는 로컬 `data/actual-board.json`이 있어야 실행할 수 있습니다.
tests/service.test.mjs: 저장소 실패와 원천 실패 분리, 4시간 캐시, 자정 이후 실제 호출, 503 상태와 다른 Origin 거부. 일반 검사는 `node --test tests/*.test.mjs`이며 첫 실행 전 `node capture-once.mjs`로 실제 캡처를 만듭니다.
`node tests/actual-http-check.mjs`는 실제 ECB 호출을 서버 HTTP 요청으로 실행하고 저장값·재조회 캐시·화면 API 값을 대조합니다. 격리된 임시 DB를 사용하므로 이 실행은 제출의 두 번째 실제 날짜로 세지 않습니다.
개발 도구 안에서 오래 실행되는 서버는 다른 도구의 브라우저 연결에 제한이 있습니다. 8896의 검토 화면은 파일 열람용이며, 실제 실행 서버는 사용자가 일반 PowerShell에서 `start-local.ps1`을 실행하면 됩니다. 도구 내 서버의 계속 실행과 브라우저 접근이 확인되었다고 주장하지 않습니다.
과거 차트 64건은 2026-10-06 KST에 실제 받은 역사 자료이며, 서로 다른 실제 조회 날짜 두 건을 대신하지 않습니다.
첫 실제 조회 2026-10-06. 둘째 실제 KST 날짜 기록은 대기 상태입니다. 플랫폼 t04_day 증명도 아직 발급되지 않았습니다. 정확한 관측 시각이 필요한 T04-C07/C23은 date 정밀도 허용 여부 확인이 필요합니다. 모든 기준이 완료되었다고 주장하지 않습니다.

## 제출 설명 초안
위치: 공개 정보판의 실제 원천 조회 및 데이터가 안 올 때 구역.
행동: 실제 원천 조회 → 실패 재생 선택 → 다시 시도 D2 복구.
통과: 출처·단위·시각이 표시되고, 합성 실패의 105가 유지된 뒤 복구 120, fresh/none, 기록 2건으로 바뀝니다.
안 될 때: 원천 오류 종류 또는 저장소 연결 오류를 표시하고 이전 정상값을 유지합니다.

AI에게 맡긴 일: 데이터 연결·화면·저장·검사 구현과 실제 응답 대조.
직접 판단한 일: 사용자가 공개 재배포가 허용된 데이터로 제출용 정보판 전환을 선택했습니다.
AI 제안을 따르지 않은 일: 공개 데이터 전환은 제안된 방향에 동의하여 진행했습니다. 별도로 거절한 제안과 그 이유는 제출 전에 본인의 판단으로 확인합니다.
