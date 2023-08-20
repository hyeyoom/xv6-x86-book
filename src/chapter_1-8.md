- [운영체제 구조](./chapter_1.md)
    - [물리적 리소스 추상화](./chapter_1-1.md)
    - [유저모드, 커널모드 그리고 시스템콜](./chatper_1-2.md)
    - [커널 구조](./chapter_1-3.md)
    - [프로세스 개요](./chapter_1-4.md)
    - [코드: 첫 번째 주소 공간](./chapter_1-5.md)
    - [코드: 첫 번째 프로세스 생성](./chapter_1-6.md)
    - [코드: 첫 번째 프로세스 실행](./chapter_1-7.md)
    - [코드: 첫 번째 시스템콜 - exec (this page)](./chapter_1-8.md)
    - [현실 세계에서](./chapter_1-9.md)
    - [연습문제](./chapter_1-10.md)

# The first system call: exec

Now that we have seen how the kernel provides strong isolation for processes, let’s look at how a user-level process re-enters the kernel to ask for services that it cannot perform itself.

이제 우리는 커널이 프로세스에 대한 강력한 격리를 제공하는 방법을 살펴보았으니, 이제 사용자 레벨 프로세스가 스스로 수행할 수 없는 서비스를 요청하기 위해 커널로 다시 진입하는 방법을 살펴보겠습니다.

The first action of `initcode.S` is to invoke the exec system call. As we saw in Chapter 0, exec replaces the memory and registers of the current process with a new program, but it leaves the file descriptors, process id, and parent process unchanged.

`initcode.S`의 첫 번째 동작은 exec 시스템 콜을 호출하는 것입니다. 우리는 0장에서 보았듯이, exec은 현재 프로세스의 메모리와 레지스터를 새 프로그램으로 교체하지만, 파일 디스크립터, 프로세스 ID 및 부모 프로세스는 변경하지 않습니다.

Initcode.S (8409) begins by pushing three values on the stack—$argv, $init, and $0—and then sets %eax to SYS_exec and executes int T_SYSCALL: it is asking the kernel to run the exec system call. If all goes well, exec never returns: it starts running the program named by $init, which is a pointer to the NUL-terminated string /init (8422-8424). 

`initcode.S` (8409)는 먼저 스택에 세 개의 값을 푸시합니다 - $argv, $init 및 $0 - 그리고 그 다음에 %eax를 SYS_exec로 설정하고 int T_SYSCALL을 실행하여 커널에 exec 시스템 콜을 실행하도록 요청합니다. 모든 것이 잘 되면, exec는 결코 반환하지 않습니다. 대신에 $init으로 지정된 프로그램을 실행합니다. 이때 $init은 /init이라는 NUL-로 종료된 문자열을 가리키는 포인터입니다 (8422-8424).

The other argument is the argv array of command-line arguments;the zero at the end of the array marks its end. If the exec fails and does return, initcode loops calling the exit system call, which definitely should not return (8416-8420). This code manually crafts the first system call to look like an ordinary system call, which we will see in Chapter 3. As before, this setup avoids special-casing the first process (in this case, its first system call), and instead reuses code that xv6 must provide for standard operation.

다른 인자는 명령행 인수의 argv 배열이며, 배열 끝에 있는 0은 그 끝을 나타냅니다. exec가 실패하고 반환하면, initcode는 exit 시스템 콜을 계속 호출하는 루프를 실행합니다. 이때 exit는 절대로 반환해서는 안됩니다 (8416-8420). 이 코드는 첫 번째 시스템 콜을 일반 시스템 콜처럼 보이도록 수동으로 조작하는 것입니다. 이러한 설정은 첫 번째 프로세스 (이 경우에는 첫 번째 시스템 콜)를 특별 케이스로 처리하지 않고, 대신 xv6가 표준 작업을 위해 제공해야 하는 코드를 재사용하는 방식을 사용합니다. (Chapter 3에서 더 자세히 다룰 예정입니다.)

Chapter 2 will cover the implementation of exec in detail, but at a high level it replaces initcode with the /init binary, loaded out of the file system. Now initcode (8400) is done, and the process will run /init instead. Init (8510) creates a new console device file if needed and then opens it as file descriptors 0, 1, and 2. Then it loops, starting a console shell, handles orphaned zombies until the shell exits, and repeats. The system is up.

2장에서는 exec의 구현을 자세히 다룰 것입니다. 그러나 높은 수준에서 보면 exec은 initcode를 파일 시스템에서 불러온 /init 바이너리로 교체합니다. 이제 initcode (8400)는 완료되었고, 프로세스는 대신 /init을 실행할 것입니다. Init (8510)은 필요한 경우 새 콘솔 디바이스 파일을 생성하고 파일 디스크립터 0, 1 및 2로 엽니다. 그런 다음 콘솔 쉘을 시작하고, 쉘이 종료될 때까지 고아 된 좀비 프로세스를 처리하고, 이를 반복합니다. 시스템이 구동됩니다.
