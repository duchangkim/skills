---
name: cloudflare
description: "개인 Cloudflare 계정(`duchi.click` 도메인·DNS·Workers 등)을 Cloudflare 공식 `cf` CLI로 관리한다. Cloudflare 리소스 조회·생성·변경, DNS 레코드 작업, Workers 배포, `cf` 인증 프로필 점검 시 사용한다."
---

# Cloudflare

개인 도메인, DNS, 서버리스 인프라는 Cloudflare로 운영한다.
Cloudflare 관리는 `cf` CLI를 사용하며, 모든 `cf` 명령에 `--profile duchangkim`을 명시한다. `cf auth activate`로 디렉터리에 바인딩하지 않는다.

`cf`가 없으면 `npm install --global cf`로 설치한다(Node.js 22.18 이상, Bun 미지원).

작업 전 반드시 다음처럼 `duchangkim` 프로필의 인증 상태를 점검한다. `authenticated`와 `tokenValid`가 `true`여야 한다.

```bash
cf auth whoami --profile duchangkim
```

`duchangkim` Auth 프로필이 없거나(`cf auth list`로 확인) 인증이 만료된 경우, 작업을 중단하고 사용자에게 다음 인증 절차를 안내한다. 로그인은 사용자만 진행한다.

```bash
# Auth 프로필 생성 및 로그인
cf auth create duchangkim
```

## 명령 탐색과 실행

- 명령은 기억에 의존하지 말고 `cf cli search "<작업 설명>"`으로 찾고 `--help`로 확인한다. 검색어에는 도메인·ID 같은 식별 값을 넣지 않는다.
- 변경 명령은 먼저 `--dry-run`으로 요청을 확인한다. 삭제·`--force`는 사용자 확인 후 실행한다.
- 프로젝트에 Wrangler 설정 파일이 있으면 `dev`·`build`·`deploy`는 Wrangler를 유지한다.

## 홈랩 연동

Cloudflare Tunnel과 `*.duchi.click` 서비스 공개 절차는 `homelab-infrastructure` 스킬을 따른다.
