# 05. 코드 작성 규정

## 공통 (Kotlin)
- Kotlin 공식 코딩 컨벤션을 따른다. 포맷터/린터: **ktlint + detekt** (설정 후 모든 변경에서 통과해야 한다).
- 네이밍: 클래스 `PascalCase`, 함수·변수 `camelCase`, 상수 `UPPER_SNAKE_CASE`, 패키지 소문자.
- `!!` 사용 금지. null은 안전 호출, `?:`, `requireNotNull`(근거 있을 때)로 처리한다.
- `var`보다 `val`, 가변 컬렉션보다 불변 컬렉션을 우선한다.
- 함수는 한 가지 일만 하며 짧게 유지한다. 매직 넘버·문자열은 상수로 뺀다.
- 주석은 **왜**를 설명할 때만 쓴다. 코드가 말해주는 **무엇**은 쓰지 않는다. 주석 언어는 한국어.
- 죽은 코드, 주석 처리된 코드, 사용하지 않는 import를 남기지 않는다.

## 코루틴·비동기
- `GlobalScope` 금지. 앱은 `viewModelScope`/`lifecycleScope`, 서버는 주입된 스코프를 사용한다.
- Dispatcher는 주입받아 사용한다 (테스트 가능성). 메인 스레드에서 블로킹 작업 금지.
- 취소(`CancellationException`)를 삼키지 않는다.
- UI에서는 `collectAsStateWithLifecycle()`로 Flow를 수집한다.

## 에러 처리
- 예외를 빈 `catch`로 삼키지 않는다. 처리하거나, 로그를 남기고 의미 있는 오류로 변환한다.
- 앱 도메인 계층은 성공/실패를 `Result` 또는 sealed 타입으로 표현하고, UI는 이를 `UiState`로 매핑한다.
- 서버는 `@RestControllerAdvice`에서 오류를 한곳에서 변환한다 (`01-architecture.md`의 응답 형식).
- 사용자에게 스택 트레이스·내부 메시지를 노출하지 않는다.

## 로깅
- 앱은 `Timber`(승인 시) 또는 `Log`, 서버는 SLF4J를 쓴다. `println` 금지.
- 로그 레벨을 구분한다. **개인정보·토큰·기사 본문은 로그에 남기지 않는다** (`02-security.md`).

## 테스트
- 새 로직에는 테스트를 함께 작성한다.
  - 앱: ViewModel, UseCase, Mapper → JUnit5 + MockK + Turbine(Flow). UI 테스트는 핵심 화면만.
  - 서버: Service 단위 테스트, Repository/Controller는 슬라이스 테스트(`@DataJpaTest`, `@WebMvcTest`).
- 외부 API는 호출하지 않고 가짜(Fake)나 목(Mock)으로 대체한다.
- 테스트 이름은 동작을 설명한다 (`요약이_없으면_빈_상태를_반환한다`).
- 버그 수정에는 재현 테스트를 먼저 추가한다.

## Git
- 브랜치: `main`(안정), 작업은 `feature/*`, `fix/*`, `docs/*`.
- 커밋은 **Conventional Commits**: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`. 제목은 한국어 가능, 한 커밋에 한 가지 목적.
- 작업 단위가 끝나면 커밋·푸시한다 (사용자 위임). `push --force`, 브랜치 삭제, 히스토리 재작성은 사용자 확인 후에만 한다.
- push 인증이 실패하면 `gh auth status`로 확인하고 `git -c credential.helper= -c credential.helper='!gh auth git-credential' push`를 시도한다.
- 생성된 빌드 산출물, IDE 설정, 비밀 파일은 커밋하지 않는다.

## 완료 기준 (Definition of Done)
작업을 끝냈다고 보고하기 전에 다음을 확인한다.
1. 빌드가 성공한다 (앱 `./gradlew assembleDebug`, 서버 `./gradlew build`).
2. 관련 테스트가 통과하고 린트에 문제가 없다.
3. 규정 문서(`documents/`)를 위반하지 않는다.
4. 구조·API·DB가 바뀌었다면 관련 문서도 함께 갱신했다.
5. 실행하지 못한 검증이 있으면 **무엇을 못 했는지 그대로 보고**한다.
