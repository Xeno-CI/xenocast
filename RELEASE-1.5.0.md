# XenoCast 1.5.0 초안

태그와 릴리스 발행은 운영 서버 배포 뒤에 한다. 이 문서는 초안이다.

## 변경

- `xenocast feedback "메시지"`로 막힌 빌드를 계정에 남긴다. `--build`, `--with-log`, `--attach`, `--json`. CI 토큰은 거절된다.
- MCP 도구 `feedback`과 `webhooks`가 공개 목록에 있다. 웹훅은 list, add, test, remove.
- `--json` 출력은 오류도 JSON 한 개다.
- 설치 뒤 `setup --no-tui`가 이어진다. `XENOCAST_NO_SETUP=1` 또는 `--no-setup`이면 설치만 한다. (xenocast PR #4와 같은 내용)

## 설치

```
curl -fsSL https://github.com/Xeno-CI/xenocast/releases/latest/download/install.sh | sh
```

실행 파일 6종은 이 초안에 없다. 릴리스를 발행할 때 빌드한다.
