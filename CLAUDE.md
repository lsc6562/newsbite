# Newsbite

관심 분야 뉴스를 매일 요약해 Android 앱, 텔레그램, Slack으로 전달하는 **포트폴리오 프로젝트**. 이 저장소는 앱(`app/`), 서버(`server/`), 규정 문서(`documents/`)로 구성된다.

## 최우선 규칙
- 아래 규정 문서는 **모든 작업의 기준**이다. 코드를 쓰기 전에 관련 규정을 따른다.
- 규정과 요청이 충돌하면 작업 전에 사용자에게 알린다. 규정을 몰래 어기지 않는다.
- 응답은 항상 **한국어**로 한다 (코드, 명령어, 경로, 오류 원문은 그대로).
- 요청한 범위만 구현한다. 새 의존성·구조 변경은 승인 후에 한다.
- 커밋·푸시는 사용자가 요청할 때만 한다.

## 규정 문서
@documents/00-project-overview.md
@documents/01-architecture.md
@documents/02-security.md
@documents/03-db-design.md
@documents/04-ui-guidelines.md
@documents/05-coding-conventions.md
@documents/06-legal-and-content-policy.md
@documents/07-ai-working-rules.md

## 절대 금지 (요약)
- 비밀 키·토큰·chat_id·Slack Webhook URL을 코드, 로그, 커밋에 넣기
- 기사 본문 저장·공개
- 앱에서 뉴스·LLM·SNS API 직접 호출 (서버 경유)
- `!!`, `GlobalScope`, 하드코딩된 색·문자열
- 검증하지 않은 작업을 완료로 보고하기

## 자주 쓰는 명령
(프로젝트 생성 후 채운다)
- 앱 빌드: `./gradlew assembleDebug` (app/)
- 서버 빌드·테스트: `./gradlew build` (server/)
