# org-workflows

조직 전체에서 공유하는 GitHub Actions 워크플로우 중앙 저장소

## 구조

```
org-workflows/workflows/
├── .github/
│   ├── workflows/           # Reusable Workflows (호출용)
│   │   ├── reusable-ci.yml
│   │   ├── reusable-deploy.yml
│   │   ├── reusable-security.yml
│   │   └── reusable-code-review.yml
│   └── actions/             # Composite Actions
│       └── setup-node/
│
├── templates/               # 템플릿 워크플로우 (복사용)
│   ├── 01-issue-triage.yml
│   ├── 02-issue-analysis.yml
│   ├── 03-code-review.yml
│   ├── 04-ci.yml
│   ├── 05-migration-check.yml
│   ├── 06-deploy-staging.yml
│   ├── 07-deploy-production.yml
│   ├── 08-release.yml
│   ├── 09-dependency-check.yml
│   └── 10-doc-sync.yml
│
└── examples/                # 사용 예제
```

---

## 사용 방법

### 방법 1: Reusable Workflow 호출 (권장)

프로젝트에서 직접 호출하여 사용:

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  ci:
    uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1
    with:
      node-version: '20'
    secrets: inherit
```

### 방법 2: 템플릿 복사

`templates/` 폴더에서 필요한 워크플로우를 복사하여 사용:

```bash
# 전체 템플릿 복사
curl -sL https://github.com/org-workflows/workflows/archive/refs/heads/main.zip -o workflows.zip
unzip workflows.zip "*/templates/*" -d temp
cp -r temp/*/templates/*.yml .github/workflows/
rm -rf workflows.zip temp
```

---

## Reusable Workflows

### reusable-ci.yml

CI 파이프라인 (lint, test, build)

```yaml
uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1
with:
  node-version: '20'           # Node.js 버전
  package-manager: 'npm'       # npm, yarn, pnpm
  working-directory: '.'       # 모노레포용
  run-lint: true
  run-tests: true
  run-build: true
  coverage-threshold: 80       # 커버리지 임계값
secrets: inherit
```

### reusable-deploy.yml

배포 파이프라인

```yaml
uses: org-workflows/workflows/.github/workflows/reusable-deploy.yml@v1
with:
  environment: 'staging'       # staging, production
  url: 'https://example.com'
  run-health-check: true
  health-check-path: '/health'
  notify-slack: true
secrets:
  SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### reusable-security.yml

보안 스캔

```yaml
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

---

## 템플릿 워크플로우

| 파일 | 설명 | 트리거 |
|-----|------|-------|
| `01-issue-triage.yml` | 이슈 자동 분류 (Claude) | Issue 생성 |
| `02-issue-analysis.yml` | 이슈 분석 (OpenAI) | Issue 할당 |
| `03-code-review.yml` | AI 코드 리뷰 | PR 생성/업데이트 |
| `04-ci.yml` | CI 파이프라인 | Push/PR |
| `05-migration-check.yml` | DB 마이그레이션 검증 | PR (migrations/) |
| `06-deploy-staging.yml` | 스테이징 배포 | develop push |
| `07-deploy-production.yml` | 프로덕션 배포 | Release 발행 |
| `08-release.yml` | 릴리즈 자동화 | 태그 생성 |
| `09-dependency-check.yml` | 의존성 보안 스캔 | 주간 스케줄 |
| `10-doc-sync.yml` | 문서 동기화 체크 | PR (API/타입 변경) |

---

## 필수 시크릿

프로젝트에서 다음 시크릿을 설정하세요:

| 시크릿 | 용도 | 필수 |
|-------|------|------|
| `ANTHROPIC_API_KEY` | Claude AI | 코드 리뷰/이슈 분류 |
| `OPENAI_API_KEY` | OpenAI | 코드 리뷰/분석 |
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
| `@v1.0.0` | 특정 버전 고정 |
| `@main` | 최신 개발 버전 |

```yaml
# 권장: 메이저 버전
uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1

# 버전 고정
uses: org-workflows/workflows/.github/workflows/reusable-ci.yml@v1.0.0
```

---

## 새 프로젝트 시작하기

```bash
# 1. 프로젝트 생성
mkdir my-project && cd my-project
git init

# 2. 워크플로우 폴더 생성
mkdir -p .github/workflows

# 3. 기본 워크플로우 생성
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

# 4. 코드 리뷰 워크플로우 (선택)
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
