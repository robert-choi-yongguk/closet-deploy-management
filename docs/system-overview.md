# 전체 시스템 흐름도

이 다이어그램은 중앙 이력 관리 시스템의 전체 구성 요소와 워크플로우 간의 상호작용을 보여줍니다.

```mermaid
graph TD
    subgraph "운영 레포지토리 (N개)"
        direction LR
        Dev[개발자] -- "1. 기능 PR 병합" --> ReleaseBranch(release 브랜치)
        ReleaseBranch -- "Trigger" --> RecordHistory(record-release-history.yml)
        MainBranch(main 브랜치) -- "Trigger" --> CleanupLog(cleanup-release-log.yml)
    end

    subgraph "이력 관리 레포지토리"
        direction LR
        Admin[관리자] -- "수동 실행" --> CreatePR(create-release-pr.yml)
        Admin2[관리자] -- "수동 실행" --> Notify(notify-pending-releases.yml)
        PendingLogs(pending/ 디렉토리)
        ProcessedLogs(processed/ 디렉토리)
    end

    RecordHistory -- "2. 릴리즈 이력 기록 (Write)" --> PendingLogs
    CreatePR -- "3. 이력 조회 (Read)" --> PendingLogs
    CreatePR -- "4. 배포 PR 생성" --> OperationalRepoPR[운영 레포지토리의 PR]
    CleanupLog -- "5. 이력 아카이빙 (Move)" --> PendingLogs
    CleanupLog -- " " --> ProcessedLogs
    Notify -- "릴리즈 대기 항목 조회 (Read)" --> PendingLogs
    Notify -- "Slack 알림" --> Slack[(Slack)]

    style Dev fill:#f9f,stroke:#333,stroke-width:2px
    style Admin fill:#f9f,stroke:#333,stroke-width:2px
    style Admin2 fill:#f9f,stroke:#333,stroke-width:2px
```
