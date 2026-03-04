# Commit Review Hook - Refactoring Requirements

## 1. 파일 구조 변경

```
review-commit-hook/
├── install                      # 설치 스크립트
├── uninstall                    # 제거 스크립트
├── pre-commit-hook              # .git/hooks/pre-commit에 삽입될 진입점
├── request-review-to-user       # 전략 오케스트레이터
└── strategy/
    ├── 1-ubuntu-zenity          # Zenity GUI 전략
    └── 2-ubuntu-terminal        # Terminal 프롬프트 전략
```

## 2. 확장자 제거

모든 스크립트에서 `.sh` 확장자 제거 (일관성 유지)

## 3. 책임 분리

| 파일 | 책임 | Exit Code |
|------|------|-----------|
| `pre-commit-hook` | `.git/hooks/pre-commit`에 삽입될 진입점, `request-review-to-user` 호출 | - |
| `request-review-to-user` | strategy/ 폴더의 전략을 순서대로 시도 | 0/1 전달 |
| `strategy/1-ubuntu-zenity` | Ubuntu + Zenity GUI 다이얼로그 | 0=APPROVED, 1=REJECTED, 2=UNAVAILABLE |
| `strategy/2-ubuntu-terminal` | Ubuntu + Terminal 프롬프트 | 0=APPROVED, 1=REJECTED, 2=UNAVAILABLE |

## 4. `install` 스크립트

```
1. git repository 확인
2. 설치 경로 질문 (기본값: utility/hook/)
3. 설치 경로 디렉토리 생성
4. 파일 복사:
   - {TARGET}/pre-commit-hook
   - {TARGET}/request-review-to-user
   - {TARGET}/strategy/1-ubuntu-zenity
   - {TARGET}/strategy/2-ubuntu-terminal
5. .git/hooks/pre-commit 생성/수정:
   - 설치 경로를 변수로 저장 (uninstall에서 참조용)
   - {TARGET}/pre-commit-hook 실행하도록 설정
```

## 5. `uninstall` 스크립트

```
1. git repository 확인
2. .git/hooks/pre-commit에서 저장된 설치 경로 변수 읽기
3. 해당 경로의 파일들 삭제:
   - {TARGET}/pre-commit-hook
   - {TARGET}/request-review-to-user
   - {TARGET}/strategy/1-ubuntu-zenity
   - {TARGET}/strategy/2-ubuntu-terminal
4. .git/hooks/pre-commit에서 관련 코드 제거
5. 빈 strategy/ 디렉토리면 디렉토리도 삭제
```

## 6. Exit Code 규약

| Code | 의미 |
|------|------|
| 0 | USER APPROVED (커밋 진행) |
| 1 | USER REJECTED (커밋 차단, AI에게 메시지 출력) |
| 2 | UNAVAILABLE STRATEGY (다음 전략으로 폴백) |

## 7. pre-commit hook 템플릿 예시

```bash
#!/bin/bash
# Review Commit Hook
# INSTALL_PATH=/path/to/utility/hook  # uninstall에서 참조

if [ -x "$INSTALL_PATH/pre-commit-hook" ]; then
    "$INSTALL_PATH/pre-commit-hook"
    exit $?
fi
exit 0
```
