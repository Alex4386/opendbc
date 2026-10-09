# ax1ev-port-lda-hack 브랜치 구성

이 브랜치는 `sunnypilot/opendbc` master 위에 여러 작업 브랜치를 합쳐 놓은 통합 브랜치다.
upstream으로 그대로 올릴 수 있는 상태가 아니므로, 어떤 브랜치가 어떤 방식으로 들어왔는지 여기에 기록한다.
새 브랜치를 merge / cherry-pick 하면 아래 표에 한 줄씩 추가할 것.

## 베이스

| 항목 | 내용 |
|---|---|
| 베이스 | `origin/master` (`sunnypilot/opendbc` master 미러, 주기적으로 merge) |
| 마지막 master merge | `6e9f9631` (2026-09-03) |

## 합쳐진 브랜치

| 브랜치 | 출처 | 합친 방식 | 커밋 | 내용 |
|---|---|---|---|---|
| `elantra-2025-port` | Jason Wen (`origin/elantra-2025-port`, `c8810ccf`) | squash (`14c49615`에 포함) | `14c49615` | Hyundai CAN "refreshed" 메시지 대응. `hyundai_can_refresh.dbc`, `HyundaiRefreshPlatformConfig`, `CAN_REFRESH_MSGS` 플래그, Elantra 2024-25 / Elantra Hybrid 2024-25 |
| `elantra-2025-port-sp` | Alex4386 (`origin/elantra-2025-port-sp`, `4b316aa4`) | squash (`14c49615`에 포함) | `14c49615` | 위 브랜치의 sunnypilot 버전 + Inster 초기 지원 |
| `ax1ev-port-sp` | Alex4386 (`origin/ax1ev-port-sp`, `89c7a3bf`) | squash (`14c49615`에 포함) | `14c49615` | Inster (AX1EV) 포트: `ALT_AX1EV_BUTTONS`, `AX1_CRUISE_BUTTONS` / `AX1_LDA_BUTTON`, 세이프티 |
| (LDA hack) | Alex4386 | squash | `14c49615` | Inster LDA 버튼 → MADS, `_hyundai_can_common.dbc` 분리 리팩터 |
| `hyundai-radar-tracks` | royjr (`origin/hyundai-radar-tracks`, `82683e54`) | merge | `fcb6405e` | fingerprint 기반 Hyundai 레이더 트랙 자동 감지 (`500_51F`, `210_21F`, `235_248`, `3A5_3C4`, `602_617`). Inster는 bus 1의 Mando MRR35 (`0x3A5`-`0x3C4`) 사용 |

`14c49615`는 `elantra-2025-port` → `elantra-2025-port-sp` → `ax1ev-port-sp` 계열 작업을 하나로 squash한 커밋이라
git ancestry로는 원래 브랜치와 연결되지 않는다. 출처 확인은 파일 내용 비교로 했다.

## 이 브랜치에서 직접 한 작업

| 커밋 | 내용 |
|---|---|
| `fca0d767` | `LFAHDA_MFC` 송신 시 순정 HDA 상태 전달 (HDA 표시 사라지는 문제), `Navi_HU.Navi_RoadClass`, `AX1_CRUISE_BUTTONS` 값 테이블 |
| `fcb6405e` | radar-tracks merge 후 수정: deprecated `RadarPoint` 필드 제거, 디버그 `print()` 제거, `HyundaiRadar` 파라미터 미설정 시 기본값, 레이더 결과 20 Hz 발행 |

## 알려진 문제 / TODO

- `hyundai-radar-tracks`가 모든 차종에서 `MANDO_RADAR` 플래그를 제거했지만, `sunnypilot/car/interfaces.py`의
  `_initialize_radar_tracks()`는 여전히 그 플래그를 조건으로 레이더 트랙 활성화(UDS `0x7d0`)를 한다. 현재 어떤 차종에서도 실행되지 않음.
  트랙 활성화가 필요한 Mando 레이더 차종(Sonata, Santa Fe, Telluride 등)은 트랙이 안 나올 수 있다.
- MRR35 `REL_ACCEL`은 Inster 데이터에서 `d(REL_SPEED)/dt`와 맞지 않는다 (상관 0.57, 기울기 0.19). 현재 사용하지 않음.

## 참고만 한 브랜치 (합치지 않음)

| 브랜치 | 출처 | 이유 |
|---|---|---|
| `ccnc-lane-change-smonth` | winstar0070/opendbc | ccNC 계기판(CAN-FD `CCNC_0x161` / `CCNC_0x162`) 전용. Inster는 클래식 CAN이라 해당 메시지가 없음. Inster에서는 `HDA11_MFC` (`0x484`)가 비슷한 역할로 보여 별도 분석 중 |
