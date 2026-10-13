# XenoCast

XenoCI macOS 러너에 잡을 보내는 CLI(`xenocast`)와 MCP 서버입니다. 인증은 XenoCI 콘솔에서 발급한 API 키(`xci_live_…`) 하나뿐입니다. 옛 이름 `xenoci`도 같은 명령으로 동작합니다(설치 스크립트가 `xenoci` 링크도 만듭니다)

**AI로 셋업:** 쓰는 AI 에이전트에 이 주소 하나만 보내세요: https://github.com/Xeno-CI/xenocast/blob/main/SETUP.md
사람이 하는 일은 콘솔(https://xenoci.com/console)에서 크레딧 충전과 API 키 발급, 키를 비밀 저장소에 넣는 것입니다. AI가 비밀 저장소에서 키를 받아 `xenocast setup`을 실행하고 MCP를 등록합니다. [docs/quickstart-ai.md](docs/quickstart-ai.md), [Claude Code 연결](docs/claude-code.md)

## 설치와 키

Node 없이 쓰는 실행 파일 하나입니다. 설치 스크립트가 SHA256SUMS로 체크섬을 확인합니다

```sh
curl -fsSL https://github.com/Xeno-CI/xenocast/releases/latest/download/install.sh | XENOCAST_NO_SETUP=1 sh
export XENOCI_API_KEY="$(security find-generic-password -a "$USER" -s XENOCI_API_KEY -w)"   # 비밀 저장소에서 키 꺼내기(macOS 키체인 예)
xenocast setup --no-tui --key-env XENOCI_API_KEY   # 키 확인, 이 컴퓨터에 키 저장, 설치된 AI 앱에 MCP 등록, MCP 서버 확인
unset XENOCI_API_KEY
xenocast build --script ./ci.sh      # 현재 폴더 업로드 → 잡 실행 → 로그 → 스크립트 종료 코드로 끝남
```

Windows (PowerShell):

```powershell
irm https://github.com/Xeno-CI/xenocast/releases/latest/download/install.ps1 | iex
```

`setup`은 키를 macOS 키체인, Windows 자격 증명 관리자, Linux secret-service에 저장하고, 셋 다 없으면 이 기기에 묶인 AES-256-GCM 암호화 파일에 저장합니다. 환경 변수 `XENOCI_API_KEY`가 있으면 저장된 키보다 우선합니다. 키가 없으면 명령은 서버에 요청하지 않고 `api_key_required`로 끝납니다. 키는 명령 인자로 받지 않습니다(`--key-env 변수 이름` 또는 표준 입력 `xenocast config set api-key`)

## AI 에이전트 (MCP: Claude Code, Codex, Cursor, opencode, OMO)

`xenocast setup`이 설치된 AI 앱에 MCP 서버 `xenoci`(명령 `xenocast mcp`)를 등록합니다. 설정 파일에는 키를 넣지 않습니다. 직접 설정: [mcp/CONNECT.md](mcp/CONNECT.md)

공개 MCP 도구: build, status, cancel, logs, list_runners, whoami, credits, usage.
키가 없어도 도구 목록은 같고, 호출하면 `api_key_required`로 답합니다

## 명령

같은 소스와 같은 설정(스크립트, Xcode, 러너, env, 결과물 경로)으로 24시간 안에 성공한 잡이 있으면 `xenocast build`는 업로드와 실행 없이 그 잡 ID를 알려 주고 종료 코드 0으로 끝납니다, 다시 실행하려면 `--force`를 붙입니다. `--lane 이름 --supersede`는 같은 워크스페이스에서 같은 작업 이름의 앞선 대기, 실행 잡을 새 잡 접수 뒤 취소합니다(대기 중이던 잡은 0원, 실행 중이던 잡은 시작한 분까지만 과금). MCP `build`도 같은 `force`, `lane`, `supersede` 인자를 받습니다

`xenocast setup`, `xenocast config`, `xenocast whoami`, `xenocast credits`, `xenocast usage`, `xenocast runners`, `xenocast build --script ./ci.sh`, `xenocast status <job_id>`, `xenocast jobs`, `xenocast logs <job_id>`, `xenocast cancel <job_id>`, `xenocast wait <job_id>`, `xenocast artifacts <job_id> --out ./dist`, `xenocast mcp`. 모든 명령 `--json`, 전체는 `xenocast --help`

충전은 사람이 콘솔에서 합니다. `xenocast credits topup`은 충전 링크만 보여 줍니다. 오류는 `{"error":{"code","message","retryable","next",...}}` 형식이고 `next`에 다음 행동이 들어 있습니다

환경 변수: `XENOCI_API_KEY`(API 키, 저장된 키보다 우선), `XENOCI_API_URL`(기본 https://xenoci.com), `XENOCAST_CREDENTIAL_STORE`(keychain, wincred, secret-service, file), `XENOCI_CONFIG_DIR`(저장 위치)

## 부가 기능: 다른 CI에서 빌드

CI 비밀 변수 `XENOCI_API_KEY`에 콘솔에서 발급한 키를 넣고 설치 후 `xenocast build --script ./ci.sh`를 실행합니다. GitHub Actions는 [Xeno-CI/build](https://github.com/Xeno-CI/build) 액션에 같은 키를 `api-key`로 넘깁니다. CLI는 키 원문을 출력하지 않고, 로그에 섞여 나온 키 모양 문자열도 가립니다

문서: https://xenoci.com/docs/agents/cli, MCP: https://xenoci.com/docs/agents/mcp
