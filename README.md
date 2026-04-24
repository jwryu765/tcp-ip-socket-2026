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

## Day03
#### inaddr any
#### 소켓에 인터넷 주소 할당하기
#### thread 구조
#### parent와 child
#### 지역변수와 전역변수

## Day04
#### 프로세스란?
#### 멀티프로세스란?
#### 멀티스레드
#### 프로세서와 스레드의 차이
#### 단방향 파이프와 양방향 파이프

## Day05
#### 멀티스레드 동기화
#### 임계영역
#### 동기화처리 기법(MUTEX,SEMAPHORE)
#### IO 멀티플레싱
#### 멀티프로세서의 단점
#### 멀티플레싱 - select
#### 멀티플레싱 - epoll

## Day06
#### UDP소켓 전용 입출력 함수
#### 멀티캐스트
#### 테스트(웹서버에 접속했을떼 파일 안에 있는 JPG그림을 웹에 띄우기)

```
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

    return 0; // 정상 종료 (하지만 while 무한루프 때문에 실제로는 도달하지 않는 코드입니다)
}
```
