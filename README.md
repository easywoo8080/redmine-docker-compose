# redmine-docker-compose
레드마인 도커 컴포즈 생성 2025-04-29일 작업 확인

# Redmine Docker Compose 설정

이 문서는 Redmine을 Docker와 Docker Compose를 사용하여 설정하는 방법을 안내합니다.

## 1. 요구 사항

- Docker
- Docker Compose
- PostgreSQL 데이터베이스
- SMTP 서버 (메일 전송용)

## 2. 프로젝트 파일 구조

```bash
.
├── .env                # 환경 변수 파일
├── docker-compose.yml  # Docker Compose 설정 파일
└── config
    └── configuration.yml  # Redmine 설정 파일
```

## 3. Redmine Docker Compose 설정 (PostgreSQL 연결)

이 문서는 **Redmine 6.0.5**와 **PostgreSQL**을 연결하는 Docker Compose 설정을 설명합니다.

## 1. docker-compose.yml 파일 구조

```yaml
services:
  redmine:
    image: redmine:6.0.5  # 사용할 Redmine 도커 이미지 버전
    container_name: redmine  # 컨테이너 이름 지정
    ports:
      - "${REDMINE_PORT}:3000"  # 호스트 포트는 ${REDMINE_PORT} (예: 8588)
    environment:
      REDMINE_DB_ADAPTER: postgresql  # 데이터베이스 어댑터 (PostgreSQL 사용)
      REDMINE_DB_POSTGRES: "${DB_HOST}"  # PostgreSQL 데이터베이스 호스트 (예: db_host)
      REDMINE_DB_PORT: "${DB_PORT}"  # PostgreSQL 포트 (예: 5432)
      REDMINE_DB_USERNAME: "${DB_USER}"  # PostgreSQL 사용자명 (예: redmine_user)
      REDMINE_DB_PASSWORD: "${DB_PASS}"  # PostgreSQL 비밀번호 (예: redmine_password)
      REDMINE_DB_DATABASE: "${DB_NAME}"  # PostgreSQL 데이터베이스 이름 (예: redmine_db)
    volumes:
      - redmine-data:/usr/src/redmine/files  # Redmine 파일 저장소 마운트
      - ./config/configuration.yml:/usr/src/redmine/config/configuration.yml  # Redmine 설정 파일 마운트
      - ./plugins:/usr/src/redmine/plugins  # Redmine 플러그인 디렉토리 마운트

volumes:
  redmine-data:
    driver: local  # 로컬 볼륨 사용
```


## 4. Redmine Email Delivery 설정 (SMTP) (configuration.yml)

이 문서는 Redmine의 `production` 환경에서 이메일 발송을 설정하는 방법을 설명합니다. 이메일 발송을 위해 SMTP 서버를 사용하며, Gmail을 예시로 설명합니다.

- 이메일 발송 설정

```yaml
production:
  email_delivery:
    delivery_method: :smtp  # 메일 발송 방식 (SMTP 사용)
    smtp_settings:
      address: <%= ENV['SMTP_ADDRESS'] %>  # SMTP 서버 주소 (Gmail은 smtp.gmail.com)
      port: <%= ENV['SMTP_PORT'] %>        # SMTP 포트 (Gmail은 587)
      domain: <%= ENV['SMTP_DOMAIN'] %>    # SMTP 서버 도메인 (smtp.gmail.com 또는 gmail.com)
      authentication: :plain               # 인증 방식 (Gmail은 plain)
      user_name: <%= ENV['SMTP_USER'] %>   # Gmail 주소 (예: yourname@gmail.com)
      password: <%= ENV['SMTP_PASS'] %>    # Gmail 앱 비밀번호 (2단계 인증 시 앱 비밀번호 사용)
      enable_starttls_auto: true           # TLS 암호화 사용 여부 (Gmail은 true)
```


## 5. .env 파일 설정

이 파일은 Docker Compose와 Redmine의 이메일 발송 설정에 필요한 환경 변수들을 정의합니다. `.env` 파일을 사용하여 민감한 정보(비밀번호, 사용자명 등)를 안전하게 관리하고, 설정 파일에서 이 값을 불러올 수 있습니다.

- .env 파일 예시

```env
# Redmine 포트 설정
REDMINE_PORT=your_port  # Redmine 웹 애플리케이션이 호스트에서 접근 가능한 포트

# PostgreSQL 연결 설정
DB_HOST=db_host  # PostgreSQL 데이터베이스 호스트
DB_PORT=5432     # PostgreSQL 포트 (기본값: 5432)
DB_USER=redmine_user  # PostgreSQL 사용자명
DB_PASS=redmine_password  # PostgreSQL 비밀번호
DB_NAME=redmine_db  # PostgreSQL 데이터베이스 이름

# SMTP 서버 설정 (Gmail 예시)
SMTP_ADDRESS=smtp.gmail.com  # SMTP 서버 주소 (Gmail)
SMTP_PORT=587               # SMTP 포트 (Gmail은 587)
SMTP_DOMAIN=gmail.com        # SMTP 서버 도메인
SMTP_USER=yourname@gmail.com  # Gmail 사용자명 (이메일 주소)
SMTP_PASS=your_app_password  # Gmail 앱 비밀번호 (2단계 인증 사용 시 앱 비밀번호)
```
