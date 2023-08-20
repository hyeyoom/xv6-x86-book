- [운영체제 구조](./chapter_1.md)
    - [물리적 리소스 추상화](./chapter_1-1.md)
    - [유저모드, 커널모드 그리고 시스템콜](./chatper_1-2.md)
    - [커널 구조](./chapter_1-3.md)
    - [프로세스 개요](./chapter_1-4.md)
    - [코드: 첫 번째 주소 공간 (this page)](./chapter_1-5.md)
    - [코드: 첫 번째 프로세스 생성](./chapter_1-6.md)
    - [코드: 첫 번째 프로세스 실행](./chapter_1-7.md)
    - [코드: 첫 번째 시스템콜 - exec](./chapter_1-8.md)
    - [현실 세계에서](./chapter_1-9.md)
    - [연습문제](./chapter_1-10.md)

# Code: the first address space

To make the xv6 organization more concrete, we’ll look how the kernel creates the first address space (for itself), how the kernel creates and starts the first process, and how that process performs the first system call. By tracing these operations we see in detail how xv6 provides strong isolation for processes. The first step in providing strong isolation is setting up the kernel to run in its own address space.

xv6의 구조를 더 구체적으로 이해하기 위해, 커널이 첫 번째 주소 공간(자체를 위한 것)을 어떻게 생성하는지, 커널이 첫 번째 프로세스를 생성하고 시작하는지, 그리고 그 프로세스가 첫 번째 시스템 콜을 수행하는지 살펴보겠습니다. 이러한 작업을 추적함으로써 xv6가 어떻게 프로세스에 대한 강력한 격리를 제공하는지 자세히 살펴볼 수 있습니다. 강력한 격리를 제공하는 첫 번째 단계는 커널이 자체 주소 공간에서 실행되도록 설정하는 것입니다.

When a PC powers on, it initializes itself and then loads a boot loader from disk into memory and executes it. Appendix B explains the details. Xv6’s boot loader loads the xv6 kernel from disk and executes it starting at entry (1044). The x86 paging hardware is not enabled when the kernel starts; virtual addresses map directly to physical addresses.

PC가 전원을 켤 때, 자체를 초기화한 다음 디스크에서 부트 로더를 메모리로 로드하고 실행합니다. 부록 B에서 이에 대한 자세한 내용을 설명합니다. xv6의 부트 로더는 디스크에서 xv6 커널을 로드하고 (1044)에서 시작하여 실행합니다. x86 페이징 하드웨어는 커널이 시작될 때 활성화되지 않으며, 가상 주소는 직접 물리 주소로 매핑됩니다.

The boot loader loads the xv6 kernel into memory at physical address 0x100000. The reason it doesn’t load the kernel at 0x80100000, where the kernel expects to find its instructions and data, is that there may not be any physical memory at such a high address on a small machine. The reason it places the kernel at 0x100000 rather than 0x0 is because the address range 0xa0000:0x100000 contains I/O devices.

부트 로더는 xv6 커널을 물리적 주소 0x100000에 메모리로 로드합니다. 커널이 그 자신의 명령어와 데이터를 찾기를 기대하는 0x80100000에 커널을 로드하지 않는 이유는 작은 기기에서는 그러한 높은 주소에 물리적 메모리가 없을 수 있기 때문입니다. 또한 커널을 0x100000 대신 0x0에 배치하는 이유는 주소 범위 0xa0000:0x100000에는 I/O 장치가 존재하기 때문입니다.

To allow the rest of the kernel to run, entry sets up a page table that maps virtual addresses starting at 0x80000000 (called `KERNBASE` (0207)) to physical addresses starting at 0x0 (see [`Figure 1-2`](./assets/fig-1-2.png)). Setting up two ranges of virtual addresses that map to the same physical memory range is a common use of page tables, and we will see more examples like this one.

커널의 나머지 부분이 실행될 수 있도록, entry 함수는 가상 주소 0x80000000(인 `KERNBASE`(0207)라고 부름)부터 시작하여 물리 주소 0x0부터 시작하는 페이지 테이블을 설정합니다([`Figure 1-2`](./assets/fig-1-2.png) 참조). 동일한 물리 메모리 범위에 대한 가상 주소의 두 범위를 설정하는 것은 페이지 테이블의 일반적인 사용 사례이며, 이와 유사한 예제를 더 많이 볼 것입니다.

The entry page table is defined in main.c (1306). We look at the details of page tables in Chapter 2, but the short story is that entry 0 maps virtual addresses 0:0x400000 to physical addresses 0:0x400000. This mapping is required as long as entry is executing at low addresses, but will eventually be removed.

entry 페이지 테이블은 main.c (1306)에서 정의됩니다. 페이지 테이블의 자세한 내용은 2장에서 살펴보겠지만, 간단하게 설명하면 entry 0은 가상 주소 0:0x400000를 물리 주소 0:0x400000에 매핑합니다. 이 매핑은 entry가 낮은 주소에서 실행 중인 동안 필요하지만, 최종적으로 제거될 것입니다.

Entry 512 maps virtual addresses `KERNBASE`:`KERNBASE`+0x400000 to physical addresses 0:0x400000. This entry will be used by the kernel after entry has finished; it maps the high virtual addresses at which the kernel expects to find its instructions and data to the low physical addresses where the boot loader loaded them. This mapping restricts the kernel instructions and data to 4 Mbytes.

Entry 512은 가상 주소 `KERNBASE`:`KERNBASE`+0x400000을 물리 주소 0:0x400000에 매핑합니다. 이 엔트리는 entry가 완료된 후에 커널에 의해 사용될 것이며, 커널이 명령어와 데이터를 찾기를 기대하는 높은 가상 주소를 부트 로더가 로드한 낮은 물리 주소로 매핑합니다. 이 매핑은 커널의 명령어와 데이터를 4 메가바이트로 제한합니다.

Returning to entry, it loads the physical address of entrypgdir into control register %cr3. The value in %cr3 must be a physical address. It wouldn’t make sense for %cr3 to hold the virtual address of entrypgdir, because the paging hardware doesn’t know how to translate virtual addresses yet; it doesn’t have a page table yet. The symbol entrypgdir refers to an address in high memory, and the macro V2P_WO (0213) subtracts `KERNBASE` in order to find the physical address. To enable the paging hardware, xv6 sets the flag CR0_PG in the control register %cr0.

entry로 돌아가서, 이 함수는 entrypgdir의 물리적 주소를 제어 레지스터 %cr3에 로드합니다. %cr3의 값은 물리적 주소여야 합니다. %cr3에 entrypgdir의 가상 주소를 저장하는 것은 의미가 없습니다. 왜냐하면 페이징 하드웨어는 아직 가상 주소를 변환할 수 없기 때문에 페이지 테이블이 아직 없습니다. entrypgdir 심볼은 높은 메모리 주소를 가리키며, V2P_WO 매크로(0213)는 물리적 주소를 찾기 위해 `KERNBASE`를 빼는 역할을 합니다. 페이징 하드웨어를 활성화하기 위해 xv6은 제어 레지스터 %cr0에서 플래그 CR0_PG를 설정합니다.

The processor is still executing instructions at low addresses after paging is enabled, which works since entrypgdir maps low addresses. If xv6 had omitted entry 0 from entrypgdir, the computer would have crashed when trying to execute the instruction after the one that enabled paging.

페이징이 활성화된 후에도 프로세서는 여전히 낮은 주소에서 명령어를 실행하고 있습니다. 이는 entrypgdir이 낮은 주소를 매핑하기 때문에 작동합니다. 만약 xv6가 entrypgdir에서 entry 0을 제외했다면, 페이징을 활성화하는 명령어 뒤에 있는 명령어를 실행하려고 할 때 컴퓨터가 충돌했을 것입니다.

Now entry needs to transfer to the kernel’s C code, and run it in high memory. First it makes the stack pointer, %esp, point to memory to be used as a stack (1058). All symbols have high addresses, including stack, so the stack will still be valid even when the low mappings are removed. Finally entry jumps to main, which is also a high address. The indirect jump is needed because the assembler would otherwise generate a PC-relative direct jump, which would execute the low-memory version of main. Main cannot return, since the there’s no return PC on the stack. Now the kernel is running in high addresses in the function main (1217).

이제 entry는 커널의 C 코드로 전환하여 높은 메모리에서 실행해야 합니다. 먼저 스택 포인터인 %esp를 스택으로 사용될 메모리를 가리키도록 설정합니다(1058). 모든 심볼은 높은 주소를 갖기 때문에, 낮은 매핑이 제거되더라도 스택은 여전히 유효합니다. 마지막으로 entry는 main으로 점프합니다. main 역시 높은 주소입니다. 이러한 간접 점프가 필요한 이유는 어셈블러가 그렇지 않으면 PC 상대적 직접 점프를 생성하게 되어 낮은 메모리 버전의 main이 실행됩니다. main은 반환할 수 없습니다. 왜냐하면 스택에 반환 PC가 없기 때문입니다. 이제 커널은 main 함수(1217) 내에서 높은 주소에서 실행되고 있습니다.
