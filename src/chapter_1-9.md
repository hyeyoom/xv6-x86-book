# Real world

In the real world, one can find both monolithic kernels and microkernels. Many Unix kernels are monolithic. For example, Linux has a monolithic kernel, although some OS functions run as user-level servers (e.g., the windowing system). Kernels such as L4, Minix, QNX are organized as a microkernel with servers, and have seen wide deployment in embedded settings.

실제 세계에서는 모놀리식 커널과 마이크로커널을 모두 찾아볼 수 있습니다. 많은 Unix 커널은 모놀리식입니다. 예를 들어, 리눅스는 모놀리식 커널을 가지고 있지만 일부 OS 기능은 사용자 수준 서버로 실행됩니다 (예: 윈도잉 시스템). L4, Minix, QNX와 같은 커널은 서버를 가진 마이크로커널로 구성되어 있으며 임베디드 환경에서 널리 사용됩니다.

Most operating systems have adopted the process concept, and most processes look similar to xv6’s. A real operating system would find free proc structures with an explicit free list in constant time instead of the linear-time search in allocproc; xv6 uses the linear scan (the first of many) for simplicity.

대부분의 운영 체제는 프로세스 개념을 채택하고, 대부분의 프로세스가 xv6와 유사한 구조를 가지고 있습니다. 실제 운영 체제에서는 상수 시간 내에 명시적인 프리 리스트를 사용하여 빈 프로세스 구조체를 찾을 것이며, allocproc의 선형 시간 검색과는 다른 방법을 사용할 것입니다. xv6는 간단함을 위해 선형 검색을 사용합니다 (많은 다른 검색 중 하나).
