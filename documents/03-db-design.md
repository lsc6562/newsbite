# 03. DB 설계 규정

## 기본
- 서버 DB: PostgreSQL. 스키마 변경은 **Flyway 마이그레이션**으로만 한다 (`server/src/main/resources/db/migration/V{번호}__{설명}.sql`).
- **이미 적용된 마이그레이션 파일을 수정하지 않는다.** 변경은 새 파일로 추가한다.
- JPA `ddl-auto`는 `validate` 또는 `none`. 자동 생성 금지.
- 앱 로컬 DB(Room)는 캐시·북마크 용도이며, 서버 DB와 별개로 관리한다. 스키마 변경 시 Migration을 작성한다.

## 네이밍
- 테이블·컬럼: `snake_case`, 테이블은 복수형 (`users`, `articles`).
- PK: `id` (bigint identity). FK: `{대상단수}_id`.
- 불리언은 `is_` 접두어, 시각은 `_at` 접미어.
- 모든 테이블에 `created_at`, `updated_at` (`timestamptz`, UTC)를 둔다.

## 핵심 테이블
| 테이블 | 주요 컬럼 | 비고 |
|---|---|---|
| `users` | id, firebase_uid(unique), nickname | 이메일 등 최소 정보만 |
| `categories` | id, code(unique), name | 국내/해외, 정치, 경제, 사회, 스포츠 등 |
| `user_categories` | user_id, category_id | PK(user_id, category_id) |
| `delivery_channels` | id, user_id, type, target_encrypted, is_enabled, send_time | type: TELEGRAM 등 |
| `articles` | id, source, title, url(unique), published_at, category_id, content_hash | **본문 저장 금지** |
| `summaries` | id, article_id, text, model, created_at | 항상 AI 생성물 |
| `delivery_logs` | id, user_id, summary_id, channel_type, status, sent_at | 중복 발송 방지에 사용 |
| `bookmarks` | user_id, summary_id | |
| `summary_reports` | id, summary_id, user_id, reason | 오류 신고 |

## 규칙
- **기사 본문은 DB, 파일, 로그 어디에도 저장하지 않는다.** 요약 생성 중 메모리에서만 쓴다. 저장 대상은 제목, URL, 메타데이터, 요약문뿐이다.
- 사용자 소유 데이터의 FK는 `ON DELETE CASCADE`로 설계해 탈퇴 시 완전 삭제되도록 한다.
- 개인정보는 soft delete를 쓰지 않는다 (hard delete).
- 중복 방지는 DB 제약으로 보장한다 (예: `articles.url` unique, `delivery_logs`에 `(user_id, summary_id, channel_type)` unique).
- 자주 조회하는 조건(`category_id`, `published_at`, `user_id`)에는 인덱스를 둔다. 인덱스는 필요가 확인된 곳에만 추가한다.
- 오래된 기사·요약·발송 로그는 보관 기간(기본 30일)이 지나면 삭제하는 정리 작업을 둔다.
- 쿼리에서 `SELECT *`와 N+1 조회를 피한다.
- 민감 컬럼(`target_encrypted`)은 애플리케이션 레벨에서 암호화하며, 키는 환경 변수로 관리한다.
