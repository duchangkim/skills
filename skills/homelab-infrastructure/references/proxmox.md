# Proxmox·백업·복구

호스트 명령은 `ssh pve <명령>`으로 실행한다. 노드 이름은 `pve`다. VM 디스크는 NVMe `local-lvm`, 백업·ISO는 HDD `hdd-storage`(`/mnt/hdd-storage`)에 둔다. 현재 VM 구성과 할당은 `qm list`, `qm config <vmid>`, `pvesm status`로 확인하고, 저장소 문서와 다르면 실제 상태를 기준으로 보고한다.

## 변경 원칙

- VM 중지·재시작·디스크·메모리 변경과 호스트 재부팅, `apt dist-upgrade`는 홈랩 전체 중단이다. 사용자 확인 후 진행하고, 먼저 수동 백업을 만든다.
  `ssh pve vzdump <vmid> --storage hdd-storage --mode snapshot --compress zstd`
- k3s VM 메모리를 바꾸면 워크로드 `limits` 합계도 검토한다.
- 호스트 네트워크(`vmbr0`) 변경은 원격 접근을 잃을 수 있다. 복구 수단을 먼저 확인한다.

## 백업

| 계층            | 방법                                                  |
| --------------- | ----------------------------------------------------- |
| VM 이미지       | vzdump 예약 백업 → `hdd-storage` (`/etc/pve/jobs.cfg`) |
| 앱 데이터       | `duchangkim/home-lab`의 `setup/backup-k3s-data.sh` (VM 안에서 실행, `--dry-run` 먼저) |

앱 데이터 백업은 결과의 `checksums.sha256`과 `backup-summary.txt`로 검증한다. 백업 파일은 Git 저장소 밖에 둔다. 새 앱에 DB나 PVC가 생기면 백업 스크립트에 포함됐는지 확인한다.

## 복구

- VM 단위: vzdump 백업을 `qmrestore`나 Web UI로 복원한다. 기존 VM 덮어쓰기는 사용자 확인 후 진행한다.
- 클러스터 재구축: `duchangkim/home-lab`의 `setup/PROXMOX_MIGRATION.md`를 따른 뒤 [sealed-secrets.md](sealed-secrets.md)의 재구축 절차를 수행한다.
