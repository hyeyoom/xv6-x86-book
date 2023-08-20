- [운영체제 구조](./chapter_1.md)
    - [물리적 리소스 추상화(이 페이지)](./chapter_1-1.md)
    - [유저모드, 커널모드 그리고 시스템콜](./chatper_1-2.md)
    - [커널 구조](./chapter_1-3.md)
    
# Abstracting physical resources

The first question one might ask when encountering an operating system is why have it at all? That is, one could implement the system calls in Figure 0-2 as a library, with which applications link. In this plan, each application could even have its own library tailored to its needs. Applications could directly interact with hardware resources and use those resources in the best way for the application (e.g., to achieve high or predictable performance). Some operating systems for embedded devices or real-time systems are organized in this way.

운영체제를 만나게 되면 사람들이 처음으로 묻는 질문 중 하나는 왜 운영체제가 필요한가요? 즉, Figure 0-2의 시스템 콜을 라이브러리로 구현하여 애플리케이션이 링크하는 방식으로 할 수 있습니다. 이 계획에서 각 애플리케이션은 심지어 자신의 Bed를 제작할 수 있는 라이브러리를 가질 수 있습니다. 애플리케이션은 하드웨어 리소스와 직접 상호작용하며 해당 리소스를 애플리케이션에 가장 적합한 방식으로 사용할 수 있습니다(예: 높은 또는 예측 가능한 성능을 달성하기 위해). 임베디드 장치나 실시간 시스템용 일부 운영체제는 이와 같은 방식으로 구성됩니다.

The downside of this library approach is that, if there is more than one application running, the applications must be well-behaved. For example, each application must periodically give up the processor so that other applications can run. Such a cooperative time-sharing scheme may be OK if all applications trust each other and have no bugs. It’s more typical for applications to not trust each other, and to have bugs, so one often wants stronger isolation than a cooperative scheme provides.

이 라이브러리 접근 방식의 단점은, 여러 애플리케이션이 실행 중인 경우 애플리케이션들이 잘 동작해야 한다는 점입니다. 예를 들어, 각 애플리케이션은 주기적으로 프로세서를 양보하여 다른 애플리케이션이 실행될 수 있도록 해야 합니다. 이러한 협력적인 시분할 체계는 모든 애플리케이션이 서로 신뢰하고 버그가 없다면 괜찮을 수 있습니다. 그러나 애플리케이션이 서로 신뢰하지 않거나 버그가 있을 가능성이 더 크므로, 종종 협력적인 체계보다 더 강력한 격리가 필요합니다.

To achieve strong isolation it’s helpful to forbid applications from directly accessing sensitive hardware resources, and instead to abstract the resources into services. For example, applications interact with a file system only through open, read, write, and close system calls, instead of read and writing raw disk sectors. This provides the application with the convenience of pathnames, and it allows the operating system (as the implementor of the interface) to manage the disk.

강력한 격리를 달성하기 위해서는 애플리케이션이 민감한 하드웨어 리소스에 직접 접근하는 것을 금지하고 대신 리소스를 서비스로 추상화하는 것이 유용합니다. 예를 들어, 애플리케이션은 파일 시스템과 상호작용할 때 원시 디스크 섹터를 직접 읽거나 쓰는 대신 open, read, write, close 시스템 콜을 통해 상호작용합니다. 이렇게 하면 애플리케이션은 경로명의 편의성을 활용할 수 있으며, 운영체제(인터페이스의 구현자로서)가 디스크를 관리할 수 있습니다.

Similarly, Unix transparently switches hardware processors among processes, saving and restoring register state as necessary, so that applications don’t have to be aware of time sharing. This transparency allows the operating system to share processors even if some applications are in infinite loops.

비슷하게, 유닉스는 필요한 경우 레지스터 상태를 저장하고 복원하여 프로세스 간에 하드웨어 프로세서를 투명하게 전환하여 애플리케이션이 시분할을 인식할 필요가 없도록 합니다. 이러한 투명성 덕분에 운영체제는 일부 애플리케이션이 무한 루프에 있더라도 프로세서를 공유할 수 있습니다.

As another example, Unix processes use exec to build up their memory image, instead of directly interacting with physical memory. This allows the operating system to decide where to place a process in memory; if memory is tight, the operating system might even store some of a process’s data on disk. Exec also provides users with the convenience of a file system to store executable program images.

또 다른 예로, 유닉스 프로세스는 물리적 메모리와 직접 상호작용하는 대신 메모리 이미지를 빌드업하기 위해 exec를 사용합니다. 이로써 운영체제는 프로세스를 메모리에 어디에 배치할지를 결정할 수 있습니다. 메모리가 부족한 경우, 운영체제는 프로세스의 일부 데이터를 디스크에 저장할 수도 있습니다. (역자: `page in`/`page out` 참고) Exec는 또한 사용자에게 실행 가능한 프로그램 이미지를 저장하기 위한 파일 시스템의 편의성을 제공합니다.

Many forms of interaction among Unix processes occur via file descriptors. Not only do file descriptors abstract away many details (e.g. where data in a pipe or file is stored), they also are defined in a way that simplifies interaction. For example, if one application in a pipeline fails, the kernel generates end-of-file for the next process in the pipeline.

유닉스 프로세스 간의 다양한 상호작용 형태는 파일 디스크립터를 통해 이루어집니다. 파일 디스크립터는 많은 세부 사항들(예: 파이프나 파일의 데이터가 어디에 저장되는지)을 추상화하는데 사용되며, 또한 상호작용을 간단하게 만들기 위해 정의됩니다. 예를 들어, 파이프라인 내의 하나의 애플리케이션이 실패하는 경우, 커널은 파이프라인 내의 다음 프로세스에 대해 파일의 끝을 나타내는 신호를 생성합니다.

As you can see, the system call interface in Figure 0-2 is carefully designed to provide both programmer convenience and the possibility of strong isolation. The Unix interface is not the only way to abstract resources, but it has proven to be a very good one.

보시는 대로, Figure 0-2의 시스템 콜 인터페이스는 프로그래머의 편의성과 강력한 격리의 가능성을 모두 제공하도록 신중하게 설계되었습니다. 유닉스 인터페이스는 리소스를 추상화하는 유일한 방법은 아니지만, 매우 우수한 방법으로 입증되었습니다.