# 00. 프로젝트 개요

## 목적
관심 분야의 뉴스 기사를 매일 요약해 앱과 SNS(텔레그램, Slack)로 전달하는 서비스. **상업 목적이 아닌 포트폴리오 프로젝트**이며 Android만 개발한다.

## 구성
| 디렉터리 | 내용 |
|---|---|
| `app/` | Android 앱 (Kotlin, Jetpack Compose) |
| `server/` | 백엔드 (Spring Boot, Kotlin): 뉴스 수집 · 요약 · 발송 |
| `documents/` | 규정 문서 (이 폴더) |

## 핵심 기능 (MVP 순서)
1. **1차**: 회원가입 시 관심 분야 선택 → 맞춤 요약 → 앱 조회 + 텔레그램 발송 + Slack Webhook 발송 (사용자가 직접 만든 Incoming Webhook URL 등록)
2. **2차**: 수치 데이터 이미지화(차트), TTS 읽어주기
3. **3차**: 위치 기반 날씨 (공개 API)

## 이번 범위에서 하지 않는 것 (Non-goals)
- iOS, 웹 클라이언트
- 카카오 알림톡, 인스타그램, 라인, Slack 앱(OAuth/Bot) 발송 (구조만 확장 가능하게 둔다)
- 결제, 광고, 관리자 페이지, 다국어
- Play 스토어 정식 공개 (내부 테스트 트랙까지)

## 핵심 원칙
1. **규정 문서가 코드보다 우선한다.** 충돌하면 코드를 고치거나, 문서 변경을 사용자에게 먼저 제안한다.
2. **MVP 우선.** 요청받지 않은 기능, 추상화, 의존성을 추가하지 않는다.
3. **저작권·개인정보를 설계 단계에서 지킨다.** 상세는 `06-legal-and-content-policy.md`.
4. 모든 요약에는 **출처, 원문 링크, "AI 요약" 표시**가 붙는다.

## 기술 스택 요약
- 앱: Kotlin, Jetpack Compose, Material 3, Hilt, Retrofit, Room, Coroutines/Flow, Navigation Compose, FCM
- 서버: Spring Boot(Kotlin), PostgreSQL, Flyway, Spring Data JPA
- 인증/푸시: Firebase Auth, FCM
- 빌드: Gradle Kotlin DSL, `libs.versions.toml` 버전 카탈로그
