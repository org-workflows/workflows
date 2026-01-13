# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2024-01-XX

### Added

- **reusable-ci.yml**: CI 파이프라인
  - Lint, Test, Build jobs
  - Node.js 버전 선택 (기본: 20)
  - 패키지 매니저 선택 (npm, yarn, pnpm)
  - 커버리지 임계값 설정
  - 모노레포 지원 (working-directory)

- **reusable-deploy.yml**: 배포 파이프라인
  - Environment 보호 지원
  - 헬스체크 자동 실행
  - Slack 알림
  - 아티팩트 다운로드

- **reusable-security.yml**: 보안 스캔
  - npm audit 취약점 스캔
  - 시크릿 패턴 스캔
  - 임계값 기반 실패 처리
  - Slack 알림

- **reusable-code-review.yml**: AI 코드 리뷰
  - Claude 보안 리뷰
  - OpenAI 품질 리뷰
  - 리뷰 요약 코멘트
  - Slack 알림

- **setup-node Action**: Node.js 환경 설정
  - 다중 패키지 매니저 지원
  - 자동 캐싱
  - 모노레포 지원

### Security

- 모든 워크플로우에 최소 권한 원칙 적용
- 시크릿 스캔 패턴 추가
