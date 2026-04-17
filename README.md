# tcp-ip-socket-2026
IoT개발자 과정 TCP/IP 소켓 프로그래밍

# 🐧 Linux & C Programming 실습 핵심 요약 (2026-04-17)

## 1. 개발 환경 세팅 (VMware & Ubuntu)
* **VMware Workstation**: 윈도우 환경 위에 가상 머신을 구축하여 우분투(Ubuntu 24.04) 실행.
* **Terminator (터미널 에뮬레이터)**:
    * **화면 분할**: `Ctrl + Shift + O` (가로), `Ctrl + Shift + E` (세로).
    * **설정 최적화**: `Preferences`에서 타이틀바 제거 및 레이아웃 설정 가능.
    * **창 이동**: `Alt` + 마우스 드래그.

[Image of Linux Terminator terminal split screen with code and compilation output]

---

## 2. Nano 에디터 설정 및 활용
텍스트 에디터 `nano`를 개발용으로 최적화하기 위해 `~/.nanorc` 파일을 수정합니다.

### 주요 설정 (`nano ~/.nanorc`)
```bash
set linenumbers      # 왼쪽 줄 번호 표시
set mouse            # 마우스 클릭으로 커서 이동 및 스크롤 허용
set tabsize 3        # 탭 간격을 4칸으로 고정
set autoindent       # 줄 바꿈 시 자동 들여쓰기 적용