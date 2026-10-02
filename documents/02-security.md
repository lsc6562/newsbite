# 02. 보안 규정

## 비밀 정보
- API 키, 토큰, DB 비밀번호, 서명 키를 **코드·문서·로그·커밋에 절대 넣지 않는다.**
- 서버: 환경 변수 또는 `.env`(gitignore). 앱: `local.properties` → `BuildConfig` (gitignore).
- 저장소에는 `.env.example`, `local.properties.example`만 둔다 (값은 비움).
- 앱에는 **비밀 키를 넣지 않는다.** 앱 안의 값은 추출될 수 있다고 가정한다. 비밀 키가 필요한 외부 호출은 서버가 맡는다.
- `.gitignore`에 `*.jks`, `*.keystore`, `.env`, `local.properties`, `google-services.json`(필요 시)을 포함한다.
- 비밀이 실수로 커밋되면 즉시 사용자에게 알리고 키 폐기를 안내한다.

## 인증·인가
- 인증은 Firebase Auth. 서버는 모든 보호 API에서 **ID 토큰을 검증**한다.
- 사용자는 **본인 데이터에만** 접근할 수 있다. 모든 조회·수정에서 소유자 검사를 한다 (IDOR 방지).
- 토큰은 앱에서 Keystore 기반 저장소(EncryptedSharedPreferences/DataStore + 암호화)에만 둔다.

## 통신
- **HTTPS만** 사용한다. `usesCleartextTraffic=false`. 개발용 예외는 debug 빌드 설정으로만 허용한다.
- 서버 CORS는 필요한 출처만 허용한다.
- 로그인, 발송 등 민감 API에는 요청 제한(rate limit)을 둔다.

## 데이터 보호
- 수집 최소화: 이메일(또는 Firebase UID), 관심 분야, SNS 연동 대상만 수집한다.
- 텔레그램 `chat_id` 등 SNS 식별자는 **암호화하여 저장**한다.
- 위치: 원본 위경도를 서버에 보내거나 저장하지 않는다. 기기 안에서만 쓴다.
- 회원 탈퇴 시 해당 사용자의 모든 데이터를 삭제한다 (관련 테이블 포함).
- **로그에 개인정보, 토큰, chat_id, 이메일을 남기지 않는다.**

## 입력 검증·인젝션 방지
- 서버는 모든 입력을 검증한다 (Bean Validation). 클라이언트 검증을 신뢰하지 않는다.
- SQL은 파라미터 바인딩만 사용한다. 문자열 연결로 쿼리를 만들지 않는다.
- LLM에 넣는 기사 텍스트는 **신뢰할 수 없는 입력**이다. 기사 내용이 지시문으로 동작하지 않도록 프롬프트에서 데이터 구간을 분리하고, LLM 출력을 그대로 신뢰해 실행하거나 HTML로 렌더링하지 않는다.
- 외부 API 응답도 검증 후 사용한다 (null, 크기, 형식).

## Android 권한·빌드
- 권한은 최소화한다. 위치는 `ACCESS_COARSE_LOCATION`만 요청하고, 사용 직전에 목적을 안내한 뒤 요청한다.
- release 빌드는 R8 난독화·축소를 켠다. `android:allowBackup`은 `false`로 둔다.
- WebView 사용은 승인 없이 추가하지 않는다.

## 의존성
- 알려진 취약점이 있는 라이브러리를 쓰지 않는다. 새 의존성 추가 시 유지보수 상태를 확인한다.
