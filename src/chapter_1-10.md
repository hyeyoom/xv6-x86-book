- [운영체제 구조](./chapter_1.md)
    - [물리적 리소스 추상화](./chapter_1-1.md)
    - [유저모드, 커널모드 그리고 시스템콜](./chatper_1-2.md)
    - [커널 구조](./chapter_1-3.md)
    - [프로세스 개요](./chapter_1-4.md)
    - [코드: 첫 번째 주소 공간](./chapter_1-5.md)
    - [코드: 첫 번째 프로세스 생성](./chapter_1-6.md)
    - [코드: 첫 번째 프로세스 실행](./chapter_1-7.md)
    - [코드: 첫 번째 시스템콜 - exec](./chapter_1-8.md)
    - [현실 세계에서](./chapter_1-9.md)
    - [연습문제 (this page)](./chapter_1-10.md)

# Exercises

1. Set a breakpoint at swtch. Single step with gdb’s `stepi` through the ret to forkret, then use gdb’s finish to proceed to trapret, then `stepi` until you get to initcode at virtual address zero.
2. `KERNBASE` limits the amount of memory a single process can use, which might be irritating on a machine with a full 4 GB of RAM. Would raising `KERNBASE` allow a process to use more memory?

1. 스위치(swtch)에 중단점을 설정하세요. GDB의 `stepi` 명령을 사용하여 ret에서 forkret로 이동하고, 그런 다음 GDB의 finish 명령을 사용하여 trapret로 이동한 후, initcode 함수로 가기 전까지 `stepi`를 계속 사용하세요.
2. `KERNBASE`는 단일 프로세스가 사용할 수 있는 메모리 양을 제한합니다. 4GB의 메모리를 가진 기계에서는 불편할 수 있습니다. `KERNBASE`를 높이면 프로세스가 더 많은 메모리를 사용할 수 있을까요?