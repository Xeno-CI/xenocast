# XenoCast

XenoCI에서 빌린 Mac mini M4로 빌드하는 CLI(`xenocast`)와 MCP 서버입니다. GitHub으로 로그인하고, API 키는 쓰지 않습니다. 옛 이름 `xenoci`도 같은 명령으로 동작합니다(설치 스크립트가 `xenoci` 링크도 만듭니다). GitHub Actions 액션은 [Xeno-CI/build](https://github.com/Xeno-CI/build)에 있습니다.

**AI로 약 3분 셋업:** 쓰는 AI 에이전트에 이 주소 하나만 보내세요: https://github.com/Xeno-CI/xenocast/blob/main/SETUP.md
에이전트가 보여 주는 로그인 링크를 열어 GitHub으로 승인하면 됩니다. [docs/quickstart-ai.md](docs/quickstart-ai.md), [Claude Code 연결](docs/claude-code.md)

## 설치와 로그인

Node 없이 쓰는 실행 파일 하나입니다. 설치 스크립트가 SHA256SUMS로 체크섬을 확인합니다.

```sh
curl -fsSL https://github.com/Xeno-CI/xenocast/releases/latest/download/install.sh | sh
xenocast login                       # 링크를 열고 GitHub으로 로그인, 화면의 코드를 확인하고 승인
xenocast setup --no-tui              # 설치된 AI 앱에 MCP 등록 + doctor + 빌린 Mac에서 연결 확인 빌드
xenocast build --script ./ci.sh      # 현재 폴더 업로드 → 빌드 → 로그 → 빌드 종료 코드로 끝남
```

Windows (PowerShell):

```powershell
irm https://github.com/Xeno-CI/xenocast/releases/latest/download/install.ps1 | iex
```

로그인은 macOS 키체인, Windows 자격 증명 관리자, Linux secret-service에 저장되고, 셋 다 없으면 이 기기에 묶인 AES-256-GCM 암호화 파일에 저장됩니다. 접근 토큰은 약 1시간이고 CLI와 MCP 서버가 알아서 갱신합니다. `xenocast whoami`로 계정과 세션 만료를, `xenocast logout`으로 서버 폐기와 삭제를 합니다. 예전 API 키는 종료되었고, 쓰던 키가 있으면 `xenocast login`을 한 번 하면 됩니다.

Node가 있으면 `npx -y -p github:xeno-ci/xenocast xenocast build --script ./ci.sh`도 같습니다. `--no-wait`는 접수만 하고 빌드 ID를 출력합니다. 작업을 중단(Ctrl+C, SIGTERM)하면 빌드도 취소됩니다. `--pr`, `--commit`을 주지 않으면 CI 변수와 `git rev-parse HEAD`로 PR 번호와 커밋을 자동으로 붙입니다.

## Mac과 남은 시간

`xenocast macs`가 빌린 Mac, Xcode, 남은 시간을 보여 주고, 6시간 이하면 연장 명령을 함께 보여 줍니다. `xenocast extend <rt_id>`는 연장 주문을 만들고 결제 링크를 브라우저로 엽니다(결제는 사람이). 결제 뒤 `xenocast wait <주문번호>`로 반영을 기다립니다. Mac이 없으면 https://xenoci.com/app/store 에서 빌립니다.

## CI (GitHub Actions, GitLab, Jenkins 등)

CI에서는 로그인 대신 CI 토큰을 씁니다. 로그인한 컴퓨터에서 한 번 만들어 CI 시크릿 `XENOCAST_TOKEN`에 넣습니다. 토큰은 한 번만 출력되고 어디에도 저장되지 않습니다.

```sh
xenocast token --ci --name "owner/name CI" --raw | gh secret set XENOCAST_TOKEN --repo owner/name
xenocast token --ci --revoke <ses_id>    # 폐기 (웹의 로그인된 기기 화면에서도 가능)
```

CI 토큰은 빌드 전용입니다: 빌드, 상태, 로그, 취소, Mac 목록만 되고 서명, 주문, 연장은 되지 않습니다. 기본 만료는 90일입니다(`--expires 30d`).

```sh
# GitLab, Jenkins, CircleCI, Buildkite, Bitrise, Azure Pipelines: XENOCAST_TOKEN 비밀 변수를 두고
curl -fsSL https://github.com/Xeno-CI/xenocast/releases/latest/download/install.sh | sh
xenocast build --script ./ci.sh
```

GitHub Actions는 [Xeno-CI/build](https://github.com/Xeno-CI/build) 액션에 `token: ${{ secrets.XENOCAST_TOKEN }}`을 넘깁니다. 포크 PR에 토큰이 새지 않게 하는 방법(`pull_request_target` 금지 예시 포함)은 [SETUP.md의 CI 연결](SETUP.md)을 보세요. CLI는 토큰 원문을 출력하지 않고, 빌드 로그에 섞여 나온 토큰 모양 문자열도 가려서 보여 줍니다.

## AI 에이전트 (MCP: Claude Code, Codex, Cursor, opencode, OMO)

`xenocast setup`이 설치된 AI 앱에 MCP 서버 `xenoci`(명령 `xenocast mcp`)를 등록합니다. 설정 파일에는 토큰을 넣지 않습니다. 직접 설정과 연결 확인: [mcp/CONNECT.md](mcp/CONNECT.md)

공개 MCP 도구: budget, build, status, cancel, logs, macs, feedback, webhooks.
예전 도구 이름은 숨겨진 호환 별칭으로 계속 답하고, 주문, 연장, 서명처럼 MCP에서 뺀 일은 그 CLI 명령을 알려 줍니다.

## 명령

`xenocast login`, `xenocast logout`, `xenocast whoami`, `xenocast setup`, `xenocast token --ci`, `xenocast doctor`, `xenocast mcp`, `xenocast build --script ./ci.sh`, `xenocast status`, `xenocast logs <rb_id>`, `xenocast artifacts <rb_id> --out ./dist`, `xenocast cancel <rb_id>`, `xenocast watch`, `xenocast diagnose <rb_id>`, `xenocast macs`, `xenocast extend <rt_id>`, `xenocast catalog`, `xenocast order`, `xenocast orders`, `xenocast wait <주문번호>`, `xenocast reset`, `xenocast xcode`, `xenocast job <op_id>`, `xenocast errors`, `xenocast webhooks`, `xenocast ios`. 모든 명령 `--json`, 전체는 `xenocast --help`

빌드 결과물(ipa, xcarchive, xcresult)은 `xenocast build --artifacts 'build/*.ipa'`로 남기고 `xenocast artifacts <id> --out ./dist`로 받습니다(7일 보관). 오류는 `{"error":{"code","message","retryable","next","request_id",...}}` 형식이고 `next`에 다음 행동이 들어 있습니다. iOS 서명, TestFlight 업로드(`xenocast ios archive|sign|ship`)는 검증 전입니다.

환경 변수: `XENOCAST_TOKEN`(CI 토큰, 있으면 저장된 로그인 대신 씀), `XENOCI_API_URL`(기본 https://xenoci.com), `XENOCAST_CREDENTIAL_STORE`(keychain, wincred, secret-service, file), `XENOCI_CONFIG_DIR`(저장 위치), `XENOCI_NO_BROWSER=1`(로그인 링크를 브라우저로 열지 않음)

AI용 안내: https://xenoci.com/llms.txt, OpenAPI: https://xenoci.com/openapi.json, 오류 코드: https://xenoci.com/docs/errors
