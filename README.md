# org-workflows

조직 전체에서 공유하는 GitHub Actions Reusable Workflows 중앙 저장소

## 구조

```
org-workflows/workflows/
└── .github/
    ├── workflows/              # Reusable Workflows
    │   ├── reusable-ci.yml
    │   ├── reusable-deploy.yml
    │   ├── reusable-security.yml
    │   ├── reusable-code-review.yml
    │   ├── reusable-issue-triage.yml
    │   ├── reusable-issue-analysis.yml
    │   ├── reusable-migration-check.yml
    │   ├── reusable-release.yml
    │   └── reusable-doc-sync.yml
    └── actions/                # Composite Actions
        └── setup-node/
```

---

## 워크플로우 목록

| 워크플로우 | 설명 | AI 사용 |
|-----------|------|---------|
| `reusable-ci.yml` | CI 파이프라인 (lint, test, build) | - |
| `reusable-deploy.yml` | 배포 파이프라인 | - |
| `reusable-security.yml` | 보안 스캔 | - |
| `reusable-code-review.yml` | AI 코드 리뷰 | Claude + OpenAI |
| `reusable-issue-triage.yml` | 이슈 자동 분류 | Claude |
| `reusable-issue-analysis.yml` | 이슈 분석 | OpenAI |
| `reusable-migration-check.yml` | DB 마이그레이션 검증 | Claude |
| `reusable-release.yml` | 릴리즈 자동화 | Claude |
| `reusable-doc-sync.yml` | 문서 동기화 체크 | OpenAI |

---

## 사용 방법

### reusable-ci.yml

CI 파이프라인 (lint, test, build)

```yaml
name: CI
on: [push, pull_request]

jobs:
  ci:
    uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1
    with:
      node-version: '20'
      run-lint: true
      run-tests: true
      run-build: true
      coverage-threshold: 80
    secrets: inherit
```

### reusable-deploy.yml

배포 파이프라인

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: org-workflows/workflows/.github/workflows/reusable-deploy.yml@v1
    with:
      environment: staging
      url: 'https://staging.example.com'
      run-health-check: true
      health-check-path: '/health'
      notify-slack: true
    secrets:
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### reusable-security.yml

보안 스캔

```yaml
name: Security
on:
  schedule:
    - cron: '0 0 * * 1'  # 매주 월요일

jobs:
  security:
    uses: org-workflows/workflows/.github/workflows/reusable-security.yml@v1
    with:
      run-npm-audit: true
      run-secret-scan: true
      fail-on-high: true
      notify-slack: true
    secrets:
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### reusable-code-review.yml

AI 코드 리뷰 (Claude + OpenAI)

```yaml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    uses: org-workflows/workflows/.github/workflows/reusable-code-review.yml@v1
    with:
      run-security-review: true    # Claude 보안 리뷰
      run-quality-review: true     # OpenAI 품질 리뷰
      notify-slack: true
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### reusable-issue-triage.yml

이슈 자동 분류 (Claude)

```yaml
name: Issue Triage
on:
  issues:
    types: [opened]

jobs:
  triage:
    uses: org-workflows/workflows/.github/workflows/reusable-issue-triage.yml@v1
    with:
      issue-number: ${{ github.event.issue.number }}
      issue-title: ${{ github.event.issue.title }}
      issue-body: ${{ github.event.issue.body }}
      issue-url: ${{ github.event.issue.html_url }}
      notify-slack: true
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### reusable-issue-analysis.yml

이슈 분석 (OpenAI)

```yaml
name: Issue Analysis
on:
  issues:
    types: [assigned]

jobs:
  analyze:
    uses: org-workflows/workflows/.github/workflows/reusable-issue-analysis.yml@v1
    with:
      issue-number: ${{ github.event.issue.number }}
      issue-title: ${{ github.event.issue.title }}
      issue-body: ${{ github.event.issue.body }}
      assignee: ${{ github.event.assignee.login }}
    secrets:
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

### reusable-migration-check.yml

DB 마이그레이션 검증 (Claude)

```yaml
name: Migration Check
on:
  pull_request:
    paths:
      - 'migrations/**'
      - 'prisma/migrations/**'

jobs:
  migration:
    uses: org-workflows/workflows/.github/workflows/reusable-migration-check.yml@v1
    with:
      pr-number: ${{ github.event.number }}
      generate-rollback: true
      commit-rollback: true
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### reusable-release.yml

릴리즈 자동화 (Claude)

```yaml
name: Release
on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  release:
    uses: org-workflows/workflows/.github/workflows/reusable-release.yml@v1
    with:
      tag-name: ${{ github.ref_name }}
      draft: true
      generate-changelog: true
      notify-slack: true
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### reusable-doc-sync.yml

문서 동기화 체크 (OpenAI)

```yaml
name: Doc Sync
on:
  pull_request:
    paths:
      - 'src/api/**'
      - 'src/types/**'
      - '*.config.*'

jobs:
  docs:
    uses: org-workflows/workflows/.github/workflows/reusable-doc-sync.yml@v1
    with:
      pr-number: ${{ github.event.number }}
      stale-days: 30
    secrets:
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

---

## 필수 시크릿

프로젝트에서 다음 시크릿을 설정하세요:

| 시크릿 | 용도 | 필수 |
|-------|------|------|
| `ANTHROPIC_API_KEY` | Claude AI | 코드 리뷰, 이슈 분류, 마이그레이션, 릴리즈 |
| `OPENAI_API_KEY` | OpenAI | 코드 리뷰, 이슈 분석, 문서 체크 |
| `SLACK_WEBHOOK` | Slack 알림 | 선택 |
| `NPM_TOKEN` | npm 배포 | 선택 |

### 필수 변수

| 변수 | 용도 |
|-----|------|
| `SLACK_ENABLED` | Slack 알림 활성화 (`true`/`false`) |

---

## 버전 관리

| 태그 | 설명 |
|-----|------|
| `@v1` | 최신 안정 버전 (권장) |
| `@v2.0.0` | 특정 버전 고정 |
| `@main` | 최신 개발 버전 |

```yaml
# 권장: 메이저 버전
uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1

# 버전 고정
uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v2.0.0
```

---

## 새 프로젝트 빠른 시작

```bash
# 1. 프로젝트 생성
mkdir my-project && cd my-project
git init

# 2. 워크플로우 폴더 생성
mkdir -p .github/workflows

# 3. CI 워크플로우 생성
cat > .github/workflows/ci.yml << 'EOF'
name: CI
on: [push, pull_request]

jobs:
  ci:
    uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1
    with:
      node-version: '20'
    secrets: inherit
EOF

# 4. 코드 리뷰 워크플로우 생성
cat > .github/workflows/code-review.yml << 'EOF'
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
EOF
```

---

## 라이선스

MIT License
