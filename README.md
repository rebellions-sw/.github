# Rebellions SW — Org Standards

이 repo 는 **조직 차원의 단일 진실 (single source of truth)** 입니다.

## 무엇이 있나

| 디렉토리 | 용도 |
|---------|------|
| `.github/workflows/reusable-*.yaml` | 모든 service repo 가 호출하는 reusable GitHub Actions workflow |
| `org-templates/base-app/`           | 신규 application repo 가 복사하는 표준 파일 (CODEOWNERS, pre-commit, PR template, …) |

## Reusable Workflows

| Workflow | 용도 | 호출 측 |
|----------|------|---------|
| `reusable-build.yaml`            | Build (Kaniko) → SBOM(Syft) → Trivy → Cosign → Harbor push → manifest dev tag bump | `<svc>` (app code repo) |
| `reusable-promote.yaml`          | overlay tag 복사 (dev→stage→prod) → PR 생성 | `<svc>-manifests` |
| `reusable-manifest-validate.yaml`| kustomize build + kubeconform + conftest | `<svc>-manifests` PR |

### Runner 표준

기본 라벨: `[self-hosted, rebel-k8s-runner]` (ARC scale set, 8대 — 자동 scale).
재정의: 호출시 `with.runs-on: '["self-hosted","rbcn"]'` (단독 VM 1대) 가능.

### 호출 예 — application repo (`.github/workflows/ci.yml`)

```yaml
name: ci
on:
  push:    { branches: [main, dev] }
  pull_request:
jobs:
  build:
    uses: rebellions-sw/.github/.github/workflows/reusable-build.yaml@main
    with:
      service: my-service
      language: go
    secrets: inherit
```

### 호출 예 — manifests repo (`.github/workflows/promote.yml`)

```yaml
name: promote
on:
  workflow_dispatch:
    inputs:
      from: { type: choice, options: [dev, stage] }
      to:   { type: choice, options: [stage, prod] }
jobs:
  promote:
    uses: rebellions-sw/.github/.github/workflows/reusable-promote.yaml@main
    with:
      from: ${{ inputs.from }}
      to:   ${{ inputs.to }}
    secrets: inherit
```

### 호출 예 — manifests repo PR 검증 (`.github/workflows/validate.yml`)

```yaml
name: validate
on: [pull_request]
jobs:
  v:
    uses: rebellions-sw/.github/.github/workflows/reusable-manifest-validate.yaml@main
```

## Required Org Secrets

조직 secret 으로 다음을 등록하면 `secrets: inherit` 로 모든 caller 가 자동 사용:

| Secret | 용도 |
|--------|------|
| `HARBOR_USER`         | Harbor registry user (robot account 권장) |
| `HARBOR_PASS`         | Harbor registry password |
| `MANIFESTS_PAT`       | manifests repo write 가능한 fine-grained PAT |
| `COSIGN_PRIVATE_KEY`  | (선택) keyless 가 안 될 때 fallback 서명 키 |
| `COSIGN_PASSWORD`     | (선택) cosign key passphrase |

## org-templates/base-app

`rbcn new <svc> --type=…` 가 새 repo 부트스트랩 시 이 디렉토리를 복사합니다.

| 파일 | 용도 |
|------|------|
| `.github/CODEOWNERS`              | 코드 소유자 |
| `.github/PULL_REQUEST_TEMPLATE.md`| PR template |
| `.github/ISSUE_TEMPLATE/bug.yaml` | 버그 issue template |
| `.pre-commit-config.yaml`         | local hook (lint / secrets / format) |
| `commitlint.config.js`            | conventional commits |
| `.editorconfig`, `.markdownlint.json` | 포매팅 |
| `SECURITY.md`                     | 보안 정책 |
| `README-template.md`              | README 시작점 |

## 버전 정책

- `@main` → 최신 (default).
- `@vX.Y.Z` → 안정성 필요시 tag 사용.

문의: `#platform`
