# duchangkim 개인 스킬

개인적으로 사용하는 Agent Skills를 모아 관리하는 저장소다. [Agent Skills 표준](https://agentskills.io) 구조를 따르므로 `npx skills`로 Claude Code, Codex 등 여러 에이전트에 설치할 수 있다.

## 설치

설치 가능한 스킬 목록을 확인한다.

```bash
npx skills add duchangkim/skills --list
```

Claude Code에 모든 스킬을 전역으로 설치한다.

```bash
npx skills add duchangkim/skills --global --agent claude-code --skill '*' --yes
```

특정 스킬만 설치하거나 다른 에이전트에 설치할 수도 있다.

```bash
npx skills add duchangkim/skills --global --agent claude-code --skill homelab-infrastructure --yes
npx skills add duchangkim/skills --global --agent codex --skill '*' --yes
```

설치 후 스킬이 보이지 않으면 에이전트를 다시 시작한다. 최신 버전으로 갱신할 때는 다음 명령을 사용한다.

```bash
npx skills update -g
```

## 스킬 목록

| 스킬                                                         | 설명                                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------- |
| [homelab-infrastructure](skills/homelab-infrastructure/SKILL.md) | 홈랩(Proxmox, k3s, ArgoCD, Cloudflare)의 배포·진단·백업·복구 |

## 필수 준비 사항

- Bitwarden CLI(`bw`)가 설치되어 있고 개인 계정으로 로그인한 뒤 `BW_SESSION` 환경변수가 설정되어 있어야 한다.

## 저장소 구조

```text
skills/
└── <skill-name>/
    ├── SKILL.md
    ├── references/          # 선택 사항
    └── scripts/             # 선택 사항
```

스킬은 `skills/<skill-name>/SKILL.md` 아래에 추가한다. 저장소 루트에는 `SKILL.md`를 두지 않는다. 이 구조가 `npx skills add`의 다중 스킬 탐색과 호환된다.

## 새 스킬 추가

1. 새 디렉터리 이름과 `SKILL.md`의 `name`을 동일한 kebab-case로 정한다.
2. frontmatter에 `name`과 `description`을 작성한다. 설명에는 스킬의 목적과 활성화할 상황을 함께 적는다.
3. 본문에는 에이전트가 따라야 할 작업 흐름과 중요한 경계를 한국어로 작성한다.
4. 위 스킬 목록 표에 추가한다.

세부 작성 규칙은 [AGENTS.md](AGENTS.md)를 따른다.
