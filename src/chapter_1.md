- [운영체제 구조 (this page)](./chapter_1.md)
  - [물리적 리소스 추상화](./chapter_1-1.md)
  - [유저모드, 커널모드 그리고 시스템콜](./chatper_1-2.md)
  - [커널 구조](./chapter_1-3.md)
  - [프로세스 개요](./chapter_1-4.md)
  - [코드: 첫 번째 주소 공간](./chapter_1-5.md)
  - [코드: 첫 번째 프로세스 생성](./chapter_1-6.md)
  - [코드: 첫 번째 프로세스 실행](./chapter_1-7.md)
  - [코드: 첫 번째 시스템콜 - exec](./chapter_1-8.md)
  - [현실 세계에서](./chapter_1-9.md)
  - [연습문제](./chapter_1-10.md)

# Operating system organization

A key requirement for an operating system is to support several activities at once. For example, using the system call interface described in chapter 0 a process can start new processes with fork. The operating system must time-share the resources of the computer among these processes. For example, even if there are more processes than there are hardware processors, the operating system must ensure that all of the processes make progress. The operating system must also arrange for isolation between the processes. That is, if one process has a bug and fails, it shouldn’t affect processes that don’t depend on the failed process. Complete isolation, however, is too strong, since it should be possible for processes to interact; pipelines are an example. Thus an operating system must fulfil three requirements: multiplexing, isolation, and interaction.

운영체제의 중요한 요구사항 중 하나는 여러 동작을 동시에 지원하는 것입니다. 예를 들어, 챕터 0에서 설명한 시스템콜 인터페이스를 사용하면 프로세스는 fork를 사용하여 새로운 프로세스를 시작할 수 있습니다. 운영체제는 컴퓨터의 자원을 이러한 프로세스들 사이에서 시분할해야 합니다. 예를 들어, 하드웨어 프로세서보다 더 많은 프로세스가 있는 경우라도, 운영체제는 모든 프로세스가 진행되도록 보장해야 합니다. 또한, 운영체제는 프로세스 간의 격리를 조율해야합니다. 즉, 한 프로세스에 버그가 있고 실패하는 경우에도 해당 프로세스에 의존하지 않는 다른 프로세스에 영향을 미치면 안됩니다. 하지만 완전한 격리는 지나치게 강력합니다. 왜냐하면 프로세스가 상호작용할 수 있어야 하기 때문입니다. 예를 들어, 파이프라인이 그런 예입니다. 따라서 운영체제는 세 가지 요구 사항을 충족해야 합니다: 멀티플렉싱, 격리 그리고 상호작용

This chapter provides an overview of how operating systems are organized to achieve these 3 requirements. It turns out there are many ways to do so, but this text focuses on mainstream designs centered around a monolithic kernel, which is used by many Unix operating systems. This chapter introduces xv6’s design by tracing the creation of the first process when xv6 starts running. In doing so, the text provides a glimpse of the implementation of all major abstractions that xv6 provides, how they interact, and how the three requirements of multiplexing, isolation, and interaction are met. Most of xv6 avoids special-casing the first process, and instead reuses code that xv6 must provide for standard operation. Subsequent chapters will explore each abstraction in more detail.

이 장에서는 운영체제가 이러한 3가지 요구 사항을 충족하기 위해 어떻게 구성되는지에 대한 개요를 제공합니다. 실제로 이를 달성하기 위한 다양한 방법이 있지만, 이 교과서는 유닉스 운영체제를 사용하는 많은 운영체제에서 사용되는 모놀리식 커널을 중심으로 한 주류 디자인에 초점을 맞추고 있습니다. 이 장에서는 xv6이 실행되기 시작할 때 첫 번째 프로세스가 생성되는 과정을 따라가며 xv6이 제공하는 모든 주요 추상화의 구현을 엿볼 수 있도록 합니다. 이를 통해 이 텍스트는 이러한 추상화가 어떻게 상호작용하며 다중화, 격리 및 상호작용이라는 세 가지 요구 사항이 어떻게 충족되는지를 보여줍니다. xv6의 대부분은 첫 번째 프로세스에 대해 특별한 경우를 회피하고 대신 표준 작동을 위해 xv6이 제공해야 하는 코드를 재사용합니다. 이어지는 장에서는 각 추상화를 더 자세히 탐구할 것입니다.

Xv6 runs on Intel 80386 or later (`x86`) processors on a PC platform, and much of its low-level functionality (for example, its process implementation) is x86-specific. This book assumes the reader has done a bit of machine-level programming on some architecture, and will introduce x86-specific ideas as they come up. Appendix A briefly outlines the PC platform.

xv6는 인텔 80386 이상 (`x86`) 프로세서가 장착된 PC 플랫폼에서 실행됩니다. 그리고 그 중 많은 저수준 기능(예: 프로세스 구현)은 x86 아키텍처에 특화되어 있습니다. 이 책은 독자가 어떤 아키텍처에서 조금의 머신 레벨 프로그래밍을 해본 경험이 있다고 가정하며, x86 특정 아이디어를 필요에 따라 소개할 것입니다. 부록 A에서는 PC 플랫폼에 대한 간략한 개요가 제공됩니다.

