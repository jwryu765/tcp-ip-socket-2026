# tcp-ip-socket-2026
IoT개발자 과정 TCP/IP 소켓 프로그래밍

## Day01

### 1. 개발 환경 세팅 (VMware & Ubuntu)
- **VMware Workstation**: 윈도우 환경 위에 가상 머신을 구축하여 우분투(Ubuntu 24.04) 실행.
- **Terminator (터미널 에뮬레이터)**:
    - **화면 분할**: `Ctrl + Shift + O` (가로), `Ctrl + Shift + E` (세로).
    - **설정 최적화**: `Preferences`에서 타이틀바 제거 및 레이아웃 설정 가능.
    - **창 이동**: `Alt` + 마우스 드래그.

[Image of Linux Terminator terminal split screen with code and compilation output]

---

### 2. Nano 에디터 설정 및 활용
텍스트 에디터 `nano`를 개발용으로 최적화하기 위해 `~/.nanorc` 파일을 수정합니다.

#### 주요 설정 (`nano ~/.nanorc`)
```bash
set linenumbers      # 왼쪽 줄 번호 표시
set mouse            # 마우스 클릭으로 커서 이동 및 스크롤 허용
set tabsize 3        # 탭 간격을 4칸으로 고정
set autoindent       # 줄 바꿈 시 자동 들여쓰기 적용
```
## Day02

### 🌐 Network Programming: Core Concepts

네트워크 프로그래밍과 소켓(Socket) 통신의 핵심 개념을 정리한 문서입니다.

#### 1. 프로토콜 (Protocol)
컴퓨터 간에 데이터를 주고받기 위해 약속한 **통신 규약**입니다. 서로 다른 시스템이 원활하게 통신할 수 있도록 돕는 규칙의 집합입니다.

- **TCP (Transmission Control Protocol)**: 연결 지향형 통신. 데이터의 신뢰성과 순서를 완벽하게 보장합니다. (예: 웹 브라우징, 파일 전송)
- **UDP (User Datagram Protocol)**: 비연결 지향형 통신. 신뢰성보다는 전송 속도가 중요할 때 사용합니다. 데이터 유실 가능성이 있습니다. (예: 실시간 스트리밍, 게임)

#### 2. IP 주소 (IP Address)
네트워크상에서 컴퓨터(호스트)를 식별하기 위한 **고유한 논리적 주소**입니다. 데이터 패킷이 목적지를 찾아가는 '우편번호' 역할을 합니다.

- **IPv4**: `192.168.0.1`과 같이 8비트씩 4부분(총 32비트)으로 구성된 주소 체계입니다.
- **IPv6**: IPv4의 주소 고갈 문제를 해결하기 위해 등장한 128비트 주소 체계입니다.

#### 3. IP 주소의 표현과 변환
사람은 문자열 형태를 선호하지만, 컴퓨터 내부(네트워크 통신)에서는 32비트 정수(Binary) 형태를 사용하므로 변환이 필요합니다.

- **Dotted-Decimal Notation**: 사람이 읽기 편한 문자열 형태 (예: `"127.0.0.1"`)
- **Network Byte Order**: 네트워크 전송을 위한 컴퓨터 내부의 정수 형태 (Big-Endian 방식


#### inaddr any

#### 소켓에 인터넷 주소 할당하기