<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset=".github/assets/xenocast-dark.svg">
  <img src=".github/assets/xenocast-light.svg" alt="XenoCast" width="320">
</picture>

**지금 쓰는 컴퓨터에서, 내 Mac 빌드까지**

[![Release](https://img.shields.io/github/v/release/Xeno-CI/xenocast?style=flat-square&color=2563eb)](https://github.com/Xeno-CI/xenocast/releases) [![MIT License](https://img.shields.io/badge/license-MIT-475569?style=flat-square)](LICENSE) [![Docs](https://img.shields.io/badge/docs-xenoci.com-2563eb?style=flat-square)](https://xenoci.com/docs) [![Downloads](https://img.shields.io/github/downloads/Xeno-CI/xenocast/total?style=flat-square&color=475569)](https://github.com/Xeno-CI/xenocast/releases)

[한국어](README.md) | [English](README.en.md)

</div>

## 설치

macOS와 Linux

```sh
curl -fsSL https://xenoci.com/install.sh | sh
```

Windows PowerShell

```powershell
irm https://xenoci.com/install.ps1 | iex
```

설치 한 줄이 GitHub 로그인, MCP 등록, 빌린 Mac에서 연결 확인 빌드까지 이어지며 로그인 링크에서 직접 승인하면 됩니다

연결 확인 빌드에는 [대여 중인 Mac](https://xenoci.com/app/store)이 필요하며 대여와 결제는 직접 진행합니다

## AI에게 이 한 줄

Claude Code, Codex, Cursor, opencode에 그대로 붙여 넣으세요

> XenoCI 연결해줘: curl -fsSL https://xenoci.com/install.sh | sh 실행하고 나오는 로그인 링크를 나한테 보여줘

## 왜 XenoCast

**Mac 없이 Windows에서 iOS 빌드**
프로젝트를 빌린 Mac mini M4로 보내 빌드합니다
Xcode는 원격 Mac에서 실행하고, 나는 Windows나 Linux, macOS에서 작업합니다

**내 AI가 빌드부터 로그 확인까지**
MCP 도구 다섯 개로 빌드, 상태, 취소, 로그, Mac 목록을 다룹니다
대화하던 AI가 작업을 시작하고 로그를 읽고 필요하면 취소합니다

**공유 러너가 아닌 내 대여 Mac**
공유 CI 러너 대신 내 대여에 배정된 Mac에서 빌드합니다
`xenocast macs`로 Xcode 버전과 남은 대여 시간을 확인합니다

**기존 CI에 빌드 전용 토큰 하나**
GitHub Actions, GitLab, Jenkins에서 CI 토큰으로 연결합니다
빌드와 상태, 로그, 취소, Mac 조회만 허용하고 서명이나 주문, 연장 권한은 주지 않습니다

**서명과 TestFlight는 준비 중**
원격 빌드와 결과물 다운로드는 지금 사용할 수 있습니다
관리형 iOS 서명과 TestFlight 업로드는 이 셋업에서 아직 검증 전이므로 릴리스 배포 경로로 사용하지 않습니다

## 첫 빌드

프로젝트 폴더에서 저장소의 빌드 스크립트를 실행합니다

```sh
xenocast build --script ./ci.sh
```

현재 폴더 업로드부터 로그 출력까지 기다리고 원격 빌드의 종료 코드로 끝납니다
`--no-wait`는 빌드 ID만 출력하고, 기다리는 중 Ctrl+C를 누르면 원격 빌드도 취소합니다

## 자주 쓰는 명령

| 할 일 | 명령 |
| --- | --- |
| 로그인과 계정 확인 | `xenocast login` / `xenocast whoami` |
| AI 연결과 점검 | `xenocast setup --no-tui` / `xenocast doctor` |
| Mac과 남은 시간 | `xenocast macs` |
| 빌드 실행 | `xenocast build --script ./ci.sh` |
| 상태와 로그 | `xenocast status <rb_id>` / `xenocast logs <rb_id>` |
| 실패 원인과 취소 | `xenocast diagnose <rb_id>` / `xenocast cancel <rb_id>` |
| 결과물 받기 | `xenocast artifacts <rb_id> --out ./dist` |
| 로그인 폐기 | `xenocast logout` |

명령별 옵션은 `xenocast <명령> --help`, 전체 명령은 `xenocast --help`, 기계가 읽을 출력은 `--json`으로 확인합니다

<details>
<summary>결과물과 빌드 추적</summary>

```sh
xenocast build --script ./ci.sh --artifacts 'build/*.xcarchive'
xenocast artifacts <rb_id> --out ./dist
```

결과물은 7일 보관합니다
`--pr`, `--commit`을 생략하면 CI 환경 변수와 현재 Git 커밋에서 추적 정보를 가져옵니다
빌드와 watch는 원격 빌드의 종료 코드를 반환하며 사용법 오류는 2, 인증이나 서버, 파일 오류는 1입니다

</details>

<details>
<summary>GitHub Actions와 GitLab 등 CI 연결</summary>

로그인한 컴퓨터에서 토큰을 만들어 GitHub 시크릿으로 바로 보냅니다

```sh
xenocast token --ci --name "owner/name CI" --raw | gh secret set XENOCAST_TOKEN --repo owner/name
```

[GitHub Action](https://github.com/Xeno-CI/build)에 `token: ${{ secrets.XENOCAST_TOKEN }}`을 전달합니다
GitLab과 다른 CI에서는 비밀 변수 `XENOCAST_TOKEN`을 등록하고 CLI로 빌드합니다

```sh
xenocast build --script ./ci.sh
```

CI 토큰은 기본 90일 후 만료하며 `--expires 30d`로 바꾸거나 `xenocast token --ci --revoke <ses_id>`로 폐기합니다
포크 PR에는 토큰을 전달하지 않고 `pull_request_target`으로 외부 PR 코드를 실행하지 않습니다
설치와 CI별 시크릿 등록은 [SETUP.md](SETUP.md)를 따르세요

</details>

<details>
<summary>MCP 연결과 지원 AI</summary>

`xenocast setup`이 설치된 Claude Code, Codex, Cursor, opencode, OMO에 MCP 서버 `xenoci`를 등록합니다
서버 명령은 `xenocast mcp`이며 설정 파일에 토큰을 넣지 않습니다

| 도구 | 역할 |
| --- | --- |
| `build` | 빌드 실행 |
| `status` | 빌드 상태 |
| `cancel` | 빌드 취소 |
| `logs` | 빌드 로그 |
| `macs` | Mac 목록 |

등록 후 AI의 새 세션에서 연결을 확인하세요
[직접 MCP 설정](mcp/CONNECT.md)과 [Claude Code 연결](docs/claude-code.md)에서 자세한 예제를 확인할 수 있습니다

</details>

<details>
<summary>로그인과 보안</summary>

API 키 대신 GitHub 로그인으로 연결합니다
macOS 키체인, Windows 자격 증명 관리자, Linux secret-service를 사용하며 없으면 이 기기에 묶인 AES-256-GCM 암호화 파일에 저장합니다
약 1시간짜리 접근 토큰은 CLI와 MCP가 자동 갱신하고 `xenocast logout`은 서버 세션도 폐기합니다
설치 스크립트는 릴리스의 SHA256SUMS로 실행 파일을 확인합니다
CI 토큰은 자격 저장소에 쓰지 않으며 빌드 로그에 섞인 토큰 모양 문자열은 가려서 출력합니다

</details>

<details>
<summary>문제 해결과 환경 변수</summary>

| 증상 | 다음 행동 |
| --- | --- |
| 명령을 찾지 못함 | 설치 출력의 PATH 안내 확인 |
| 로그인이 없거나 만료됨 | `xenocast login` 후 `xenocast doctor` |
| 대여 중인 Mac이 없음 | [Mac 대여](https://xenoci.com/app/store) 후 `xenocast macs` |
| 빌드 실패 | `xenocast diagnose <rb_id>`와 `xenocast logs <rb_id>` |
| AI에 도구가 안 보임 | `xenocast setup --no-tui` 후 AI 새 세션 시작 |

| 환경 변수 | 용도 |
| --- | --- |
| `XENOCAST_TOKEN` | 저장된 로그인 대신 CI 토큰 사용 |
| `XENOCI_API_URL` | API 주소, 기본 `https://xenoci.com` |
| `XENOCAST_CREDENTIAL_STORE` | `keychain`, `wincred`, `secret-service`, `file` |
| `XENOCI_CONFIG_DIR` | 설정 저장 위치 |
| `XENOCI_NO_BROWSER=1` | 로그인 링크의 자동 브라우저 열기 생략 |

이전 명령 이름 `xenoci`도 계속 동작합니다
[오류 코드](https://xenoci.com/docs/errors)의 `next` 안내를 따라 다음 행동을 선택하세요

</details>

[문서](https://xenoci.com/docs) | [AI용 안내](https://xenoci.com/llms.txt) | [OpenAPI](https://xenoci.com/openapi.json) | [MIT 라이선스](LICENSE)
