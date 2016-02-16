저자 : 게시물 linuxer : 2014 년 9 월 4 일 오전 19시 59 분 카테고리 : Linux 커널 분석

I. 소개

GIC (일반 인터럽트 컨트롤러) 일반 인터럽트 컨트롤러 ARM 사이며, 그 아키텍처 사양은 현재 4 가지 버전이 있으며, V1~V4 (V2는 8 ARM 코어를 지원하고 V3 / V4는 메인 더 ARM 코어를 지원하고 있습니다 ARM64 서버 시스템 아키텍처를위한). 현재 공식 사이트에서는 ARM 버전의 GIC 아키텍처 사양 2에 다운로드 할 수 있습니다.

PL390, GIC-400, GIC-500 : 모델이있는 것을 포함 GIC의 ARM IP, IP를 구입할 것이다 때 자신의 ARM vensor SOC를 개발하려면. GIC-500은 128 CPU 코어를 지원하는 그것은 GIC 아키텍처 사양 버전 3에 따라 ARMV8 명령어 세트 (등의 Cortex-A57와 같이) 일 필요가 ARM 코어가 필요합니다. 본고에서는 버전 GIC 아키텍처 사양 2에 따라 임베디드 시스템에 적합, GIC-400에 대해 설명합니다. GIC-400 버스는 AMBA (고급 마이크로 컨트롤러 버스 아키텍처)의 하나 이상의 ARM 프로세서 등의 시트에 연결되어있다.

본 논문에서는 GIC-400 Linux 커널의 인터럽트 컨트롤러 드라이버 코드를 분석. 구체적인 분석은 단락 분석에 의한 인덱스 단락의 소스 코드에 따른다. 코드 분석의 모든 작품을 분명히 세부 사항을 달성하기 위해 노력하고 있습니다. 그것은 GIC의 하드웨어 기술을 많이 소개하는 피할 수없는 이외에 특정 GIC-400 인터럽트 컨트롤러 드라이버 코드와 Linux 커널의 인터럽트 서브 시스템은 상호 작용을 설명하지만,이 기사에서는 일반적 적인 인터럽트 서브 시스템의 Linux 커널을 설명하지 않습니다.

참고 : Linux 커널의 특정 버전은 Linux-3.17-RC3입니다.

둘, GIC-400 하드웨어 설명

1, GIC-400 입출력 신호

입력 신호와 출력 신호의 모식도 (1) GIC-400

빌딩 블록 (소프트웨어 또는 하드웨어 여부)을 이해하기 위해서는 블랙 박스로 그것을 넣을 수 있지만, 입력, 출력을 연구한다. (: 우리는 시계, config 및 다른 신호를 선택적주의) 아래와 같이 회로도 GIC-400의 입력 신호와 출력 신호는 다음과 같습니다.

gic-400

(2) 입력 신호

왼쪽 위 인터럽트 소스 이미지는 주변에서의 입력 신호이다. 두 가지, 즉 PPI (전용 주변 장치 인터럽트)와 SPI (공유 주변기기 인터럽트)로 분할. 사실, 이름은 PPI의 CPU 인터럽트 신호가 개인적으로 인터럽트 신호 특성의 두 가지에서 알 수 있듯이, 각 CPU는 자신의 특정 PPI 신호선이있다. SPI는 모든 CPU간에 공유되고있다. GICD_TYPER SPI는 (480까지) 레지스터 번호를 사용하여 설정할 수 있습니다. GIC-400은 SPI 인터럽트 번호 입력 신호선을지지하는 많은 SPI 인터럽트 요청 신호가있다. 마찬가지로, 레지스터 GICD_TYPER에서도 (8까지) CPU 인터페이스의 수를 설정할 수 있으며, GIC-400은 PPI는 신호선을 인터럽트 입력 신호선 군을 많이 제공하고 CPU 인터페이스의 수를 지원하고 있습니다 . PPI 인터럽트 신호선 그룹은 6 개의 실제 신호를 포함한다.

(A) nLEGACYIRQ 신호선. 바이 패스 모드 (우회 우회 GIC 기능을 의미한다)에서 인터럽트 ID 31이 대응하는 nLEGACYIRQ는 CPU nIRQCPU 대응하는 신호선에 직접 연결할 수있다. 이러한 설정은 CPU가 CPU의 PPI와 SPI 인터럽트 응답의 다른 부분에 참여했지만, 특히이 1 인터럽트 라인 서비스 용되지 않습니다.

(B) nCNTPNSIRQ 신호선. 비보안 물리적 타이머 해당 인터럽트 ID 30에서 이벤트를 중단합니다. 로컬 타이머는 CPU에 인터럽트를하기 위해이 이벤트가있다. 거기에 지역의 타이머가 글로벌 타이머를 기반으로하며 멀티 CPU 아키텍처 일반 타이머 (deboucing 타이머 예를 들어 키패드 또는 배터리 구동 및 모든 상황 500ms 폴링 전기)에서 일반적으로 글로벌 타이머 했다이어야하며 일부 소프트웨어의 CPU 관련 행동 (CPU의 실행 시간을 표시하는) 로컬 타이머 구동이다.

(C) nCNTPSIRQ 신호선. 안전한 물리적 타이머 인터럽트 이벤트 해당 인터럽트 ID 29에서. 컨셉 전술 설명하지 않는다.

(D) nLEGACYFIQ 신호선. 인터럽트 ID 28이 대응한다. nLEGACYIRQ 신호선과 개념은 설명하지 않는다.

(E) nCNTVIRQ 신호선. 인터럽트 ID 27이 대응한다. 타이머 이벤트 가상 및 가상화 관련, 그리고 설명이 없습니다.

(F) nCNTHPIRQ 신호선. 인터럽트 ID 26이 대응한다. 하이퍼 바이저 타이머 이벤트 및 가상화 관련에서는 그 설명은 없습니다.

(3) 출력 신호

이른바 출력 신호는 실제로는 각 CPU GIC와 직접 인터페이스가 포함되어야한다.

(A) 트리거 CPU의 인터럽트 신호. nIRQCPU과 nFIQCPU 신호선과 두 신호선에 정통한 CPU 엔지니어가 생소이어서는 안 ARM은 주로 IRQ 모드, FIQ 모드에 ARM의 CPU를 실행하는 데 사용된다. 가상화와 관련된 기재가 없습니다.

(B) 신호 일어난다. nFIQOUT과 nIRQOUT 신호 라인은 CPU를 깨워하기 위해 사용되는 ARM CPU 전원 관리 모듈로 이동

(C) AXI 슬레이브 인터페이스 신호를. AXI (고급 확장 Interface) 버스 프로토콜은 AMBA 사양에 속하는 일부입니다. 이 신호선은 ARM CPU와 GIC-400 (예를 들어, 레지스터 액세스) 통신 할 수있다.

할당 된 인터럽트 번호 (4)

GIC-400은 인터럽트 유형은 다음과 같은 범주가 있습니다 지원합니다.

(A) 주변 인터럽트 (주변 인터럽트). 그들은 인터럽트 요청 신호의 실제 물리적 인터럽트가 위에서 소개되고있다.

(B) 트리거 (SGI 소프트웨어 생성 인터럽트) 인터럽트 소프트웨어. 소프트웨어 인터럽트 이벤트를 트리거하는 레지스터 GICD_SGIR을 쓸 수 그러한 중단은 프로세서 간의 통신을 위해 사용할 수있다.

(C) 가상 인터럽트 (가상 인터럽트)와 유지 보수가 인터럽트. 중단과이 기사는 둘 다 아무 상관이없는 그들을 반복하지 않는다.

이러한 인터럽트 요인을 파악하기 위해, 우리는 그들의 다음과 같이 할당 된 고유 한 ID를 인코딩해야합니다.

(A) ID0~ID31 인터럽트의 특정 프로세스에 분배하는 데 사용된다. 각 원본이 동일한 ID0~ID31로 식별되는 인터럽트 이후, 다만 ID에 의존 할 수 없습니다 이러한 인터럽트를 파악하고 따라서 이러한 인터럽트 ID + CPU 인터페이스 번호를 중단 식별해야합니다. SGI위한 ID0~ID15, PPI위한 ID16~ID31. 중단 PPI 형은 자체 프로세스로 전송되고 다른 프로세스에 의존하지 않는 것이다. SGI가 트리거 GICD_SGIR 레지스터를 작성하여 중단되었다. 프로세서 원래 ID에 의해 분배는 고유 SGI를 식별하기 위해 ID와 타겟 프로세서 ID를 중단.

(B) SPI 용 ID32~ID1019. 이것은 GIC 사양은 실제로 GIC-400은 480 SPI를 지원하는 최대 크기입니다.

내부 로직 2, GIC-400

(1) GIC의 블록 다이어그램

GIC의 블록 다이어그램은 다음과 같습니다.

gic

GIC는 분명히 두 블록으로 나눌 수 있으며, 블록은 CPU 인터페이스이다 (그림의 블록 왼쪽) 분배이다. CPU 인터페이스는 두 가지가 하나이며, 범용 프로세서 인터페이스 및 기타 가상 머신 인터페이스이다. 가상 CPU 인터페이스는 본 명세서에서는 상세하게 설명하지 않는다.

(2) 분배의 개요

분배의 주요 역할은 소스를 인터럽트 각 인터럽트 요인의 상태를 감지 각 동작을 제어하는 것으로, 각 분포에 의해 생성 된 이벤트는 소스가 하나 이상의 지정된 CPU 인터페이스에 전달되는 인터럽트 인터럽트. 분배는 여러 개의 인터럽트 소스를 관리 할 수 있지만, 그것은이지만 항상 가장 우선 순위가 높은 인터럽트 요구는 CPU 인터페이스로 전송됩니다. : 포함 인터럽트 제어를위한 분배,

(1) 컨트롤을 활성화하거나 비활성화 할 인터럽트. 분배 인터럽트 제어는 두 단계로 나뉜다. 하나는 글로벌 인터럽트 제어 (GIC_DIST_CTRL)입니다. 일단 글로벌 인터럽트를 비활성화 생성 된 모든 인터럽트 소스 인터럽트 이벤트는 CPU 인터페이스에 전달되는 것은 아닙니다. 다른 레벨에서는 소스 인터럽트 인터럽트 이벤트는 CPU 인터페이스에 배포되지 않습니다 원인이되는 1 인터럽트 소스를 해제 할 때마다 (GIC_DIST_ENABLE_CLEAR)을 제어 할 수 있지만, 분산 형 발전의 다른 인터럽트 소스 인터럽트 이벤트 에 영향을주지 않습니다.

(2) 1 또는 CPU 인터페이스 그룹에 배포하고 현재 가장 높은 우선 순위의 인터럽트 이벤트를 제어합니다. 여러 CPU 인터페이스 인터럽트 이벤트 분포가시 GIC의 내부 로직이 확보되어야한다 때에는 그 CPU만을 주장하고있다.

(3) 우선 제어.

(4) 속성 설정을 중단. 예를 들어, 레벨 민감한 또는 에지 트리거

(5) 인터럽트 그룹 설정

분배는 어떤 인터럽트 소스를 관리 할 수 인터럽트 소스를 ID로 식별됩니다 우리는 인터럽트 ID를 호출합니다.

(3) CPU 인터페이스

CPU 인터페이스와이 블록은 주로 인터페이스하는 데 사용되는 프로세스. 이 블록의 주요 기능은 다음과 같습니다.

(A)의 이벤트를 인터럽트 CPU의 주장을 연결하기위한 CPU 인터페이스를 활성화 또는 비활성화합니다. 인터럽트 신호선 ARM의 CPU 인터페이스 블록과 CPU의 nIRQCPU과 nFIQCPU입니다. 당신은 인터럽트를 비활성화했을 경우, 그 후에도 분배는 CPU 인터페이스 인터럽트 이벤트를 배포하지만, 그것은 예고 지정된 프로세서를 주장 하나 nFIQ nIRQ 것은 아닙니다.

(B) ackowledging 인터럽트. 프로세서는 유통이 활성화 변경 보류 상태에서 인터럽트 상태를 둔 한 번 답했다 중단하고 인터럽트 CPU 인터페이스 블록에 대답합니다. 당신이 인터럽트를 보류에 따르지 않는 경우는 CPU 인터페이스 nIRQCPU과 nFIQCPU 신호 라인을 해제한다. 과정에서 새로운 인터럽트를 생산하는 경우는 유통 보류 상태에서 보류 및 활성화 인터럽트 상태를 변경합니다. 이 때, CPU 인터페이스는 여전히 즉 다음 프로세서 신호 인터럽트에의 nIRQ 또는 주장 상태 nFIQ 신호를 유지합니다.

(C) 완료 인터럽트 통지 과정. 인터럽트 핸들러는 인터럽트 처리 시간이 종료하면 인터럽트 처리를 완료 한 GIC의 CPU에 통지하기 위해서 등록하는 CPU 인터페이스를 설명합니다. 한편으로는이 작업을 수행하려면 대리점에 통지하는 것이다 비활성 있도록 변경 상태가 중단 된 반면에, 다른 하나는 CPU 인터페이스의 제출 보류중인 인터럽트를 가능하게 할 수있다.

(D)는 우선 순위 마스크를 설정합니다. 우선 순위 마스크함으로써 좀 낮은 우선 순위의 인터럽트를 마스크 할 수 인터럽트가 CPU에이를 통지하지 않습니다.

(E) 설정 선점 전략
이벤트 여러 인터럽트 (F)는 동시에 온다, 가장 높은 우선 순위 통지 프로세서 중 하나를 선택합니다.

(4) 실시 예

다음과 같이 우리는 상호 작용 GIC와 CPU 인터페이스를 설명하기 위해 실용적인 예제를 사용하여 특정 프로세스는 다음과 같습니다.

xxx

(참고 : 사진이 너무 길어서 때 조금 심한보기 위하여 마지막에 넣어 그 위에 목시 활성)

먼저 전제 조건 :

(A) N과 M은 M보다 N 우선 순위가 높은 두 주변 인터럽트를 식별하는 데 사용되는

(B) 2 개의 인터럽트는 SPI의 종류, 레벨 트리거, 액티브 하이이다

(C) 2 개의 인터럽트는 CPU와 함께 가도록 설정되어있는

(D)는 FIQ 인터럽트에 의해 트리거 그룹 0으로 설정되어있는

상호 작용을 설명하는 타임 라인에 따라 다음의 표 :

1

시간 상호 작용 행동을 설명하십시오
T0 시간 분배는 인터럽트 요인의 유효한 트리거 레벨 M을 감지하고
T2 시간 소스가 중단됩니다. 이 상태는 보류로 설정되어 분배 M
T17 시간 약 15 시계, nFIQCPU 신호선 다운 CPU 인터페이스, 후 CPU의 인터럽트 요청 보고서 M 주변기기에. 이 때, 승인 레지스터 (GICC_IAR)의 CPU 인터페이스의 내용은 M 인터럽트 소스에 대응하는 ID 변경 될
T42 시간 분배는 N 트리거 이벤트의 우선 순위가 높은 인터럽트 요인을 발견 한
T43 시간 분배 N이 상태에서는 소스는 보류로 설정되어있는 중단됩니다. 더 높은 우선 순위 N에 동시에 배포자가 현재 가장 우선 순위가 높은 인터럽트를 맞이 때문에,
T58 시간 약 15 클럭 후 CPU 보고서 N 주변 인터럽트 요청에 nFIQCPU 신호선 다운 CPU 인터페이스. 물론, 시간과 T17은 CPU를 주장하기 때문에 신호의 실제 수준이 주장 된 채로하고있다. 이 때, 승인 레지스터 (GICC_IAR)의 CPU 인터페이스의 내용이 ID의 N 인터럽트 소스에 업데이트되는
T61 시간 소프트웨어가 레지스터 ACK의 내용을 읽고 현재 가장 높은 우선 순위를 취득하고 상태가 인터럽트 ID (ID를 지원하는 즉, N 개의 인터럽트 소스)에 계속중인이 레지스터를 읽을 수 따라 CPU는 인터럽트 소스 N. 승인합니다 이때 분배 N이 상태는 소스가 보류로 설정되어 활성 (트리거 레벨 한 외부 레벨 신호를 주장이 남아있는 것이다 있도록 반드시 인터럽트가 CPU에 의해 처리되는 동안 보류 된 중단로 따라서 상태가 보류되고 활성) 참고 : T61는 CPU가 인터럽트 서비스를 시작 식별
T64 시간 3 클럭 후 CPU는 CPU 인터페이스 모듈 GIC는 CPU 인터럽트 요청에 머리를 들고 nFIQCPU 신호선을 해제한다 때문에 ACK가 인터럽트가 있기 때문에
T126 모멘트 N 주변 제어 레지스터의 인터럽트 서비스 루틴 동작은 (주변 인터럽트를 ACK) 때문에, N 주변 장치는 인터럽트 요청 신호를 해제한다
T128 모멘트 분배 리프트 N 주변기기 보류 상태 상태 N 개의 인터럽트 소스가 활성으로 설정되어 있기 때문에,
T131 모멘트 인터럽트 레지스터의 소프트웨어 작업 종료 (N GICC_EOIR이 등록하는 인터럽트 ID를 지원하는 쓰기) 인터럽트 처리가 종료를 식별합니다. 분배 N이 상태가 유휴 인터럽트 요인 주를 변경합니다 : T61~T131는이 기간 동안 CPU 인터럽트 서비스 시간 N 주변 지역에서 높은 우선 순위가있는 경우에는 보류중인 인터럽트 선점은 ( 하드웨어 센스를) 발생 끼어 듭니다 이번에는 CPU 인터페이스는 CPU가 새로운 주장 중단합니다.
T146 모멘트, 즉 M 약 15 클럭 후 분배 CPU 인터페이스는 현재 보류 된 가장 우선 순위가 높은 인터럽트 요인에보고한다. 긴 보류 후 M은 마지막으로 봄을 맞이했다. CPU의 인터럽트 요청 보고서 M 주변기기에 nFIQCPU 신호선 다운 CPU 인터페이스. 이 때, 승인 레지스터 (GICC_IAR)의 CPU 인터페이스의 내용은 M 인터럽트 소스에 대응하는 ID 변경 될
T211 모멘트 (GICC_IAR 레지스터를 읽어내는 것으로) M 인터럽트 ACK CPU는 낮은 우선 순위의 인터럽트를 시작했습니다.

셋째, 초기화 과정 GIC-400 IRQ 칩 드라이버

의 linux-3.17-RC3의 \ drivers \ 다른 인터럽트 컨트롤러 드라이버 코드 다양한 저장 irqchip 디렉토리에 커널이 버전은 GICV3을 지원하고 있습니다. IRQ-GIC-는 common.c GIC는 GIC의 다양한 버전에서 사용할 수있는 범용 드라이버 코드입니다. IRQ-gic.c은 V2 버전의 GIC의 컨트롤러이며, IRQ-GIC-v3.c는 V3 버전의 GIC 컨트롤러입니다.

1, GIC-400 장치 노드와 GIC-400 IRQ 칩 드라이버 매칭 처리

(1) IRQ 칩 드라이버 선언

irqchip.h 파일 linux-3.17-RC3의 \ drivers에 \ 다음과 같이 정의되어 irqchip 디렉토리 IRQCHIP_DECLARE 매크로 :

의 #define IRQCHIP_DECLARE (이름 compat, FN) OF_DECLARE_2 (irqchip 이름 compat, FN)

의 #define OF_DECLARE_2 (테이블 이름 compat, FN) \ _OF_DECLARE (테이블 이름 compat, FN, of_init_fn_2)

의 #define _OF_DECLARE (테이블 이름 compat, FN, fn_type) \

정적 상수 구조체의 of_device_id __of_table _ ## 명 \

__used __section (__ ## 테이블 ## _ of_table) \

= {.compatible = compat의 \

? 의 .data = (FN == (fn_type) NULL) FN : FN}

이 매크로는 실제로는 정적 상수의 구조 of_device_id를 초기화 __ irqchip_of_table 섹션에 배치되어있다. IRQ-gic.c 다음과 같이 정적 구조 of_device_id 상수를 정의하는 파일 IRQCHIP_DECLARE 사용 :

IRQCHIP_DECLARE (gic_400 "팔, GIC-400"gic_of_init);

Linux 커널 컴파일시 (섹션 이름이 __irqchip_of_table있는) 시스템은 특별 섹션에 IRQCHIP_DECLARE 매크로 정의에 모든 데이터를 컴파일하고 커널에 여러 IRQ 칩을 구성 할 수 있습니다, 우리는이 특별한 섹션을 호출 칩 테이블 IRQ로 불린다. 이 표에는 커널 인터럽트 컨트롤러는 모든 ID 정보 (가장 중요한 드라이버 코드의 초기화 함수와 DT 호환 문자열)을 지원하고 유지하고있다. 구조 of_device_id의 정의를 살펴 보자 :

of_device_id 구조

{

CHAR 이름 [32]; 이름과 일치하도록 ------ 장치 노드

일치하는 유형 ------- 장치 노드와 char 형 [32]

CHAR 호환 [128]; --- 일치하는 문자열 (DT 호환 문자열) 적절한 장치 노드를 일치

상수 void * 데이터는 -------- GIC 내용은 여기를 초기화 함수 포인터입니다

};

이 데이터 구조는 주로 정합 장치 노드 및 드라이버 모듈을 위해 사용된다. 데이터 구조의 정의에서 알 수 있듯이, 매칭 처리 장치 이름, 장치 유형 및 DT 호환 문자열 요인이 고려된다. 상세한 내용은 __ of_device_is_compatible 함수를 참조하십시오.

(2) 장치 노드

장치 노드는 메모리 자원, IRQ 설명 및 특정 초기화시 배터리에 전달 될 필요가있는 다른 정보를 포함하여 다양한 속성을 정의하고, 따라서 공정 노드 및 드라이버 모듈과 일치하는 장치를 필요로한다. 장치 트리 모듈 시스템에서는 다음과 같이 정의되어있다 (예를 들면, 우리의 같은 Rockchip RK3288 프로세서) GIC-400 및 전형적인 장치 노드 GIC-400 노드의 장치 노드 모두 포함 송어 :

GIC : 인터럽트 컨트롤러 @ ffc01000 {

호환 = "팔, GIC-400";

인터럽트 컨트롤러를.

# 인터럽트 세포 = <3>.

# 주소 세포 = <0>;

REG = <0xffc01000의 0x1000 = ""> ---- 분배 주소 범위

<0xffc02000의 0x1000 = ""> ----- CPU 인터페이스 주소 범위

<0xffc04000 0x2000에 = ""> ----- 가상 인터페이스 제어 블록

<0xffc06000 0x2000에 = "">; ----- 가상 CPU 인터페이스

인터럽트 =

.

};

(3)과 일치하는 장치 노드와 IRQ 칩 드라이버

컴퓨터 드라이버 초기화 함수는 irqchip_init IRQ 칩 드라이버의 초기화를 호출합니다. 드라이버 / irqchip / irqchip.c 파일에 정의 된 Irqchip_init 함수 다음과 같습니다 :

보이드 __init irqchip_init (공극)

{

of_irq_init (__ irqchip_begin);

}

__irqchip_begin 커널 IRQ 칩 테이블의 시작 주소이다. 또한이 표는 커널의 인터럽트 컨트롤러는 모든 ID 정보를 지원하고 보유하고있는 (일치하는 장치 노드). of_irq_init 함수가 실행되기 전에 시스템은 시스템의 노드에있는 모든 장치가 트리 구조를 형성하고있는 바와 같이, 각 노드가 장치 노드 장치를 나타내는 초기 장치 트리를 완료했다. of_irq_init 인터럽트 컨트롤러 노드에있는 모든 장치 노드를보고있는 트리 구조를 형성한다 (시스템이 여러 개의 인터럽트 컨트롤러를 가질 수 나무 형성을위한 컨트롤러 이유를 인터럽트 인터럽트 컨트롤러 드라이버의 모든 에 따른 시스템을 만드는 것입니다) 특정 순서를 초기화합니다. 그 루트 인터럽트 컨트롤러 노드는 각각 한 번 매치 한 컨트롤러 장치 노드 스캔 IRQ 칩 테이블 일치를 인터럽트를위한 초기화 함수가 인터럽트 컨트롤러, 인터럽트 컨트롤러와 장치 노드와 부모라고 시작 함 에서 칩 드라이버와 IRQ 인수로 컨트롤러 장치 노드를 중단. . 코드 구체적인 매칭 처리 내용 장치 트리의 모듈이며, 더 자세한 정보가있는 장치 트리 코드 분석 문서.

2, GIC 드라이버의 초기화 코드 분석

다음과 같이 (1) gic_of_init 코드는 다음과 같습니다.

int 형 __init gic_of_init (구조 device_node * 노드 구조 device_node * 부모)

{

공동 __iomem * cpu_base.

공동 __iomem * dist_base.

U32의 percpu_offset.

int 형의 IRQ.

dist_base = of_iomap (노드, 0); ---------------- 매핑 된 레지스터 어드레스 공간 GIC 분배

cpu_base = of_iomap (노드 1); ---------------- 매핑 GIC CPU 인터페이스 레지스터 주소 공간

IF (of_property_read_u32 (노드 "CPU- 오프셋", & percpu_offset)) -------- 핸들 CPU 오프셋 성. percpu_offset = 0.

gic_init_bases (gic_cnt -1, dist_base, cpu_base, percpu_offset 노드);)) ----- 주요 공정 나중에 자세히 설명

(만약!의 gic_cnt)

gic_init_physaddr (노드); ----- big.LITTLE 스위처를 지원하지 않습니다 (CONFIG_BL_SWITCHER) 시스템 함수는 비어 있습니다.

(부모) {-------- 캐스케이드 인터럽트 처리하는 경우

IRQ = irq_of_parse_and_map (노드 0); --- 분석 둘째 GIC의 인터럽트 속성 및 매핑은 IRQ 번호를 반환

gic_cascade_irq (gic_cnt, IRQ).

}

gic_cnt ++;

0을 반환합니다.

}

우리는 먼저이 함수의 매개 변수를보고 노드 초기화 파라미터는, 그 부모에게 인터럽트 컨트롤러 장치 노드 부모 파라미터 지점의 요구를 나타냅니다. 매핑 메모리 맵 GIC-400 I / O 공간에서 우리는 다만 분배와 CPU 인터페이스 레지스터의 주소 공간을 매핑하고 관련 레지스터를 가상화 맵되어 있지 않기 때문에, GIC 드라이버의이 버전은 가상화를 지원해야하지 않습니다 (? 임베디드 플랫폼 지원 가상화 현실적인 의미에서 후속 버전을 지원하는지 여부를 확인하십시오 ARM64 + GICV3 / 4 등의 플랫폼 일 필요가 있고, 가상화를 지원하는 최초는 아니다).

더 많은 CPU 오프셋을 배우고, 우리는 첫 번째 레지스터 뱅크되는지를 이해해야한다. 이른바 레지스터 뱅크 어드레스 레지스터의 여러 복사본을 제공하는 데있다. 예를 들어, 시스템은 CPU 레지스터 주소가 동일 액세스는 해당 주소가 동일하지만, 뱅크 된 레지스터 때문에 실제로는 다른 CPU는 다른 레지스터에 접근이있다 경우에는 4 개의 CPU를 가지고있다. GIC는 레지스터 뱅크하지 않는 경우, CPU의 인덱스에 따라 일련의 주소를주고, 오프셋 및 주소는 = CPU- 오프셋 * CPU-NR을 상쇄 제공해야합니다.

인터럽트 컨트롤러는 계단식 수 있습니다. 루트 GIC 위해 그 들어오는 부모는 NULL이기 때문에 코드의 계단식 일부는 실행되지 않습니다. 일반 IRQ 소스의 상위 (루트 GIC), IRQ 핸들러를 등록하기 위해 필요하다 둘째 GIC 내용은. 한편이 인터럽트 컨트롤러로 사용되는 루트 GIC가 같은 초기화 코드를 실행된다. 따라서 비 루트의 초기 GIC는 2 개의 부분으로 나누어 져있다. 한편, 또한 등록 된 인터럽트 핸들러와 같은 일반적인 장치 드라이버를 필요로 일반 인터럽트 발생 장치로 GIC. 필요한 IRQ 도메인 지식을 irq_of_parse_and_map 이해를 참조하십시오 (2)의 Linux 커널의 인터럽트 서브 시스템 : 도메인 설명 IRQ.

2) 다음과 같이 코드는 gic_init_bases :

공동 __init의 gic_init_bases (unsigned int 형의 gic_nr는 irq_start int 형

공동 __iomem * dist_base 충치 __iomem * cpu_base,

U32의 percpu_offset 구조 device_node의 * 노드)

{

irq_hw_number_t hwirq_base.

구조 gic_chip_data의 *의 GIC.

int 형 gic_irqs, irq_base, I;

GIC = & gic_data [gic_nr];

gic-> dist_base.common_base = dist_base. ---- 은행 이외의 경우 생략

gic-> cpu_base.common_base = cpu_base.

gic_set_base_accessor (GIC, gic_get_common_base);

위한 (I = 0; 나는 NR_GIC_CPU_IF을 <; 나는 +) --- 의미 gic_cpu_map의 뒷부분에서 자세하게 설명하는

gic_cpu_map [i]는 0xff로를 =;

만약 (gic_nr == 0 && (irq_start & 31)> 0) {-------------------- (A)

hwirq_base = 16.

IF (irq_start! = -1)

irq_start = (irq_start & ~ 31) +16.

} 그렇지 {

hwirq_base = 32.

}

gic_irqs = readl_relaxed (gic_data_dist_base (GIC) + GIC_DIST_CTR) & 0x1F를 ---- (B)

gic_irqs = (gic_irqs +1) * 32.

IF (gic_irqs> 1020)

= 1020 gic_irqs.

gic-> gic_irqs = gic_irqs.

gic_irqs - = hwirq_base. ---------------------------- (C)

IF (of_property_read_u32 (노드 "팔 라우팅 할 수있는 -의 IRQ"---------------- (D)

& Nr_routable_irqs)) {

irq_base = irq_alloc_descs (irq_start, 16 gic_irqs, numa_node_id ()); ------- (E)

IF (IS_ERR_VALUE (irq_base)) {

1 (WARN "사전에 할당 된 \ n을 가정하여 IRQ % d @ irq_descs를 할당 할 수 없다"

irq_start);

irq_base = irq_start.

}

gic-> 도메인 = irq_domain_add_legacy (노드 gic_irqs, irq_base, ------- (F)

hwirq_base & gic_irq_domain_ops, GIC);

} 그렇지 {

gic-> 도메인 = irq_domain_add_linear (노드 nr_routable_irqs, -------- (F)

& Gic_irq_domain_ops,

GIC);

}

IF (gic_nr == 0) {--- 세트 콜백은 [OK]를 한 번만 알림 등록 된 때문에 root 만 GIC 조작

CONFIG_SMP의 #ifdef

set_smp_cross_call (gic_raise_softirq); ------------------ (G)

register_cpu_notifier (& gic_cpu_notifier); ------------------ (H)

#endif의

set_handle_irq (gic_handle_irq); ---이 함수 이름이 좋지 않아 실제로는 아치 관련 IRQ 핸들러를 설정하는

}

gic_chip.flags | = gic_arch_extn.flags.

gic_dist_init (GIC); --------- 특정 하드웨어 초기화 코드 작성을 참조하십시오

gic_cpu_init (GIC);

gic_pm_init (GIC);

}

0 인 (A) gic_nr 로고 GIC 번호는 루트 GIC있다. hwirq는 HW 아니라 모든 IRQ 프레임 워크는 SGI위한 IRQ 번호를 가지고 Linux에지도상에서 ID GIC를 중단하고 위로의 ID GIC를 중단하는 것을 의미하고, CPU 간의 통신 사용되는 소프트웨어 인터럽트는 필요하지 않습니다 매핑 IRQ 번호 HW 인터럽트 ID를 실시하고 있습니다. 변수 hwirq_base베이스 ID를 매핑하려면 GIC에 발현하고 hwirq_base = 16의 수단은 16 SGI를 무시했다. PPI가 필요한 매핑이 아닌 시스템에서는 다른 GIC 따라서 hwirq_base 32을 =.

이 시나리오에서는 irq_start = -1, 그들은 IRQ 번호를 지정하지 않은 고 말했다. 일부 장면은 IRQ 번호, 이번에는 당신이 IRQ 번호의 일치 동작을해야 지정됩니다.

(B) 변수 gic_irqs는 인터럽트이 GIC 지원되는 최대 수를 구했다. 이 정보는 GIC_DIST_CTR 레지스터에서 낮은 5 ITLinesNumber 취득 (레지스터 이름의 V1 버전입니다, V2는 GICD_TYPER 인터럽트 컨트롤러 타입 레지스터입니다). N에 동일한 ITLinesNumber 경우 인터럽트 지원되는 최대 수는 32 (N + 1)이다. 또한 GIC 사양은 인터럽트 1020,1020-1023의 최대 개수는 특히 사용자 인터럽트 ID로 초과 할 수 없다.

그 (C) 맵을 뺄 필요는 없지만 (IRQ를 할당 할 필요가 없습니다) ID는 [OK]를 그것이 결국 합의에 gic_irqs하고 그 이름을 소중히하는 때가 인터럽트 . gic_irqs 마세 그대로 GIC는 그것을 위해 필요한만큼 할당 된 IRQ 번호입니다?

(D) 팔 of_property_read_u32 기능은 속성 값은 0이 성공적으로 반환 된 경우 변수를 nr_routable_irqs 라우팅 가능 -IRQ을 읽는다. 일부 SOC 설계는 주변 인터럽트 요청 신호 선은 GIC에 직접 연결되지 않고 크로스바 / 멀티플렉서 HW 블록을 통해 GIC에 연결되어있다. 암이 속성은 인터럽트 요청의 수를 정의하는 데 사용되는 라우팅 가능 -IRQ가 직접 그 GIC에 연결되어 있지 않다.

(E)는 GIC에 직접 연결 이러한 상황을 위해, 우리는 irq_alloc_descs이 디스크립터를 인터럽트 호출하여 할당해야합니다. irq_start가 0보다 크지 만, 그 우리의 시나리오를 위해 할당 된 지정된 IRQ 번호의 경우는 -1에 동일한 irq_start 때문에 IRQ 번호를 지정하지 않습니다. 당신은 IRQ 번호를 지정하지 않으면 검색해야 두 번째 매개 변수는 16 IRQ 번호의 첫 번째 검색입니다. gic_irqs IRQ 번호의 지정된 숫자가 할당된다. 제대로 인터럽트 설명자에 할당하지 않으면 프로그램은 전에 준비하고 있을지도 모른다고 생각합니다.

(F)이 코드는 주로 시스템의 IRQ 도메인 등록 데이터 구조이다. 왜 우리는 구조 등의 데이터 구조를 irq_domain 필요한 것일까 요? Linux 커널의 관점에서 모든 외부 장치는 비동기 이벤트 인터럽트 커널은이 이벤트를 인식 할 필요가있다. 커널 인터럽트 요청 장치를 식별하기 위해 IRQ 번호를 사용합니다. IRQ 번호를 사용하면 인터럽트 디스크립터 (구조 irq_desc)로 이동할 수 있습니다. 그러나 인터럽트 컨트롤러의 목적을 위해, 그것은 IRQ 번호를 몰라, 그냥 (인코딩 하드웨어 인터럽트 번호로 불리고있는 암호화 된 인터럽트 소스에 대한 지원을 위해 인터럽트 컨트롤러) HW 숫자를 중단 알고있다. 다른 ID를 가진 다른 소프트웨어 모듈은 인터럽트 소스를 식별하기 때문에, 당신은 매핑 된해야합니다. IRQ 번호, 거기에 하드웨어 인터럽트 번호를 매핑하려면? 이것은 커널 구조 irq_domain로 정의되고 번역 개체가 필요합니다.

각 인터럽트 컨트롤러가 IRQ 도메인을 형성하고, 하류 소스 interrut 분석하기위한 책임이 있습니다. 당신은 상황이 인터럽트 컨트롤러를 직렬 연결 한 경우는 root 이외의 인터럽트 컨트롤러의 인터럽트 컨트롤러는 일반적으로 인터럽트 소스 도메인 IRQ 부모이다. 다음과 같이 정의 된 struct irq_domain :

2) 다음과 같이 코드는 gic_init_bases :

공동 __init의 gic_init_bases (unsigned int 형의 gic_nr는 irq_start int 형

공동 __iomem * dist_base 충치 __iomem * cpu_base,

U32의 percpu_offset 구조 device_node의 * 노드)

{

irq_hw_number_t hwirq_base.

구조 gic_chip_data의 *의 GIC.

int 형 gic_irqs, irq_base, I;

GIC = & gic_data [gic_nr];

gic-> dist_base.common_base = dist_base. ---- 은행 이외의 경우 생략

gic-> cpu_base.common_base = cpu_base.

gic_set_base_accessor (GIC, gic_get_common_base);

위한 (I = 0; 나는 NR_GIC_CPU_IF을 <; 나는 +) --- 의미 gic_cpu_map의 뒷부분에서 자세하게 설명하는

gic_cpu_map [i]는 0xff로를 =;

만약 (gic_nr == 0 && (irq_start & 31)> 0) {-------------------- (A)

hwirq_base = 16.

IF (irq_start! = -1)

irq_start = (irq_start & ~ 31) +16.

} 그렇지 {

hwirq_base = 32.

}

gic_irqs = readl_relaxed (gic_data_dist_base (GIC) + GIC_DIST_CTR) & 0x1F를 ---- (B)

gic_irqs = (gic_irqs +1) * 32.

IF (gic_irqs> 1020)

= 1020 gic_irqs.

gic-> gic_irqs = gic_irqs.

gic_irqs - = hwirq_base. ---------------------------- (C)

IF (of_property_read_u32 (노드 "팔 라우팅 할 수있는 -의 IRQ"---------------- (D)

& Nr_routable_irqs)) {

irq_base = irq_alloc_descs (irq_start, 16 gic_irqs, numa_node_id ()); ------- (E)

IF (IS_ERR_VALUE (irq_base)) {

1 (WARN "사전에 할당 된 \ n을 가정하여 IRQ % d @ irq_descs를 할당 할 수 없다"

irq_start);

irq_base = irq_start.

}

gic-> 도메인 = irq_domain_add_legacy (노드 gic_irqs, irq_base, ------- (F)

hwirq_base & gic_irq_domain_ops, GIC);

} 그렇지 {

gic-> 도메인 = irq_domain_add_linear (노드 nr_routable_irqs, -------- (F)

& Gic_irq_domain_ops,

GIC);

}

IF (gic_nr == 0) {--- 세트 콜백은 [OK]를 한 번만 알림 등록 된 때문에 root 만 GIC 조작

CONFIG_SMP의 #ifdef

set_smp_cross_call (gic_raise_softirq); ------------------ (G)

register_cpu_notifier (& gic_cpu_notifier); ------------------ (H)

#endif의

set_handle_irq (gic_handle_irq); ---이 함수 이름이 좋지 않아 실제로는 아치 관련 IRQ 핸들러를 설정하는

}

gic_chip.flags | = gic_arch_extn.flags.

gic_dist_init (GIC); --------- 특정 하드웨어 초기화 코드 작성을 참조하십시오

gic_cpu_init (GIC);

gic_pm_init (GIC);

}

0 인 (A) gic_nr 로고 GIC 번호는 루트 GIC있다. hwirq는 HW 아니라 모든 IRQ 프레임 워크는 SGI위한 IRQ 번호를 가지고 Linux에지도상에서 ID GIC를 중단하고 위로의 ID GIC를 중단하는 것을 의미하고, CPU 간의 통신 사용되는 소프트웨어 인터럽트는 필요하지 않습니다 매핑 IRQ 번호 HW 인터럽트 ID를 실시하고 있습니다. 변수 hwirq_base베이스 ID를 매핑하려면 GIC에 발현하고 hwirq_base = 16의 수단은 16 SGI를 무시했다. PPI가 필요한 매핑이 아닌 시스템에서는 다른 GIC 따라서 hwirq_base 32을 =.

이 시나리오에서는 irq_start = -1, 그들은 IRQ 번호를 지정하지 않은 고 말했다. 일부 장면은 IRQ 번호, 이번에는 당신이 IRQ 번호의 일치 동작을해야 지정됩니다.

(B) 변수 gic_irqs는 인터럽트이 GIC 지원되는 최대 수를 구했다. 이 정보는 GIC_DIST_CTR 레지스터에서 낮은 5 ITLinesNumber 취득 (레지스터 이름의 V1 버전입니다, V2는 GICD_TYPER 인터럽트 컨트롤러 타입 레지스터입니다). N에 동일한 ITLinesNumber 경우 인터럽트 지원되는 최대 수는 32 (N + 1)이다. 또한 GIC 사양은 인터럽트 1020,1020-1023의 최대 개수는 특히 사용자 인터럽트 ID로 초과 할 수 없다.

그 (C) 맵을 뺄 필요는 없지만 (IRQ를 할당 할 필요가 없습니다) ID는 [OK]를 그것이 결국 합의에 gic_irqs하고 그 이름을 소중히하는 때가 인터럽트 . gic_irqs 마세 그대로 GIC는 그것을 위해 필요한만큼 할당 된 IRQ 번호입니다?

(D) 팔 of_property_read_u32 기능은 속성 값은 0이 성공적으로 반환 된 경우 변수를 nr_routable_irqs 라우팅 가능 -IRQ을 읽는다. 일부 SOC 설계는 주변 인터럽트 요청 신호 선은 GIC에 직접 연결되지 않고 크로스바 / 멀티플렉서 HW 블록을 통해 GIC에 연결되어있다. 암이 속성은 인터럽트 요청의 수를 정의하는 데 사용되는 라우팅 가능 -IRQ가 직접 그 GIC에 연결되어 있지 않다.

(E)는 GIC에 직접 연결 이러한 상황을 위해, 우리는 irq_alloc_descs이 디스크립터를 인터럽트 호출하여 할당해야합니다. irq_start가 0보다 크지 만, 그 우리의 시나리오를 위해 할당 된 지정된 IRQ 번호의 경우는 -1에 동일한 irq_start 때문에 IRQ 번호를 지정하지 않습니다. 당신은 IRQ 번호를 지정하지 않으면 검색해야 두 번째 매개 변수는 16 IRQ 번호의 첫 번째 검색입니다. gic_irqs IRQ 번호의 지정된 숫자가 할당된다. 제대로 인터럽트 설명자에 할당하지 않으면 프로그램은 전에 준비하고 있을지도 모른다고 생각합니다.

(F)이 코드는 주로 시스템의 IRQ 도메인 등록 데이터 구조이다. 왜 우리는 구조 등의 데이터 구조를 irq_domain 필요한 것일까 요? Linux 커널의 관점에서 모든 외부 장치는 비동기 이벤트 인터럽트 커널은이 이벤트를 인식 할 필요가있다. 커널 인터럽트 요청 장치를 식별하기 위해 IRQ 번호를 사용합니다. IRQ 번호를 사용하면 인터럽트 디스크립터 (구조 irq_desc)로 이동할 수 있습니다. 그러나 인터럽트 컨트롤러의 목적을 위해, 그것은 IRQ 번호를 몰라, 그냥 (인코딩 하드웨어 인터럽트 번호로 불리고있는 암호화 된 인터럽트 소스에 대한 지원을 위해 인터럽트 컨트롤러) HW 숫자를 중단 알고있다. 다른 ID를 가진 다른 소프트웨어 모듈은 인터럽트 소스를 식별하기 때문에, 당신은 매핑 된해야합니다. IRQ 번호, 거기에 하드웨어 인터럽트 번호를 매핑하려면? 이것은 커널 구조 irq_domain로 정의되고 번역 개체가 필요합니다.

각 인터럽트 컨트롤러가 IRQ 도메인을 형성하고, 하류 소스 interrut 분석하기위한 책임이 있습니다. 당신은 상황이 인터럽트 컨트롤러를 직렬 연결 한 경우는 root 이외의 인터럽트 컨트롤러의 인터럽트 컨트롤러는 일반적으로 인터럽트 소스 도메인 IRQ 부모이다. 다음과 같이 정의 된 struct irq_domain :

GIC_DIST_TARGETn (Interrupt Processor Targets Registers) 정도 우 Distributor HW block 중 能控 제 송 达的 CPU interface, 并不 옳다고 특히 CPU, 如果 구체적 实现 중 CPU interface 합 CPU 옳다고 严格 按照에서 图中 那样 열한 对应, 那么 GIC_DIST_TARGET 전송 达了 CPU Interface n, 也就 是送 达了 CPU n. 당연히 现实 미필적 如你 所愿, 那么 怎样 온 获取 这个 CPU 인 mask 呢? 우리 们知 도로 SGI 화 PPI 불 수요 사용 GIC_DIST_TARGET 공 제 target CPU. SGI 송 达目 标 CPU도 자기 특유으로 寄存 기 온 공 제 (Software Generated Interrupt Register) 对于 PPI, 其是 CPU 사유적이고 원인 此不 수요 공 제 target CPU. GIC_DIST_TARGET0 ~ GIC_DIST_TARGET7 是控制 0 ~ 31 这 32 개 interrupt ID (SGI 화 PPI)으로 target CPU적이고 但是 实际에서 SGI 화 PPI 옳다고 불 수요 공 제 target CPU적이고 因此, 这些 寄存 기 시정 read only적이고 读取 这些 寄存 기 返回으로 就是 cpu mask 值. 假设 CPU0 接在 CPU interface 4 위, 那么 运行 주한 CPU 0 上 的 정도 서론 주한 读 GIC_DIST_TARGET0 ~ GIC_DIST_TARGET7으로 时候, 返回으로 就是 0b00010000.

당연히, 由于 GIC-400 척지지 8 개 CPU, 因此 CPU mask 值只 수요 8bit, 但是 寄存 기 GIC_DIST_TARGETn 返回 32 개 bit으로 值, 怎么 对应? 很简 单, cpu mask 무게 复 네 차 취 OK 료. 양해 료 这些 지식인 회 头看 대 码就 很简 单了.

(c) step (b) 중 获取 료 8 개 bit으로 cpu mask 值 통 过简 单的 copy, 扩充 为 32 개 bit, 당 8 개 bit 都是 cpu mask으로 值, 这么 做是 为了 아래 한 步设 정 소유 IRQ (对于 GIC 而言 就是 SPI 类型으로 중단)으로 CPU mask.

(d) 设定 每个 SPI 类型으로 중단 都 是 척 송 达该 CPU.

(e) 배치 GIC distributor으로 其他 寄存 기, 대 码如 아래 :

void __init gic_dist_config (void __iomem * base, int gic_irqs, void (* sync_access) (void)