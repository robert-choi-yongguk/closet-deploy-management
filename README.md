# 🚀 Closet Deploy Management

[![GitHub Actions Workflow Status](https://github.com/robert-choi-yongguk/closet-deploy-management/actions/workflows/create-release-pr.yml/badge.svg)](https://github.com/robert-choi-yongguk/closet-deploy-management/actions/workflows/create-release-pr.yml)

중앙화된 릴리즈 이력 관리 및 배포 자동화 시스템입니다. 여러 개의 **운영 레포지토리**에서 발생하는 릴리즈 내역을 **이력 관리 레포지토리**(현재 레포지토리)에서 통합 관리하며, 클릭 한 번으로 여러 레포지토리의 배포 PR을 동시에 생성할 수 있습니다.

## 🏛️ 시스템 아키텍처

이 시스템은 두 종류의 레포지토리와 세 개의 GitHub Actions 워크플로우로 구성됩니다.

- **이력 관리 레포지토리 (현재 레포지토리):**
  - 모든 릴리즈 이력 로그가 저장되는 중앙 허브입니다.
  - 통합 배포 PR 생성을 트리거하는 워크플로우(`create-release-pr.yml`)가 위치합니다.

- **운영 레포지토리 (N개):**
  - 실제 서비스 코드가 관리되는 대상 레포지토리입니다.
  - 릴리즈 이력을 기록하고, 배포 완료 후 로그를 정리하는 워크플로우(`record-release-history.yml`, `cleanup-release-log.yml`)가 위치합니다.

## ⚙️ 동작 방식 (Workflow)

**Step 1: 릴리즈 브랜치에 PR 병합 (운영 레포지토리)**
- 개발자가 운영 레포지토리의 `release` 브랜치에 기능 PR을 병합합니다.

**Step 2: 릴리즈 이력 자동 기록**
- `record-release-history.yml` 워크플로우가 자동으로 실행됩니다.
- 병합된 PR 번호와 작성자 정보가 이력 관리 레포지토리의 `pending/소유자/레포/history.log` 파일에 한 줄 추가됩니다.

**Step 3: 통합 배포 PR 생성 (이력 관리 레포지토리)**
- 관리자가 이력 관리 레포지토리의 `Actions` 탭에서 `통합 배포 PR 수동 생성` 워크플로우를 수동으로 실행합니다.
- `create-release-pr.yml` 워크플로우가 실행되어 `pending` 디렉토리를 스캔합니다.
- 로그가 기록된 모든 운영 레포지토리에 대해, `release` -> `main`으로 향하는 배포 PR을 각각 자동으로 생성합니다.
- 생성된 PR은 `YYYY-MM-DD Release` 형식의 제목과 `Deployment` 라벨을 갖게 됩니다.

**Step 4: 배포 및 로그 아카이빙**
- 운영 레포지토리에서 생성된 배포 PR이 `main` 브랜치에 병합되어 실제 배포가 완료됩니다.
- `cleanup-release-log.yml` 워크플로우가 자동으로 실행됩니다.
- `pending`에 있던 로그 파일을 `processed` 디렉토리로 옮겨 영구 보관(아카이빙)합니다.

## 🛠️ 설치 및 설정 가이드

### 1. Personal Access Token (PAT) 설정

워크플로우가 레포지토리 간에 상호작용하려면 PAT이 필요합니다.

1.  **토큰 생성:** [Fine-grained Personal Access Token](https://github.com/settings/tokens?type=beta)으로 토큰을 생성합니다.
2.  **권한 설정:**
    - **이력 관리 레포지토리 (`closet-deploy-management`) 대상:**
      - `Contents`: `Read and write`
    - **모든 운영 레포지토리 대상:**
      - `Pull requests`: `Read and write` (PR 생성 및 라벨 추가를 위해 필요)
      - `Contents`: `Read-only`
3.  **시크릿 등록:**
    - 생성한 토큰 값을 복사하여 `GH_RELEASE_PR_PAT` 라는 이름의 시크릿으로 등록합니다.
    - **중요:** 이 시크릿은 **이력 관리 레포지토리**와 **모든 운영 레포지토리** 양쪽에 모두 등록해야 합니다.

### 2. 워크플로우 파일 배치

각 레포지토리의 `.github/workflows/` 디렉토리 아래에 다음 파일들을 배치합니다.

- **이력 관리 레포지토리 (`closet-deploy-management`):**
  - `create-release-pr.yml`

- **모든 운영 레포지토리:**
  - `record-release-history.yml`
  - `cleanup-release-log.yml`

### 3. GitHub 라벨 설정

- **운영 레포지토리**에는 `Deployment` 라는 이름의 라벨이 있는 것이 좋습니다.
- 라벨이 없어도 워크플로우가 PR 생성 시 자동으로 추가해주지만, 미리 생성해두면 색상 등을 원하는 대로 설정할 수 있어 관리에 용이합니다.

## ▶️ 사용법

1. 각 운영 레포지토리에서 `release` 브랜치로 PR을 병합하며 릴리즈를 준비합니다.
2. 배포 시점이 되면, **현재 레포지토리 (`closet-deploy-management`)**의 `Actions` 탭으로 이동합니다.
3. 좌측 메뉴에서 `통합 배포 PR 수동 생성` 워크플로우를 선택합니다.
4. `Run workflow` 버튼을 클릭하여 실행합니다.
5. 잠시 후, 릴리즈 내역이 있던 모든 운영 레포지토리에 배포 PR이 생성된 것을 확인할 수 있습니다.

## 📄 라이선스

MIT License


### 1. Personal Access Token (PAT) 설정

워크플로우가 레포지토리 간에 상호작용하려면 PAT이 필요합니다.

1.  **토큰 생성:** [Fine-grained Personal Access Token](https://github.com/settings/tokens?type=beta)으로 토큰을 생성합니다.
2.  **권한 설정:**
    - **이력 관리 레포지토리 (`closet-deploy-management`) 대상:**
      - `Contents`: `Read and write`
    - **모든 운영 레포지토리 대상:**
      - `Pull requests`: `Read and write`
      - `Contents`: `Read-only`
3.  **시크릿 등록:**
    - 생성한 토큰 값을 복사하여 `GH_RELEASE_PR_PAT` 라는 이름의 시크릿으로 등록합니다.
    - **중요:** 이 시크릿은 **이력 관리 레포지토리**와 **모든 운영 레포지토리** 양쪽에 모두 등록해야 합니다.

### 2. 워크플로우 파일 배치

각 레포지토리의 `.github/workflows/` 디렉토리 아래에 다음 파일들을 배치합니다.

- **이력 관리 레포지토리 (`closet-deploy-management`):**
  - `create-release-pr.yml`

- **모든 운영 레포지토리:**
  - `record-release-history.yml`
  - `cleanup-release-log.yml`

## ▶️ 사용법

1. 각 운영 레포지토리에서 `release` 브랜치로 PR을 병합하며 릴리즈를 준비합니다.
2. 배포 시점이 되면, **현재 레포지토리 (`closet-deploy-management`)**의 `Actions` 탭으로 이동합니다.
3. 좌측 메뉴에서 `통합 배포 PR 수동 생성` 워크플로우를 선택합니다.
4. `Run workflow` 버튼을 클릭하여 실행합니다.
5. 잠시 후, 릴리즈 내역이 있던 모든 운영 레포지토리에 배포 PR이 생성된 것을 확인할 수 있습니다.

## 📄 라이선스

MIT License