---
name: homelab-infrastructure
description: "개인 홈랩(Proxmox VE 호스트, k3s VM, ArgoCD GitOps, Sealed Secrets, Cloudflare Tunnel, *.duchi.click 서비스)을 관리한다. 홈랩 서비스의 배포·설정 변경·시크릿 추가·장애 진단·백업·복구, Proxmox VM 작업, 도메인·터널 작업 시 사용한다."
---

# 홈랩 인프라 관리 가이드

작업 전에 인벤토리에서 대상, 접근 방법, 관리 정본을 확인한다.

| 대상              | 용도                          | 접근 방법                                  | 관리 위치                        |
| ----------------- | ----------------------------- | ------------------------------------------ | -------------------------------- |
| Proxmox VE 호스트 | 하이퍼바이저, vzdump 백업     | SSH 별칭 `pve` (`root@192.168.0.100`), Web UI `:8006` | 직접 관리                        |
| VM 100 `k3s-node` | 단일 노드 k3s 클러스터        | SSH 별칭 `homelab` (`192.168.0.43`)        | `duchangkim/home-lab` (GitOps)   |
| 로컬 k3d          | 매니페스트 사전 검증          | `kubectl --context k3d-homelab`            | `duchangkim/home-lab` (`local` overlay) |
| Cloudflare        | `duchi.click` DNS, Tunnel, Access | 대시보드                               | 직접 관리                        |

---

## 정본 저장소

클러스터 위의 모든 것은 `duchangkim/home-lab`이 정본이다. 로컬 클론을 최신화하고 `AGENTS.md`, `README.md`, 관련 `docs/`를 읽은 뒤 작업한다. 스펙, 앱 목록, 리소스 할당은 그곳에서 확인한다.

영구 변경은 저장소 커밋 → push → ArgoCD 동기화로만 한다. `selfHeal`이 켜져 있어 클러스터를 직접 고친 내용은 되돌려지므로, 직접 조작은 진단과 긴급 조치에만 쓰고 같은 내용을 저장소에 반영한다.

## 클러스터 접근

운영 kubeconfig는 로컬에 없다. 운영 명령은 `ssh homelab kubectl --context default ...`로 실행한다. 로컬 `kubectl`은 항상 `--context k3d-homelab`을 붙인다.

## 보안 규칙

- 두 저장소 모두 **공개 저장소**다. 평문 `secret.yaml`, 개인키, `.env`, 토큰을 커밋하지 않는다. 시크릿은 Sealed Secret으로만 커밋한다.
- 공인 IP를 기록하지 않는다. 외부 공개는 Cloudflare Tunnel로만 하고 포트포워딩을 열지 않는다.
- 단일 저전력 노드다. 새 워크로드에는 `requests`/`limits`를 지정한다.

## 비밀 관리 (Bitwarden)

비밀은 Bitwarden 개인 금고의 `homelab` 폴더(`663f2720-95c8-4f28-885d-b4ef00f154f3`)가 정본이다. `bw` CLI로 주어진 `id`의 필요한 필드만 조회한다. 비밀은 셸 변수·명령 인자로 복사하지 않고 표준 입력으로 직접 전달하며, 컨텍스트에 노출하지 않는다.

작업 전 `bw status`의 `userEmail`이 `duchang.dev@gmail.com`이고 `unlocked`인지 확인한다. 아니면 멈추고 사용자에게 `bw login` 또는 `export BW_SESSION="$(bw unlock --raw)"`를 안내한다. 마스터 비밀번호는 사용자만 입력한다.

| 항목                                                 | id                                     |
| ---------------------------------------------------- | -------------------------------------- |
| Sealed Secrets 컨트롤러 키 (`pub-cert.pem`과 짝)      | `cd805c6f-b0a5-4b4a-b139-b4ef00f37e4f` |

Bitwarden에 등록된 SSH 키는 임시 파일에 기록해 `ssh -F /dev/null -o IdentitiesOnly=yes -i <임시 키 파일> <사용자>@<사설 IP>`로 전달하고, 끝나면 임시 파일을 지운다.

---

## 작업별 하위 문서

| 작업                                         | 하위 문서                                                    |
| -------------------------------------------- | ------------------------------------------------------------ |
| 앱 추가·변경, ArgoCD 동기화, 장애 진단       | [references/kubernetes.md](references/kubernetes.md)         |
| 시크릿 추가·변경, 컨트롤러 키 백업·복원      | [references/sealed-secrets.md](references/sealed-secrets.md) |
| 서브도메인 공개, Tunnel·Access·인증서        | [references/cloudflare.md](references/cloudflare.md)         |
| VM·호스트 관리, 백업·복구                    | [references/proxmox.md](references/proxmox.md)               |
