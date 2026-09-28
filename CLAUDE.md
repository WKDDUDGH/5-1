# CLAUDE.md

이 파일은 이 저장소에서 작업할 때 Claude Code(claude.ai/code)가 따를 안내 사항입니다.

## 프로젝트

습관 체크 개인용 웹앱. 코드는 [index.html](index.html) 하나(HTML+CSS+JS 인라인, IIFE 안에 전부)다. 외부 라이브러리·CDN·이미지·폰트 파일·네트워크 요청을 쓰지 않는다.

- [PRD.md](PRD.md) — 동작 기준. 어긋나면 PRD가 우선이며, 다르게 만들 때는 PRD도 같이 고친다. 8장이 완료 체크리스트다.
- [STEPS.md](STEPS.md) — 구현 4단계(뼈대 → 기능 → 디자인 → 점검). 1~4단계 모두 끝났고, 커밋도 단계별로 나뉘어 있다.

**미해결**: PRD 8장의 마지막 항목("디자인이 첨부 캡처와 같은 느낌")은 캡처 이미지가 전달되지 않아 미확인이다. 지금 디자인은 임시(카드형)이며, 캡처를 받으면 `:root` CSS 변수(색·모서리·그림자)와 간격을 조정하고 PRD 7장을 갱신한다.

## 실행 / 테스트

빌드·린트·테스트 도구가 없다. `index.html`을 브라우저로 열어 확인한다.

- Claude 데스크톱 앱의 브라우저 패널은 `file://` 페이지에서 `localStorage`를 막아서(`data:` 스냅샷으로 열림) 저장·새로고침 테스트를 할 수 없다. 이때는 임시 정적 서버를 띄워 `http://localhost:포트/`로 연다. 서버 스크립트는 저장소 밖(스크래치 폴더)에 두고, 끝나면 프로세스를 끄고 테스트 데이터(`localStorage.clear()`)를 지운다.
- 날짜 동작은 실제 자정을 기다리지 않고 페이지에서 `window.Date`를 +1일로 바꾼 뒤 `focus` 이벤트를 발생시켜 확인했다. 10일 이상 연속 같은 데이터는 개발자도구에서 `habit-tracker-v1`을 직접 써서 만든다.
- 좁은 화면 확인은 360px 기준이다 (PRD의 터치 영역 최소 40px 때문에 7일 칸이 360px에서 약 41px이다. 여백·간격을 늘리면 깨진다).

## 구조 (index.html 안)

데이터 흐름은 한 방향이다: 이벤트 → `state` 수정 → `commit()`(= `save()` + `render()`). `render()`는 목록 전체를 `textContent = ''`로 비우고 다시 만든다. DOM을 직접 고치는 곳은 없다.

- `state = { habits: [{ id, name, done: { "YYYY-MM-DD": true } }] }`, `localStorage` 키 `habit-tracker-v1`. 체크한 날만 저장하고 취소하면 키를 삭제한다.
- **모든 체크 동작은 `toggleDay(id, key)` 하나**로 모인다. 오늘 체크 버튼도 7일 칸도 같은 함수를 부르고, 그래서 오늘 버튼과 오늘 칸이 항상 동기화된다. 별도 상태를 두지 않는다.
- 날짜 함수 `dateKey(d)`(로컬 `YYYY-MM-DD`)와 `daysAgo(n)`을 7일 칸(`buildWeek`), 연속 기록(`streakOf`), 완료 수(`renderSummary`)가 함께 쓴다. `toISOString()`(UTC)은 한국 시간 자정~오전 9시에 하루가 어긋나므로 쓰지 않는다.
- `streakOf`: 오늘을 했으면 오늘부터, 아니면 어제부터 거꾸로 센다. 7일 칸이 아니라 저장된 전체 기록을 본다.
- 자정 처리: `render()`가 `renderedDay`에 그린 날짜를 저장하고, `visibilitychange`/`focus` 때 날짜가 바뀐 경우에만 다시 그린다.
- 저장 실패는 앱을 죽이지 않는다. `warning` 문자열 하나로 두 가지를 구분한다: `BLOCKED_MSG`(localStorage 접근 불가/저장 실패), `CORRUPT_MSG`(저장된 JSON 손상 또는 형태 불일치 → 빈 상태로 시작).
- 습관 이름은 `textContent`로만 출력한다 (`innerHTML` 사용 금지).
- 제약: 이름 20자(`maxlength` + `slice`), 습관 최대 10개, 삭제는 `confirm` 후 기록까지 함께 삭제. 수정·정렬·알림·통계는 비목표.
- `file://`로 열면 저장소가 파일 경로별이라 파일을 옮기거나 이름을 바꾸면 기록이 안 보인다. 정상 동작이다.

## Git

- 원격 `origin`은 `https://github.com/WKDDUDGH/5-1.git`, 브랜치는 `main`.
- 이 컴퓨터에는 Git 사용자 정보가 설정돼 있지 않다. 커밋할 때 `git -c user.name=... -c user.email=... commit`처럼 그 커밋에만 지정해 왔다 (설정 파일은 바꾸지 않음).
- `gh` CLI가 없고, push 때 GitHub 자격 증명 창(브라우저 로그인)이 떠서 사용자가 직접 로그인해야 한다.
