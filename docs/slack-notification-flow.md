# 슬랙 알림 워크플로우 순서도

이 다이어그램은 'notify-pending-releases.yml' 워크플로우의 내부 로직을 시각적으로 설명합니다.

```mermaid
flowchart TD
    A[Start: 'notify-pending-releases.yml' 실행] --> B{pending/ 디렉토리에<br>history.log 파일이 있는가?};
    B -- "Yes" --> C[MESSAGE_TEXT 변수 초기화<br>&quot;🚀 릴리즈 대기중인 항목 알림 🚀&quot;];
    B -- "No" --> D[MESSAGE =<br>&quot;✅ 릴리즈 대기중인 항목이 없습니다.&quot;];
    D --> K[Slack으로 MESSAGE 전송];

    C --> E[log 파일 목록 순회 시작];
    E --> F[레포지토리/소유자 정보 파싱 및 추가];
    F --> G[log 파일 내용 한 줄씩 순회 시작];
    G --> H[PR번호, 제목, 작성자 파싱];
    H --> I[Slack 멘션 ID 조회];
    I --> J[파싱된 PR 정보를<br>MESSAGE_TEXT에 추가];
    J --> G;
    G -- "파일 끝" --> E;
    E -- "모든 로그 파일 완료" --> L[MESSAGE_TEXT를 JQ로<br>JSON 포맷(MESSAGE)으로 변환];
    L --> K;
    K --> Z[End];
```
