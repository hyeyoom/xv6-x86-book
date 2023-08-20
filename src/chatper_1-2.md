- [운영체제 구조](./chapter_1.md)
  - [물리적 리소스 추상화](./chapter_1-1.md)
  - [유저모드, 커널모드 그리고 시스템콜 (this page)](./chatper_1-2.md)
  - [커널 구조](./chapter_1-3.md)
  - [프로세스 개요](./chapter_1-4.md)
  - [코드: 첫 번째 주소 공간](./chapter_1-5.md)
  - [코드: 첫 번째 프로세스 생성](./chapter_1-6.md)
  - [코드: 첫 번째 프로세스 실행](./chapter_1-7.md)
  - [코드: 첫 번째 시스템콜 - exec](./chapter_1-8.md)
  - [현실 세계에서](./chapter_1-9.md)
  - [연습문제](./chapter_1-10.md)

# User mode, kernel mode, and system calls

Strong isolation requires a hard boundary between applications and the operating system. If the application makes a mistake, we don’t want the operating system to fail or other applications to fail. Instead, the operating system should be able to clean up the failed application and continue running other applications. To achieve strong isolation, the operating system must arrange that applications cannot modify (or even read) the operating system’s data structures and instructions and that applications cannot access other process’s memory.

강력한 격리를 달성하려면 애플리케이션과 운영체제 사이에 강력한 경계가 필요합니다. 애플리케이션이 실수를 하더라도 운영체제가 실패하거나 다른 애플리케이션이 실패하는 것은 원치 않습니다. 대신, 운영체제는 실패한 애플리케이션을 정리하고 다른 애플리케이션을 계속 실행할 수 있어야 합니다. 강력한 격리를 달성하기 위해 운영체제는 애플리케이션이 운영체제의 데이터 구조와 명령을 수정(또는 읽기)하지 못하도록 하고, 다른 프로세스의 메모리에 접근하지 못하도록 해야 합니다.

Processors provide hardware support for strong isolation. For example, the x86 processor, like many other processors, has two modes in which the processor can execute instructions: kernel mode and user mode. In kernel mode the processor is allowed to execute privileged instructions. For example, reading and writing the disk (or any other I/O device) involves privileged instructions. If an application in user mode attempts to execute a privileged instruction, then the processor doesn’t execute the instruction, but switches to kernel mode so that the software in kernel mode can clean up the application, because it did something it shouldn’t be doing. Figure 0-1 in Chapter 0 illustrates this organization. An application can execute only user-mode instructions (e.g., adding numbers, etc.) and is said to be running in user space, while the software in kernel mode can also execute privileged instructions and is said to be running in kernel space. The software running in kernel space (or in kernel mode) is called the kernel.

프로세서는 강력한 격리를 지원하기 위한 하드웨어 지원을 제공합니다. 예를 들어, x86 프로세서와 많은 다른 프로세서들은 프로세서가 명령을 실행할 수 있는 두 가지 모드를 가지고 있습니다: 커널 모드와 사용자 모드. 커널 모드에서 프로세서는 특권 명령을 실행할 수 있습니다. 예를 들어, 디스크를 읽거나 쓰는 것(또는 다른 I/O 장치)은 특권 명령을 사용합니다. 사용자 모드의 애플리케이션이 특권 명령을 실행하려고 하면, 프로세서는 해당 명령을 실행하지 않고 커널 모드로 전환합니다. 그렇게 하여 커널 모드의 소프트웨어가 애플리케이션을 정리하도록 할 수 있습니다. 왜냐하면 애플리케이션이 해서는 안 되는 동작을 수행했기 때문입니다. Chapter 0의 Figure 0-1은 이 구조를 보여줍니다. 애플리케이션은 사용자 모드 명령(예: 숫자 더하기 등)만 실행할 수 있으며 사용자 공간에서 실행된다고 말합니다. 반면, 커널 모드 소프트웨어는 특권 명령도 실행할 수 있으며 커널 공간에서 실행된다고 말합니다. 커널 공간에서 실행되는 소프트웨어(또는 커널 모드에서 실행되는 소프트웨어)를 커널이라고 합니다.

An application that wants to read or write a file on disk must transition to the kernel to do so, because the application itself can not execute I/O instructions. Processors provide a special instruction that switches the processor from user mode to kernel mode and enters the kernel at an entry point specified by the kernel. (The x86 processor provides the int instruction for this purpose.) Once the processor has switched to kernel mode, the kernel can then validate the arguments of the system call, decide whether the application is allowed to perform the requested operation, and then deny it or execute it. It is important that the kernel sets the entry point for transitions to kernel mode; if the application could decide the kernel entry point, a malicious application could enter the kernel at a point where the validation of arguments etc. is skipped.

디스크의 파일을 읽거나 쓰려는 애플리케이션은 I/O 명령을 직접 실행할 수 없기 때문에 커널로 전환하여 수행해야 합니다. 프로세서는 사용자 모드에서 커널 모드로 전환하고 커널 내에서 지정된 진입점으로 들어가는 특수한 명령을 제공합니다. (x86 프로세서는 이 목적을 위해 int 명령을 제공합니다.) 프로세서가 커널 모드로 전환되면 커널은 시스템 콜의 인수를 검증하고 애플리케이션이 요청한 작업을 수행할 수 있는지 결정한 후 거부하거나 실행할 수 있습니다. 커널이 커널 모드로의 전환에 대한 진입점을 설정하는 것이 중요합니다. 애플리케이션이 커널 진입점을 결정할 수 있다면, 악의적인 애플리케이션이 인수의 유효성 검증 등을 건너뛰는 지점에서 커널에 진입할 수 있습니다.
