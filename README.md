# UART 센서 통신 시스템

UART 명령으로 온도·습도·거리를 조회하고, 센서 측정값을 PC와 Basys3 FND에 출력하도록 통합하였습니다.

- 수행 시기 2026.08
- 참여 형태 2인 팀
- 담당 범위 Top module 통합 설계 및 발표 자료 작성

[포트폴리오 상세 페이지](https://eugeneson3.github.io/projects/fpga-uart-sensors/)

## 프로젝트 목표

시계·스톱워치와 센서 모듈을 하나의 Top에 통합하고, PC 명령에 따라 센서 값을 읽어 반환하는 통신 경로를 구현하는 것을 목표로 하였습니다.

## 설계 및 구현

- UART RX, FIFO, ASCII 명령 해석, 센서 측정, 응답 생성 및 UART TX를 연결하였습니다.
- /get_temp, /get_humid, /get_dist 명령과 FND 표시 선택을 구성하였습니다.
- DHT11 수신 데이터의 40비트 구성 및 체크섬에 따른 valid 출력을 확인하였습니다.
- 센서 드라이버 설계는 팀원 김민석이 담당하였으며, 본인은 Top module 통합을 담당하였습니다.

## Troubleshooting

### DHT11 측정값의 비트 밀림

실물 동작에서 측정값이 두 배 단위로 변하는 현상이 발생하였습니다.

팀은 SYNC 상태의 펄스 판정 과정에서 비트 경계가 어긋나는 현상으로 분석하였습니다.

센서 모듈에 Edge detector를 추가하여 신호 전이와 tick count를 함께 판정하였습니다.

발표 자료에 수정 전후 측정값 영상과 안정된 출력 결과를 기록하였습니다.

### SR04 테스트벤치의 race condition

distance 상태 진입 시 시뮬레이션 상태 천이가 예상과 다르게 관찰되었습니다.

echo 하강 에지와 clk 상승 에지가 겹치는 파형을 확인하였습니다.

테스트벤치 자극이 negedge를 기다리도록 event 구문을 추가하였습니다.

수정 후 상태 천이가 클럭에 맞추어 진행되는 파형을 확인하였습니다.

## 검증 및 결과

- 온도·습도·거리 조회 및 FND 출력 동작을 발표 영상으로 기록하였습니다.
- DHT11 체크섬 일치·불일치 조건과 UART·FIFO 경계 동작을 시뮬레이션으로 확인하였습니다.

## 소스코드 구조

현재 통합본과 팀 제출 변형을 각각 루트 및 team-submission/에 구성하였습니다. 각 files.f에 필요한 공통 모듈을 포함하였습니다.

| 경로 | 내용 |
| --- | --- |
| [src/top_watch_stopwatch.v](src/top_watch_stopwatch.v) | Top 및 센서 조회 통합 로직 |
| [src/uart.v](src/uart.v) | UART 송수신 |
| [src/ascii_decoder.v](src/ascii_decoder.v) | 문자 입력 해석 |
| [src/dht11.v](src/dht11.v) | 온습도 센서 인터페이스 |
| [src/sr04_controller.v](src/sr04_controller.v) | 초음파 거리 측정 |
| [team-submission/](team-submission/) | 팀 제출 버전과 독립 파일 목록 |

## 실행 방법

1. Vivado에서 files.f의 소스와 constraints/의 보드 핀 정의를 확인합니다.
2. 최상위 모듈과 사용하는 포트에 맞추어 XDC를 활성화한 후 합성 및 보드 다운로드를 수행합니다.
3. 9600 baud 설정에서 센서 조회 명령과 FND 출력을 확인합니다.

## 동작 동영상

- [센서 통신 시스템 동작](https://www.youtube.com/watch?v=JveokxzkBsI)
- [통합 기능 시연](https://www.youtube.com/watch?v=Frnncq4ZkU8)
- [DHT11 수정 후 출력](https://www.youtube.com/watch?v=Zye28Xb_3lo)

## 참고 자료

- [발표 자료](https://docs.google.com/presentation/d/1fx_m2Z3iZAGhJlWcYz796hRYBHJudlQB50N9zZlAVSI/edit)

## 소스 이력

2026년 11월 18일 로컬 프로젝트에서 구성한 공개 스냅샷입니다. 원본 파일의 상대 경로와 SHA-256은 [SOURCE_MANIFEST.json](SOURCE_MANIFEST.json)에 기록하였습니다. 빌드 산출물과 로컬 개발 환경은 제외하였습니다. 파일별 저작권 및 라이선스 고지를 유지하였습니다.

합성 최상위 모듈은 `top_watch_stopwatch`로 지정합니다. 각 파일에 함께 정의된 `top_watch` 및 `top_stopwatch`는 별도 과제용 모듈입니다. 정적 검사 결과는 docs/에 수록하였습니다.
