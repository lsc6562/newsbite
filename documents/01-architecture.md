# 01. 아키텍처 규정

## 전체 구조
```
[Android 앱] ⇄ HTTPS ⇄ [Spring Boot 서버] ⇄ [PostgreSQL]
                              │
        ┌─────────────────────┼─────────────────────┐
   뉴스 수집(Collector)   요약(Summarizer)     발송(Delivery)
   외부 뉴스 API          LLM API             Telegram Bot API, FCM
```
- 앱은 서버 API만 호출한다. 뉴스·LLM·SNS API를 앱에서 직접 호출하지 않는다.
- 예외: 날씨는 기기 좌표를 **격자 좌표(nx, ny)로 변환한 값만** 서버 프록시(`/weather`)로 보낸다. 원본 위경도는 서버로 보내지 않는다.

## Android 앱
**패턴**: MVVM + 단방향 데이터 흐름(UDF). 계층은 `ui → domain → data` 방향으로만 의존한다.

```
app/src/main/java/.../
  ui/         화면(Composable), ViewModel, UiState, 테마
  domain/     UseCase, 도메인 모델, Repository 인터페이스
  data/       Repository 구현, Retrofit API, Room, DataStore, 매퍼
  di/         Hilt 모듈
```
규칙:
- 화면 = `XxxScreen`(Composable) + `XxxViewModel` + `XxxUiState`(sealed/data class). ViewModel은 `StateFlow<UiState>`만 노출한다.
- Composable은 상태를 직접 만들지 않는다 (hoisting). 비즈니스 로직 금지.
- `ui`는 `data`를 직접 참조하지 않는다. 반드시 `domain`의 인터페이스를 거친다.
- DTO(네트워크), Entity(Room), 도메인 모델, UiModel은 분리하고 매퍼로 변환한다.
- 초기에는 **단일 모듈**로 시작한다. 모듈 분리는 사용자 승인 후에만 한다.
- 네비게이션은 Navigation Compose 한 곳(`NavGraph`)에서 관리한다.

## 서버
**패턴**: 계층형 + 기능별 패키지.
```
server/src/main/kotlin/.../
  user/  category/  article/  summary/  delivery/  weather/  common/
    └ 각 기능: controller → service → repository, domain, dto
```
규칙:
- Controller는 요청 검증과 응답 변환만 한다. 로직은 Service에 둔다.
- Entity를 API 응답으로 직접 노출하지 않는다 (DTO 사용).
- 기능 간 호출은 Service 인터페이스를 통해서만 한다. 다른 기능의 Repository를 직접 쓰지 않는다.
- **외부 연동은 인터페이스 뒤에 둔다** (교체·테스트 용이):
  - `NewsSource` (뉴스 수집), `SummaryProvider` (LLM), `DeliveryChannel` (발송: 1차는 `TelegramChannel`만 구현)
- 일일 파이프라인: 수집 → 중복 제거 → 요약 → 사용자별 발송. 스케줄은 `@Scheduled`, 각 단계는 **멱등**하게 만든다 (재실행해도 중복 발송 금지).
- 외부 호출은 타임아웃, 재시도(제한 횟수), 실패 로깅을 반드시 설정한다.

## API 규정
- REST, JSON, 경로는 `/api/v1/...`, 복수 명사 사용.
- 응답 형식은 통일한다: 성공 `{ "data": ... }`, 실패 `{ "error": { "code": "...", "message": "..." } }`.
- 목록 조회는 페이지네이션을 적용한다.
- 시간은 ISO-8601 UTC로 주고받는다.
- 하위 호환이 깨지는 변경은 버전을 올린다.

## 의존성 규칙
- 새 라이브러리는 **사용자 승인 후** 추가한다. 이유와 대안을 함께 제시한다.
- 버전은 `libs.versions.toml`에서만 관리한다. 하드코딩 금지.
