# tcp-ip-socket-2026
IoT개발자 과정 TCP/IP 소켓 프로그래밍

## Day01

### 1. 개발 환경 세팅 (VMware & Ubuntu)
- **VMware Workstation**: 윈도우 환경 위에 가상 머신을 구축하여 우분투(Ubuntu 24.04) 실행.
- **Terminator (터미널 에뮬레이터)**:
    - **화면 분할**: `Ctrl + Shift + O` (가로), `Ctrl + Shift + E` (세로).
    - **설정 최적화**: `Preferences`에서 타이틀바 제거 및 레이아웃 설정 가능.
    - **창 이동**: `Alt` + 마우스 드래그.

---

### 2. Nano 에디터 설정 및 활용
텍스트 에디터 `nano`를 개발용으로 최적화하기 위해 `~/.nanorc` 파일을 수정합니다.

#### 주요 설정 (`nano ~/.nanorc`)
```bash
set linenumbers      # 왼쪽 줄 번호 표시
set mouse            # 마우스 클릭으로 커서 이동 및 스크롤 허용
set tabsize 3        # 탭 간격을 3칸으로 고정
set autoindent       # 줄 바꿈 시 자동 들여쓰기 적용
```

---

## Day02

### Network Programming: Core Concepts
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
- **Network Byte Order**: 네트워크 전송을 위한 컴퓨터 내부의 정수 형태 (Big-Endian 방식)

---

## Day03

### 1. `INADDR_ANY` (자동 IP 할당)
- 서버 소켓을 주소와 묶을 때(`bind`) 사용하는 상수로, "내 컴퓨터에 들어오는 랜선(IP 주소)이 무엇이든 다 받아들이겠다"는 의미입니다.
- 특정 IP 주소를 직접 적지 않아도 컴퓨터의 사용 가능한 IP를 알아서 찾아서 연결해 주므로 편리합니다.

### 2. 소켓에 인터넷 주소 할당하기
- 소켓(전화기)을 만든 후, 내가 사용할 IP 주소와 포트 번호(전화번호)를 부여하는 과정입니다.
- `sockaddr_in` 구조체에 주소 정보(IPv4, IP, 포트)를 채워 넣고, `bind()` 함수를 호출해 소켓과 주소를 연결합니다.

### 3. 스레드(Thread) 구조
- 프로세스(프로그램) 안에서 실제로 일을 하는 **'작은 일꾼(실행 흐름)'**입니다.
- 한 프로세스 안에 여러 스레드가 있으면, 이 일꾼들은 각자의 사물함(**스택 영역/지역변수**)만 따로 쓰고, 거실(**코드, 전역변수, 힙 영역**)은 다 함께 공유하면서 일합니다.

### 4. 부모(Parent)와 자식(Child) 프로세스
- `fork()` 함수를 실행하면 현재 실행 중인 프로그램(부모)을 그대로 복사한 복제 인간 같은 프로그램(자식)이 하나 더 태어납니다.
- 태어난 순간부터 둘은 완전히 남남이 되기 때문에, 한쪽에서 변수 값을 바꿔도 다른 쪽에는 전혀 영향을 주지 않습니다.

### 5. 지역변수와 전역변수 (스레드 관점)
- **지역변수**: 각 일꾼(스레드)의 개인 사물함(스택)에 들어가므로 다른 일꾼이 건드릴 수 없어 안전합니다.
- **전역변수**: 거실(데이터 영역)에 꺼내놓은 물건이라 모든 일꾼(스레드)이 동시에 만질 수 있습니다. 서로 먼저 고치려고 싸우면 데이터가 망가지므로 순서를 정해주는 동기화가 필요합니다.

#### Day03 연습 예제: 프로세스 복사(`fork`)와 스레드 생성(`pthread`)의 메모리 차이 알기
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/wait.h>

int global_var = 10; // 전역 변수 (거실 물건)

void* thread_main(void* arg) {
    int local_var = 20; // 지역 변수 (개인 사물함 물건)
    global_var += 5;
    local_var += 5;
    printf("[스레드 일꾼] 전역변수: %d, 지역변수: %d\n", global_var, local_var);
    return NULL;
}

int main() {
    pid_t pid;
    pthread_t t_id;

    printf("--- 1. fork() 프로세스 복사 테스트 ---\n");
    pid = fork();
    if (pid == 0) { // 자식 프로세스방
        global_var += 100; // 부모와 분리된 방의 변수를 바꿈
        printf("[자식] 내 방의 전역변수 변경: %d\n", global_var);
        exit(0);
    } else { // 부모 프로세스방
        wait(NULL); // 자식이 끝날 때까지 대기
        printf("[부모] 자식이 끝난 후 내 방의 전역변수 확인: %d (그대로임)\n", global_var);
    }

    printf("\n--- 2. pthread_create() 스레드 공유 테스트 ---\n");
    pthread_create(&t_id, NULL, thread_main, NULL);
    pthread_join(t_id, NULL); // 스레드 일꾼이 일 다 할 때까지 대기
    printf("[메인 스레드] 일꾼이 다녀간 후 전역변수 확인: %d (값이 바뀜!)\n", global_var);

    return 0;
}
```

---

## Day04

### 1. 프로세스란?
- 컴퓨터 바탕화면에 있는 아이콘(프로그램)을 더블 클릭해서 메모리에 올려 **현재 실행 중인 상태**로 만든 것입니다.

### 2. 멀티프로세스란?
- 하나의 프로그램을 하려고 여러 개의 프로세스(컴퓨터 안의 독립된 방)를 띄워 동시에 일을 처리하는 방식입니다.
- 서로 방이 완전히 격리되어 있어서 하나가 에러로 뻗어도 다른 방은 멀쩡하다는 큰 장점이 있지만, 컴퓨터 리소스를 많이 먹고 방을 바꿀 때(문맥 교환) 시간이 걸립니다.

### 3. 멀티스레드란?
- 방 하나(프로세스)를 같이 쓰면서 여러 명의 일꾼(스레드)이 동시에 일하는 방식입니다.
- 거실을 같이 쓰니까 가볍고 대화(데이터 주고받기)가 엄청 빠르지만, 일꾼 한 명이 실수해서 방 전체를 태워 먹으면(메모리 오류 등) 다 같이 종료되는 위험이 있습니다.

### 4. 프로세스와 스레드의 핵심 차이점
- **프로세스**: 아예 독립된 집을 지어 사는 것 (자원 공유 안 함, 안전함, 무거움).
- **스레드**: 한 집안에서 방만 따로 쓰고 거실은 같이 쓰는 것 (자원 공유함, 위험 요소 있음, 가볍고 빠름).

### 5. 단방향 파이프와 양방향 파이프 (IPC 통신)
- **단방향 파이프**: 종이컵 전화기처럼 한쪽은 말만 하고(`Write`), 한쪽은 듣기만(`Read`) 할 수 있는 통로입니다.
- **양방향 통신 하려면?**: 파이프 하나로 주고받기를 동시에 하려고 하면 내가 말한 게 내 귀로 다시 들어오는 문제가 생깁니다. 따라서 주고받는 통신을 제대로 하려면 **무조건 파이프를 2개 만들어서 엇갈리게 연결**해야 합니다.

#### Day04 연습 예제: 파이프(`pipe`)를 사용해 부모와 자식이 대화 주고받기
```c
#include <stdio.h>
#include <unistd.h>

int main() {
    int fds[2]; // fds[0]은 듣는 구멍(Read), fds[1]은 말하는 구멍(Write)
    char send_msg[] = "자식아 공부 열심히 해라!";
    char recv_buf[50];
    pid_t pid;

    if (pipe(fds) < 0) {
        perror("파이프 만들기 실패");
        return -1;
    }

    pid = fork();
    if (pid == 0) { // 자식 프로세스
        close(fds[1]); // 자식은 말 안 할 거니까 쓰기 구멍 닫기
        read(fds[0], recv_buf, sizeof(recv_buf)); // 부모가 말할 때까지 기다렸다가 들음
        printf("[자식 수신] 부모님이 하신 말씀: %s\n", recv_buf);
        close(fds[0]);
    } else { // 부모 프로세스
        close(fds[0]); // 부모는 안 들을 거니까 읽기 구멍 닫기
        write(fds[1], send_msg, sizeof(send_msg)); // 파이프에 대고 말하기
        printf("[부모 송신] 자식에게 잔소리를 전송함.\n");
        close(fds[1]);
    }
    return 0;
}
```

---

## Day05

### 1. 멀티스레드 동기화
- 공유 자원(전역변수 등)을 여러 일꾼이 동시에 건드리지 못하게 **차례차례 줄을 세우는 약속**입니다.

### 2. 임계영역 (Critical Section)
- 코드 중에서 "여기는 전역변수를 고치는 곳이라 여러 스레드가 동시에 들어가면 절대로 안 되는 위험한 구역"을 뜻합니다.

### 3. 뮤텍스(MUTEX)와 세마포어(SEMAPHORE)
- **MUTEX (뮤텍스)**: 화장실의 **'하나뿐인 열쇠'** 같은 개념입니다. 내가 열쇠로 문을 잠그고(`Lock`) 들어가면, 내가 나오면서 열쇠를 열어줄(`Unlock`) 때까지 다른 사람은 문 앞에서 무조건 기다려야 합니다.
- **SEMAPHORE (세마포어)**: 공공 주차장의 **'주차 가능 대수(카운터)'** 같은 개념입니다. 예를 들어 값이 3이면 동시에 자동차 3대까지 들어갈 수 있고, 자리가 꽉 차면 차 한 대가 빠질 때까지 대기해야 합니다.

### 4. I/O 멀티플렉싱 (Multiplexing)
- 일꾼(프로세스/스레드)을 여러 명 고용하는 건 돈(메모리)이 너무 많이 드니까, **능력 좋은 지배인 한 명만 두고 여러 개의 소켓(클라이언트 요청)을 혼자서 감시하고 처리**하는 똑똑한 기술입니다.

### 5. 멀티플렉싱 방식 비교: `select` vs `epoll`
- **`select` (구식 방식)**: 지배인이 문이 100개 있으면 1번 문부터 100번 문까지 맨날 하나씩 다 두드려보면서 "손님 계세요?" 하고 물어보는 방식입니다. 손님이 많아질수록 지배인이 지쳐서 느려집니다.
- **`epoll` (신식 리눅스 가속 방식)**: 손님이 용무가 생기면 알아서 지배인 모니터에 벨을 누르는 방식입니다. 지배인은 벨이 울린(이벤트가 발생한) 문만 딱 확인해서 처리하므로 손님이 아무리 많아도 엄청나게 빠릅니다.

#### Day05 연습 예제: 뮤텍스 열쇠(`pthread_mutex`)로 전역변수 안전하게 더하기
```c
#include <stdio.h>
#include <pthread.h>

long long total_count = 0; // 스레드들이 공유할 전역변수 (임계영역 타겟)
pthread_mutex_t lock_key;  // 동기화용 뮤텍스 열쇠 변수

void* run_increment(void* arg) {
    for (int i = 0; i < 50000; i++) {
        pthread_mutex_lock(&lock_key);   // 1. 내가 들어왔으니 문 잠그기 (Lock)
        
        total_count++;                   // 2. 임계영역 작업 진행 (안전 구역)
        
        pthread_mutex_unlock(&lock_key); // 3. 나 가니까 문 열어주기 (Unlock)
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;

    pthread_mutex_init(&lock_key, NULL); // 뮤텍스 열쇠 사용 준비

    pthread_create(&t1, NULL, run_increment, NULL);
    pthread_create(&t2, NULL, run_increment, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    pthread_mutex_destroy(&lock_key); // 사용 끝난 뮤텍스 열쇠 버리기

    // 뮤텍스로 잘 잠갔다 풀었기 때문에 정확히 100,000이 출력됩니다.
    printf("[실험 결과] 두 일꾼이 안전하게 더한 결과: %lld\n", total_count);
    return 0;
}
```

---

## Day06

### 1. UDP소켓 전용 입출력 함수
- UDP는 연결을 맺지 않는 비연결지향형 프로토콜이므로 `connect()`나 `accept()` 단계가 존재하지 않습니다.
- **`sendto()`**: 데이터를 전송할 때마다 패킷 내에 목적지의 주소 정보(IP, Port)를 매번 명시하여 송신합니다.
- **`recvfrom()`**: 데이터를 수신하면서 해당 패킷을 보낸 발신자의 주소 정보도 함께 받아옵니다.

### 2. 멀티캐스트
- 하나의 송신자가 특정 그룹(멀티캐스트 주소)에 가입한 다수의 수신자에게 데이터를 한 번에 효율적으로 전송하는 UDP 기반의 통신 방식입니다. 서버와 라우터의 트래픽 부담을 덜어줍니다.

### 3. 테스트 (웹서버에 접속했을 때 파일 안에 있는 JPG그림을 웹에 띄우기)
- 웹 브라우저 클라이언트가 HTTP 요청을 보낼 때, 서버 측에서 단순히 파일 데이터만 보내면 정상적으로 출력되지 않습니다.
- HTTP 응답 프로토콜 규칙에 따라 헤더 영역에 `Content-Type: image/jpeg`와 이미지의 크기(`Content-Length`)를 명시하고, 빈 줄(`\r\n\r\n`)을 넣은 뒤 실제 이미지의 바이너리(Binary) 바이트 데이터를 순서대로 전송해야 브라우저 화면에 정상적으로 그림이 렌더링됩니다.

```c
// [헤더 파일 포함 영역]
// 표준 입출력(printf 등), 메모리 할당(malloc 등), 문자열 처리(strcmp 등)를 위한 기본 C 라이브러리들입니다.
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 유닉스 표준 시스템 콜(read, write, close, sleep 등)을 사용하기 위한 헤더입니다.
#include <unistd.h>

// 인터넷 프로토콜(IP) 주소 변환(htons 등)과 소켓 구조체 정의를 위한 네트워크 라이브러리입니다.
#include <arpa/inet.h>
#include <sys/socket.h>

// [상수 정의 영역]
#define PORT 8080        // 웹 서버가 사용할 통신 포트 번호입니다. 브라우저는 기본적으로 80 포트를 쓰지만, 테스트용이므로 8080을 씁니다.
#define BUFFER_SIZE 2048 // 브라우저가 보내는 요청(Request) 메세지를 담을 바구니(버퍼)의 크기입니다.

// [전역 변수: HTML 응답 데이터]
// 브라우저가 처음 접속했을 때 보내줄 "편지" 전체 내용입니다.
// HTTP 규칙에 따라 '헤더(상태코드, 서버정보, 컨텐츠타입)'를 먼저 쓰고, 빈 줄(\r\n\r\n)을 넣은 뒤 '본문(HTML)'을 씁니다.
char webpage[] = "HTTP/1.1 200 OK\r\n"
                 "Server:Linux Web Server\r\n"
                 "Content-Type: text/html; charset=UTF-8\r\n\r\n"
                 "<!DOCTYPE html>\r\n"
                 "<html><head><title> My Web Page </title>\r\n"
                 "<style>body {background-color: #FFFF00 }</style></head>\r\n"
                 "<body><center><h1>Hello world!!</h1><br>\r\n"
                 "<img src=\"AUDI%20Q7.jpg\"></center></body></html>\r\n";

int main() {
    // [변수 선언 영역]
    int server_fd;         // 서버 측의 문지기 역할을 할 메인 소켓(파일 디스크립터)
    int new_socket;        // 실제 클라이언트(브라우저)와 1:1로 통신할 전용 소켓
    struct sockaddr_in address; // IP 주소와 포트 번호 등 네트워크 주소 정보를 담을 구조체
    int opt = 1;           // 소켓 옵션 설정을 위한 변수 (보통 1을 넣어 활성화/참 상태로 만듦)
    int addrlen = sizeof(address); // 주소 구조체의 크기를 저장 (accept 함수에서 요구함)
    char buffer[BUFFER_SIZE] = {0}; // 클라이언트의 메세지를 저장할 버퍼를 0(NULL)으로 초기화

    // [1단계: 소켓 생성 (전화기 개통)]
    // AF_INET: IPv4 인터넷 프로토콜 사용
    // SOCK_STREAM: 연결 지향형 TCP 프로토콜 사용
    // 0: 운영체제가 프로토콜(TCP)을 자동 선택하게 함
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("소켓 생성 실패"); // 에러 원인을 콘솔에 출력
        exit(EXIT_FAILURE);       // 프로그램 강제 종료
    }

    // [옵션 설정: 포트 재사용 허용]
    // 개발 중에 서버를 껐다 켰다 하다 보면 "이미 사용 중인 주소입니다(Address already in use)" 에러가 자주 납니다.
    // SO_REUSEADDR, SO_REUSEPORT 옵션을 켜서 서버가 강제 종료되어도 바로 같은 포트(8080)를 다시 쓸 수 있게 해줍니다.
    setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt));

    // [주소 구조체 세팅]
    address.sin_family = AF_INET;         // 이 주소는 IPv4 체계를 따른다.
    address.sin_addr.s_addr = INADDR_ANY; // INADDR_ANY: 내 컴퓨터의 어떤 IP로 접속하든 다 받아주겠다.
    address.sin_port = htons(PORT);       // htons: 우리가 아는 포트 번호(8080)를 네트워크가 이해하는 바이트 순서로 변환.

    // [2단계: 바인딩 (전화기에 전화번호 부여)]
    // 위에서 만든 소켓(server_fd)에 방금 세팅한 주소 정보(IP, 포트)를 딱 묶어줍니다(bind).
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("바인딩 실패");
        exit(EXIT_FAILURE);
    }

    // [3단계: 리슨 (대기 상태 돌입)]
    // 소켓을 수신 대기 상태로 만듭니다. 숫자 3은 '동시에 접속 요청이 오면 최대 3명까지 대기열(백로그)에 세우겠다'는 뜻입니다.
    if (listen(server_fd, 3) < 0) {
        perror("리슨 실패");
        exit(EXIT_FAILURE);
    }

    printf("웹 서버가 %d 포트에서 대기 중입니다...\n", PORT);

    // [4단계: 무한 루프 (웹 서버의 심장)]
    // 서버는 24시간 돌아가야 하므로 무한 루프를 돕니다. 한 명 처리하고 끝나는 게 아니라 다음 접속자를 계속 받기 위함입니다.
    while(1) {
        // [연결 수락 (전화 받기)]
        // 클라이언트가 접속할 때까지 여기서 멈춰서(Blocking) 기다립니다.
        // 접속이 들어오면, 문지기 소켓(server_fd)은 계속 입구를 지키고, 실제 통신을 담당할 새 소켓(new_socket)을 반환합니다.
        if ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) < 0) {
            perror("연결 수락 실패");
            exit(EXIT_FAILURE);
        }

        // 통신을 시작하기 전 버퍼를 깨끗하게 비웁니다. (이전 클라이언트 데이터가 남아있을 수 있으므로)
        memset(buffer, 0, BUFFER_SIZE);
        
        // [요청 읽기] 브라우저가 보낸 HTTP Request 데이터를 버퍼로 읽어옵니다.
        read(new_socket, buffer, BUFFER_SIZE - 1);

        // [요청 분석 (파싱)]
        char method[16] = {0}; // GET인지 POST인지 담을 변수
        char path[256] = {0};  // 어디로 접속했는지(경로) 담을 변수
        // 버퍼에 담긴 문자열에서 공백을 기준으로 첫 번째 단어(method)와 두 번째 단어(path)를 뽑아냅니다.
        sscanf(buffer, "%s %s", method, path); 

        printf("수신된 요청: %s %s\n", method, path);

        // 브라우저가 데이터를 달라고 하는 'GET' 요청일 경우에만 처리합니다.
        if (strcmp(method, "GET") == 0) {
            
            // 1. 주소창에 포트까지만 치고 들어왔을 때 (예: localhost:8080/)
            if (strcmp(path, "/") == 0) {
                // 아까 만들어둔 webpage(HTML) 문자열을 클라이언트(new_socket)에게 전송합니다.
                write(new_socket, webpage, strlen(webpage));
                printf("=> HTML 웹 페이지 전송 완료\n");
            } 
            // 2. 브라우저가 HTML을 읽다가 <img src="...">를 보고 이미지를 달라고 재요청했을 때
            else if (strcmp(path, "/AUDI%20Q7.jpg") == 0) {
                
                // 파일을 바이너리 읽기 모드("rb")로 엽니다. (이미지는 글자가 아니기 때문)
                FILE *file = fopen("./TEST/AUDI Q7.jpg", "rb"); 
                
                if (file == NULL) { // 파일이 없는 경우 방어 코드
                    char *not_found = "HTTP/1.1 404 Not Found\r\n\r\n";
                    write(new_socket, not_found, strlen(not_found));
                    printf("=> 이미지 파일을 찾을 수 없음\n");
                } else {
                    // [파일 크기 알아내기]
                    fseek(file, 0, SEEK_END);    // 파일의 맨 끝으로 커서를 이동
                    long file_size = ftell(file); // 현재 커서 위치(바이트 수 = 파일 크기)를 저장
                    fseek(file, 0, SEEK_SET);    // 파일을 읽기 위해 커서를 다시 맨 앞으로 이동

                    // [이미지용 HTTP 응답 헤더 만들기]
                    char header[256];
                    // 브라우저에게 "이건 jpeg 이미지야(Content-Type), 크기는 이만큼이야(Content-Length)"라고 미리 알려줍니다.
                    sprintf(header, 
                            "HTTP/1.1 200 OK\r\n"
                            "Content-Type: image/jpeg\r\n"
                            "Content-Length: %ld\r\n\r\n", file_size);
                    // 헤더를 먼저 전송합니다.
                    write(new_socket, header, strlen(header));

                    // [실제 이미지 데이터 전송]
                    // 파일 크기만큼 동적으로 메모리를 할당합니다.
                    char *img_buffer = malloc(file_size);
                    if (img_buffer) {
                        // 파일에서 메모리(img_buffer)로 한 번에 데이터를 읽어옵니다.
                        fread(img_buffer, 1, file_size, file);
                        // 메모리에 담긴 이미지 바이트 데이터를 소켓을 통해 브라우저로 전송합니다.
                        write(new_socket, img_buffer, file_size);
                        // 다 썼으면 메모리 누수를 막기 위해 반환합니다.
                        free(img_buffer);
                    }
                    fclose(file); // 열었던 파일을 닫아줍니다.
                    printf("=> 이미지 파일 전송 완료\n");
                }
            }
        }
        
        // [5단계: 연결 종료]
        // 하나의 요청(HTML 보내기 혹은 이미지 보내기) 처리가 끝났으므로 소켓을 닫아 통신을 깔끔하게 끊습니다.
        // 연결이 닫혀야 브라우저가 "아, 다운로드가 끝났구나" 하고 로딩 표시를 멈춥니다.
        close(new_socket);
    } // while(1) 끝: 바로 다시 위로 올라가서 다음 접속(accept)을 기다립니다.

    return 0; // 정상 종료
}
```