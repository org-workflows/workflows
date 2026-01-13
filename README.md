# org-workflows

조직 전체에서 공유하는 재사용 가능한 GitHub Actions 워크플로우 모음

## 사용 가능한 워크플로우

| 워크플로우 | 설명 | 버전 |
|-----------|------|------|
| [reusable-ci.yml](.github/workflows/reusable-ci.yml) | CI 파이프라인 (lint, test, build) | v1 |
| [reusable-deploy.yml](.github/workflows/reusable-deploy.yml) | 배포 파이프라인 | v1 |
| [reusable-security.yml](.github/workflows/reusable-security.yml) | 보안 스캔 | v1 |
| [reusable-code-review.yml](.github/workflows/reusable-code-review.yml) | AI 코드 리뷰 (Claude + OpenAI) | v1 |

## 빠른 시작

### CI 파이프라인

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1
    with:
      node-version: '20'
      coverage-threshold: 80
    secrets: inherit
```

### 배포 파이프라인

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: org-workflows/workflows/.github/workflows/reusable-deploy.yml@v1
    with:
      environment: production
      url: https://example.com
    secrets: inherit
```

### 보안 스캔

```yaml
# .github/workflows/security.yml
name: Security

on:
  schedule:
    - cron: '0 0 * * 1'  # 매주 월요일
  workflow_dispatch:

jobs:
  security:
    uses: org-workflows/workflows/.github/workflows/reusable-security.yml@v1
    secrets: inherit
```

### AI 코드 리뷰

```yaml
# .github/workflows/code-review.yml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    uses: org-workflows/workflows/.github/workflows/reusable-code-review.yml@v1
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

## Composite Actions

### setup-node

Node.js 환경을 설정하고 의존성을 캐시합니다.

```yaml
steps:
  - uses: org-workflows/workflows/.github/actions/setup-node@v1
    with:
      node-version: '20'
      package-manager: 'npm'  # npm, yarn, pnpm
```

## 버전 관리

| 태그 | 설명 |
|-----|------|
| `@v1` | 최신 안정 버전 (권장) |
| `@v1.0.0` | 특정 버전 고정 |
| `@main` | 최신 개발 버전 |

```yaml
# 권장: 메이저 버전 사용
uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1

# 버전 고정
uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1.0.0
```

## 필수 시크릿

프로젝트에서 다음 시크릿을 설정하세요:

| 시크릿 | 용도 | 필수 |
|-------|------|------|
| `ANTHROPIC_API_KEY` | Claude AI 코드 리뷰 | 코드 리뷰 시 |
| `OPENAI_API_KEY` | OpenAI 코드 리뷰 | 코드 리뷰 시 |
| `SLACK_WEBHOOK` | Slack 알림 | 선택 |
| `NPM_TOKEN` | npm 패키지 배포 | 선택 |

## 문서

- [상세 사용 가이드](docs/usage.md)
- [마이그레이션 가이드](docs/migration.md)
- [기여 가이드](CONTRIBUTING.md)

## 라이선스

MIT License
