# Exercises

1. Set a breakpoint at swtch. Single step with gdb’s `stepi` through the ret to forkret, then use gdb’s finish to proceed to trapret, then `stepi` until you get to initcode at virtual address zero.
2. `KERNBASE` limits the amount of memory a single process can use, which might be irritating on a machine with a full 4 GB of RAM. Would raising `KERNBASE` allow a process to use more memory?

1. 스위치(swtch)에 중단점을 설정하세요. GDB의 `stepi` 명령을 사용하여 ret에서 forkret로 이동하고, 그런 다음 GDB의 finish 명령을 사용하여 trapret로 이동한 후, initcode 함수로 가기 전까지 `stepi`를 계속 사용하세요.
2. `KERNBASE`는 단일 프로세스가 사용할 수 있는 메모리 양을 제한합니다. 4GB의 메모리를 가진 기계에서는 불편할 수 있습니다. `KERNBASE`를 높이면 프로세스가 더 많은 메모리를 사용할 수 있을까요?