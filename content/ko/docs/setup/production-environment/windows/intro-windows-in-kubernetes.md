---
reviewers:
title: 쿠버네티스에서 윈도우 지원 소개
content_template: templates/concept
weight: 65
---

{{% capture overview %}}

윈도우는 많은 조직에서 실행하는 서비스와 애플리케이션의 상당 부분을 구성한다. [윈도우 컨테이너](https://aka.ms/windowscontainers)는 현대화된 방식으로 프로세스와 패키지의 종속성을 감싸고 DevOps 사례 이용을 쉽게 하며, 윈도우 애플리케이션을 위한 클라우드 네이티브 패턴을 따른다. 쿠버네티스는 산업 표준 컨테이너 오케스트레이터가 되었고, 쿠버네티스 1.14 릴리스에는 쿠버네티스 클러스터의 윈도우 노드 상에서 윈도우 컨테이너를 스케줄링을 위한 실서비스 지원이 포함되어, 윈도우 애플리케이션의 광범위한 에코시스템으로 쿠버네티스의 강력한 기능을 활용할 수 있다. 윈도우 기반 애플리케이션과 리눅스 기반 애플리케이션을 모두에 투자한 조직은 워크로드를 관리하기 위해 분리된 오케스트레이터를 찾을 필요가 없어, 운영체제에 관계없이 배포 전반에 걸쳐 운영 효율성이 향상된다.

{{% /capture %}}

{{% capture body %}}

## 쿠버네티스에서 윈도우 컨테이너

쿠버네티스에서 윈도우 컨테이너의 오케스트레이션을 활성화하려면, 기존 리눅스 클러스터에 간단히 윈도우 노드를 추가만 하면 된다. 쿠버네티스의 [파드](/docs/concepts/workloads/pods/pod-overview/)에서 윈도우 컨테이너를 스케줄링하는 것은 리눅스 기반의 컨테이너를 스케줄링하는 것만큼 간단하고 쉽다.

윈도우 컨테이너를 실행하려면 사용자의 쿠버네티스 클러스터에 리눅스에서 실행하는 컨트롤 플레인 노드와 워크로드의 필요에 따라 윈도우나 리눅스를 실행하는 워커 노드를 가진 여러 운영체제를 반드시 포함해야 한다. 윈도우 서버 2019는 윈도우 상에서(kubelet, [컨테이너 런타임](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/containerd), kube-proxy 포함) [쿠버네티스 노드](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/architecture.md#the-kubernetes-node)를 활성화할 수 있는 유일한 윈도우 운영체제이다. 윈도우 배포 채널에 대한 자세한 설명은 [Microsoft 문서](https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels-19)를 살펴보자.

{{< note >}}
[마스터 구성요소](/docs/concepts/overview/components/)를 포함한 쿠버네티스 컨트롤 플레인은 계속 리눅스에서 실행한다. 윈도우로만 구성되는 쿠버네티스 클러스터에 대한 계획은 없다.
{{< /note >}}

{{< note >}}
이 문서에서 윈도우 컨테이너에 대한 이야기는 프로세스가 격리된(isolation) 윈도우 컨테이너를 의미한다. [Hyper-V 격리 기능](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container)이 있는 윈도우 컨테이너는 향후 릴리스에 계획되어 있다.
{{< /note >}}

## 지원되는 기능과 한계

### 저원되는 기능성

#### 컴퓨팅

API와 Kubectl 관점에서 윈도우 컨테이너는 리눅스 기반의 컨테이너와 거의 같은 방식으로 동작한다. 그러나 한계 단원에는 요약된 주요 기능에는 몇 가지 눈에 띄는 차이점이 있다.

운영체제의 버전부터 시작하자. 쿠버네티스에서 윈도우 운영 체제 지원에 대해서는 다음 표를 참고하자. 단일 이기종 쿠버네티스 클러스터는 윈도우와 리눅스 워커 노드를 모두 갖을 수 있다. 윈도우 컨테이너는 윈도우 노드에서 리눅스 컨테이너는 리눅스 노드에서 스케줄되어야 한다.

| 쿠버네티스 버전 | 호스트 OS 버전 (쿠버네티스 노드) | | |
| --- | --- | --- | --- |
| | *Windows Server 1709* | *Windows Server 1803* | *Windows Server 1809/Windows Server 2019* |
| *Kubernetes v1.14* | 지원하지 않음 | 지원하지 않음 | 윈도우 서버 컨테이너 빌드 177763.* 와 Docker EE-basic 18.09 포함 |

{{< note >}}
모든 윈도우 고객이 애플리케이션의 운영체제를 자주 업데이트하지는 않을 것이다. 사용자 애플리케이션을 업그레이드하려면 클러스터에 새로운 노드를 업그레이드하거나 도입해야 한다. 쿠버네티스 상에 운영되는 컨테이너의 운영체제를 업그레이드하도록 선택된 고객을 위해 새로운 운영체제 버전 지원을 추가할 때에 단계별 설명과 가이드를 제공할 것이다. 이 가이드는 클러스터 노드와 함께 사용자 애플리케이션을 업그레이드하기 위한 추천하는 업그레이드 절차를 포함할 것이다. 윈도우 노드는 리눅스 노드가 오늘까지 해온 같은 방식인 쿠버네티스 [버전 차이(skew) 정책](/docs/setup/release/version-skew-policy/)(노드에서 컨트롤 플레인 버전 부여 방식)를 준수한다.
We don't expect all Windows customers to update the operating system for their apps frequently. Upgrading your applications is what dictates and necessitates upgrading or introducing new nodes to the cluster. For the customers that chose to upgrade their operating system for containers running on Kubernetes, we will offer guidance and step-by-step instructions when we add support for a new operating system version. This guidance will include recommended upgrade procedures for upgrading user applications together with cluster nodes. Windows nodes adhere to Kubernetes [version-skew policy](/docs/setup/release/version-skew-policy/) (node to control plane versioning) the same way as Linux nodes do today.
{{< /note >}}
{{< note >}}

윈도우 서버 호스트 운영체제는 [윈도우 서버](https://www.microsoft.com/en-us/cloud-platform/windows-server-pricing) 라이선스로 종속된다.
윈도우 컨테이너 이미지는 [Windows 컨테이너에 대한 추가 라이센스 조항](https://docs.microsoft.com/en-us/virtualization/windowscontainers/images-eula)에 종속된다.
The Windows Server Host Operating System is subject to the [Windows Server ](https://www.microsoft.com/en-us/cloud-platform/windows-server-pricing) licensing. The Windows Container images are subject to the [Supplemental License Terms for Windows containers](https://docs.microsoft.com/en-us/virtualization/windowscontainers/images-eula).
{{< /note >}}
{{< note >}}
프로세스 격리화된 윈도우 컨테이너에는 [호스트 OS 버전이 컨테이너 기반 이미지의 OS 버전과 반드시 일치해야 한다](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility)는 엄격한 호환성 규칙이 있다. 쿠버네티스에서 Hyper-V를 이용하는 윈도우 컨테이너를 지원하면, 제한과 호환성 규칙은 바뀔 것이다.
Windows containers with process isolation have strict compatibility rules, [where the host OS version must match the container base image OS version](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility). Once we support Windows containers with Hyper-V isolation in Kubernetes, the limitation and compatibility rules will change.
{{< /note >}}

주요 쿠버네티스 요소는 리눅스에서 되는 것과 같은 방식으로 동작한다. 이번 단락은 몇가지 주요 워크로드를 가능케 하는 것과 윈도우에 어떻게 대치되었는지 이야기한다.
Key Kubernetes elements work the same way in Windows as they do in Linux. In this section, we talk about some of the key workload enablers and how they map to Windows.

* [파드](/docs/concepts/workloads/pods/pod-overview/)
* [Pods](/docs/concepts/workloads/pods/pod-overview/)

    파드는 사용자가 생성하고 배포할 수 있는 쿠버네티스 오브젝트 모델에서 가장 작고 단순한 단위로 기본 빌딩 블럭이다. 다음 파드 기능, 속성, 이벤트를 윈도우 컨테이너에서 지원한다.
    A Pod is the basic building block of Kubernetes–the smallest and simplest unit in the Kubernetes object model that you create or deploy. The following Pod capabilities, properties and events are supported with Windows containers:

  * 프로세스 격리와 볼륨 공유를 제공하는 파드당 단일 혹은 다중 컨테이너
  * 파드의 상태 필드
  * 준비 및 생존 신호
  * postStart와 preStop 컨테이너 수명주기 이벤트
  * 볼륨이나 환경변수로 컨피그맵, 시크릿
  * EmptyDir
  * 네임드 파이프 호스트 마운트
  * 리소스 상한
* [컨트롤러](/docs/concepts/workloads/controllers/)
  * Single or multiple containers per Pod with process isolation and volume sharing
  * Pod status fields
  * Readiness and Liveness probes
  * postStart & preStop container lifecycle events
  * ConfigMap, Secrets: as environment variables or volumes
  * EmptyDir
  * Named pipe host mounts
  * Resource limits
* [Controllers](/docs/concepts/workloads/controllers/)

    쿠버네티스 컨트롤러는 파드의 원하는 상태를 다룬다. 다음 워크로드 컨트롤러를 윈도우 컨테이너에서 지원한다.
    Kubernetes controllers handle the desired state of Pods. The following workload controllers are supported with Windows containers:

  * 리플리카셋
  * 리플리카컨트롤러
  * 배포
  * 스테이트풀셋
  * 데몬셋
  * 잡
  * 크론잡
* [서비스](/docs/concepts/services-networking/service/)
  * ReplicaSet
  * ReplicationController
  * Deployments
  * StatefulSets
  * DaemonSet
  * Job
  * CronJob
* [Services](/docs/concepts/services-networking/service/)

    쿠버네티스 서비스는 파드의 논리적인 집합과 그것을 접근하는 정책을 정의하는 추상화이다. 때때로 마이크로 서비스라 부른다. 운영체제 접속성을 넘어 서비스를 이용할 수 있다. 윈도우에서 서비스는 다음 종류와 속성과 능력을 활용한다.
    A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service. You can use services for cross-operating system connectivity. In Windows, services can utilize the following types, properties and 능력:

  * 서비스 환경 변수
  * NodePort
  * ClusterIP
  * LoadBalancer
  * ExternalName
  * Headless services
  * Service Environment variables
  * NodePort
  * ClusterIP
  * LoadBalancer
  * ExternalName
  * Headless services

파드, 컨트롤러, 서비스는 쿠버네티스의 윈도우 워크로드를 관리하는 중요한 요소이다. 그러나 그 자체로는 동적 클라우드 네이티브 환경에서 윈도우 워크로드의 적절한 수명주기를 관리하기에는 충분하지 않다. 다음 기능에 대한 지원을 추가했다.

* 파드와 컨테이너 메트릭스
* Horizontal Pod Autoscaler 지원
* kubectl Exec
* 리소스 할당양
* 스케줄러 선점
* Pod and container metrics
* Horizontal Pod Autoscaler support
* kubectl Exec
* Resource Quotas
* Scheduler preemption

#### 컨테이너 런타임

Docker EE-basic 18.09는 윈도우 서버 2019 / 1809 노드의 쿠버네티스에 필요하다. 이는 Kubelet에 포함된 dockersim 코드와 함께 동작한다. CRI-ContainerD 같은 추가 런타임은 쿠버네티스 나중 버전에서 지원될 수 있다.

#### 스토리지

쿠버네티스 볼륨을 사용하면 데이터 지속성과 볼륨 공유 요구사항이 있는 복잡한 애플리케이션을 쿠버네티스에 배포할 수 있다. 윈도우의 쿠버네티스는 다음과 같은 유형의 [볼륨](/docs/concepts/storage/volumes/)을 지원한다.

* [SMB and iSCSI](https://github.com/Microsoft/K8s-Storage-Plugins/tree/master/flexvolume/windows)를 지원하는 FlexVolume 아웃오브트리(out-of-tree) 플러그인
* [azureDisk](/docs/concepts/storage/volumes/#azuredisk)
* [azureFile](/docs/concepts/storage/volumes/#azurefile)
* [gcePersistentDisk](/docs/concepts/storage/volumes/#gcepersistentdisk)

#### 네트워킹

윈도우 컨테이너의 네트워킹은 [CNI 플러그인](/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)을 통해 노출된다. 윈도우 컨테이너는 네트워킹에 관련하여 가상머신과 유사하게 기능 동작을 한다. 각 컨테이너는 Hyper-V 가상 스위치(vSwitch)에 연결된 가상 네트워크 아답터(vNIC)가 있다. HNS(Host Networking Service)와 HCS(Host Compute Service)는 함께 동작하여 컨테어너를 만들고 컨테이너 vNIC를 네트워크에 연결한다. HCS는 컨테이너 관리를 담당하고 HNS는 다음과 같은 네트워킹 리소스 관리를 담당한다.

* 가상 네트워크(vSwitch 생성 포함)
* 엔드 포인트 / vNICs
* 네임스페이스
* 정책 (패킷 캡슐화, 로드 밸런싱 규칙, ACLs, NAT 규칙 등)

다음 서비스 사양을 지원한다.

* NodePort
* ClusterIP
* LoadBalancer
* ExternalName

윈도우는 L2bridge, L2tunnel, Overlay, Transparent 와 NAT의 다섯가지 네트워킹 드라이버/모드를 지원한다. 윈도와 리눅스 워커 노드를 가진 이기종 클러스터에서 윈도우와 리눅스 모두에 호환성있는 네트워킹 솔루션을 선택해야 한다. 다음의 아웃오브트리 외부 플러그인은 윈도우를 지원하며, 각 CNI를 사용하는 경우에 대한 권장사항이 있다.

| 네트워크 드라이버 | 설명 | 컨테이너 패킷 수정 | 네트워크 플러그인 | 네트워크 플러그인 특징 |
| -------------- | ----------- | ------------------------------ | --------------- | ------------------------------ |
| L2bridge       | Containers are attached to an external vSwitch. Containers are attached to the underlay network, although the physical network doesn't need to learn the container MACs because they are rewritten on ingress/egress. Inter-container traffic is bridged inside the container host. | MAC is rewritten to host MAC, IP remains the same. | [win-bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/windows/win-bridge), [Azure-CNI](https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md), Flannel host-gateway uses win-bridge | win-bridge uses L2bridge network mode, connects containers to the underlay of hosts, offering best performance. Requires L2 adjacency between container hosts |
| L2Tunnel | This is a special case of l2bridge, but only used on Azure. All packets are sent to the virtualization host where SDN policy is applied. | MAC rewritten, IP visible on the underlay network | [Azure-CNI](https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md) | Azure-CNI allows integration of containers with Azure vNET, and allows them to leverage the set of capabilities that [Azure Virtual Network provides](https://azure.microsoft.com/en-us/services/virtual-network/). For example, securely connect to Azure services or use Azure NSGs. See [azure-cni for some examples](https://docs.microsoft.com/en-us/azure/aks/concepts-network#azure-cni-advanced-networking) |
| Overlay (Overlay networking for Windows in Kubernetes is in *alpha* stage) | Containers are given a vNIC connected to an external vSwitch. Each overlay network gets its own IP subnet, defined by a custom IP prefix.The overlay network driver uses VXLAN encapsulation. | Encapsulated with an outer header, inner packet remains the same. | [Win-overlay](https://github.com/containernetworking/plugins/tree/master/plugins/main/windows/win-overlay), Flannel VXLAN (uses win-overlay) | win-overlay should be used when virtual container networks are desired to be isolated from underlay of hosts (e.g. for security reasons). Allows for IPs to be re-used for different overlay networks (which have different VNID tags)  if you are restricted on IPs in your datacenter. This option may be used when the container hosts are not L2 adjacent but have L3 connectivity |
| Transparent (special use case for [ovn-kubernetes](https://github.com/openvswitch/ovn-kubernetes)) | Requires an external vSwitch. Containers are attached to an external vSwitch which enables intra-pod communication via logical networks (logical switches and routers). | Packet is encapsulated either via [GENEVE](https://datatracker.ietf.org/doc/draft-gross-geneve/) or [STT](https://datatracker.ietf.org/doc/draft-davie-stt/) tunneling to reach pods which are not on the same host.  <br/> Packets are forwarded or dropped via the tunnel metadata information supplied by the ovn network controller. <br/> NAT is done for north-south communication. | [ovn-kubernetes](https://github.com/openvswitch/ovn-kubernetes) | [Deploy via ansible](https://github.com/openvswitch/ovn-kubernetes/tree/master/contrib). Distributed ACLs can be applied via Kubernetes policies. IPAM support. Load-balancing can be achieved without kube-proxy. NATing is done without using iptables/netsh. |
| NAT (*not used in Kubernetes*) | Containers are given a vNIC connected to an internal vSwitch. DNS/DHCP is provided using an internal component called [WinNAT](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/) | MAC and IP is rewritten to host MAC/IP. | [nat](https://github.com/Microsoft/windows-container-networking/tree/master/plugins/nat) | Included here for completeness |

As outlined above, the [Flannel](https://github.com/coreos/flannel) CNI [meta plugin](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel) is also supported on [Windows](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#windows-support-experimental) via the [VXLAN network backend](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) (**alpha support** ; delegates to win-overlay) and [host-gateway network backend](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#host-gw) (stable support; delegates to win-bridge). This plugin supports delegating to one of the reference CNI plugins (win-overlay, win-bridge), to work in conjunction with Flannel daemon on Windows (Flanneld) for automatic node subnet lease assignment and HNS network creation. This plugin reads in its own configuration file (net-conf.json), and aggregates it with the environment variables from the FlannelD generated subnet.env file. It then delegates to one of the reference CNI plugins for network plumbing, and sends the correct configuration containing the node-assigned subnet to the IPAM plugin (e.g. host-local).

노드, 파드와 서비스 오브젝트에서는 다음 네트워크 흐름이 TCP/UDP 트래픽에 대해 지원된다.

* 파드 -> 파드 (IP)
* 파드 -> 파드 (이름)
* 파드 -> 서비스 (클러스터 IP)
* 파드 -> 서비스 (PQDN, "." 이 없는 경우에만)
* 파드 -> 서비스 (FQDN)
* 파드 -> 외부 (IP)
* 파드 -> 외부 (DNS)
* 노드 -> 파드
* 파드 -> 노드

윈도우에서 다음 IPAM 선택사항이 지원된다.

* [호스트-로컬](https://github.com/containernetworking/plugins/tree/master/plugins/ipam/host-local)
* HNS IPAM (Inbox platform IPAM, IPAM이 설정되지 않았다면 폴백(fallback))
* [Azure-vnet-ipam](https://github.com/Azure/azure-container-networking/blob/master/docs/ipam.md) (azure-cni 만 해당)

### 한계
### Limitations

#### 컨트롤 플레인
#### Control Plane

Windows is only supported as a worker node in the Kubernetes architecture and component matrix. This means that a Kubernetes cluster must always include Linux master nodes, zero or more Linux worker nodes, and zero or more Windows worker nodes.

#### 컴퓨트
#### Compute

##### 리소스 관리와 프로세스 격리
##### Resource management and process isolation

 Linux cgroups are used as a pod boundary for resource controls in Linux. Containers are created within that boundary for network, process and file system isolation. The cgroups APIs can be used to gather cpu/io/memory stats. In contrast, Windows uses a Job object per container with a system namespace filter to contain all processes in a container and provide logical isolation from the host. There is no way to run a Windows container without the namespace filtering in place. This means that system privileges cannot be asserted in the context of the host, and thus privileged containers are not available on Windows. Containers cannot assume an identity from the host because the Security Account Manager (SAM) is separate.

##### 운영체제 제한
##### Operating System Restrictions

Windows has strict compatibility rules, where the host OS version must match the container base image OS version. Only Windows containers with a container operating system of Windows Server 2019 are supported. Hyper-V isolation of containers, enabling some backward compatibility of Windows container image versions, is planned for a future release.

##### 기능 제한
##### Feature Restrictions

* TerminationGracePeriod: not implemented
* Single file mapping: to be implemented with CRI-ContainerD
* Termination message: to be implemented with CRI-ContainerD
* Privileged Containers: not currently supported in Windows containers
* HugePages: not currently supported in Windows containers
* The existing node problem detector is Linux-only and requires privileged containers. In general, we don't expect this to be used on Windows because privileged containers are not supported
* Not all features of shared namespaces are supported (see API section for more details)

##### 메모리 예약과 처리
##### Memory Reservations and Handling

Windows does not have an out-of-memory process killer as Linux does. Windows always treats all user-mode memory allocations as virtual, and pagefiles are mandatory. The net effect is that Windows won't reach out of memory conditions the same way Linux does, and processes page to disk instead of being subject to out of memory (OOM) termination. If memory is over-provisioned and all physical memory is exhausted, then paging can slow down performance.

Keeping memory usage within reasonable bounds is possible with a two-step process. First, use the kubelet parameters `--kubelet-reserve` and/or `--system-reserve` to account for memory usage on the node (outside of containers). This reduces [NodeAllocatable](/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable)). As you deploy workloads, use resource limits (must set only limits or limits must equal requests) on containers. This also subtracts from NodeAllocatable and prevents the scheduler from adding more pods once a node is full.

A best practice to avoid over-provisioning is to configure the kubelet with a system reserved memory of at least 2GB to account for Windows, Docker, and Kubernetes processes.

The behavior of the flags behave differently as described below:

* `--kubelet-reserve`, `--system-reserve` , and `--eviction-hard` flags update Node Allocatable
* Eviction by using `--enforce-node-allocable` is not implemented
* Eviction by using `--eviction-hard` and `--eviction-soft` are not implemented
* MemoryPressure Condition is not implemented
* There are no OOM eviction actions taken by the kubelet
* Kubelet running on the windows node does not have memory restrictions. `--kubelet-reserve` and `--system-reserve` do not set limits on kubelet or processes running on the host. This means kubelet or a process on the host could cause memory resource starvation outside the node-allocatable and scheduler

#### 스토리지
#### Storage

Windows has a layered filesystem driver to mount container layers and create a copy filesystem based on NTFS. All file paths in the container are resolved only within the context of that container.

* Volume mounts can only target a directory in the container, and not an individual file
* Volume mounts cannot project files or directories back to the host filesystem
* Read-only filesystems are not supported because write access is always required for the Windows registry and SAM database. However, read-only volumes are supported
* Volume user-masks and permissions are not available. Because the SAM is not shared between the host & container, there's no mapping between them. All permissions are resolved within the context of the container

As a result, the following storage functionality is not supported on Windows nodes

* Volume subpath mounts. Only the entire volume can be mounted in a Windows container.
* Subpath volume mounting for Secrets
* Host mount projection
* DefaultMode (due to UID/GID dependency)
* Read-only root filesystem. Mapped volumes still support readOnly
* Block device mapping
* Memory as the storage medium
* CSI plugins which require privileged containers
* File system features like uui/guid, per-user Linux filesystem permissions
* NFS based storage/volume support
* Expanding the mounted volume (resizefs)

#### 네트워킹
#### Networking

Windows Container Networking differs in some important ways from Linux networking. The [Microsoft documentation for Windows Container Networking](https://docs.microsoft.com/en-us/virtualization/windowscontainers/container-networking/architecture) contains additional details and background.

The Windows host networking networking service and virtual switch implement namespacing and can create virtual NICs as needed for a pod or container. However, many configurations such as DNS, routes, and metrics are stored in the Windows registry database rather than /etc/... files as they are on Linux. The Windows registry for the container is separate from that of the host, so concepts like mapping /etc/resolv.conf from the host into a container don't have the same effect they would on Linux. These must be configured using Windows APIs run in the context of that container. Therefore CNI implementations need to call the HNS instead of relying on file mappings to pass network details into the pod or container.

The following networking functionality is not supported on Windows nodes

* Host networking mode is not available for Windows pods
* Local NodePort access from the node itself fails (works for other nodes or external clients)
* Accessing service VIPs from nodes will be available with a future release of Windows Server
* Overlay networking support in kube-proxy is an alpha release. In addition, it requires [KB4482887](https://support.microsoft.com/en-us/help/4482887/windows-10-update-kb4482887) to be installed on Windows Server 2019
* Local Traffic Policy and DSR mode
* Windows containers connected to l2bridge, l2tunnel, or overlay networks do not support communicating over the IPv6 stack. There is outstanding Windows platform work required to enable these network drivers to consume IPv6 addresses and subsequent Kubernetes work in kubelet, kube-proxy, and CNI plugins.
* Outbound communication using the ICMP protocol via the win-overlay, win-bridge, and Azure-CNI plugin. Specifically, the Windows data plane ([VFP](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/)) doesn't support ICMP packet transpositions. This means:
  * ICMP packets directed to destinations within the same network (e.g. pod to pod communication via ping) work as expected and without any limitations
  * TCP/UDP packets work as expected and without any limitations
  * ICMP packets directed to pass through a remote network (e.g. pod to external internet communication via ping) cannot be transposed and thus will not be routed back to their source
  * Since TCP/UDP packets can still be transposed, one can substitute `ping <destination>` with `curl <destination>` to be able to debug connectivity to the outside world.

These features were added in Kubernetes v1.15:

* `kubectl port-forward`

##### CNI 플러그인

* Windows reference network plugins win-bridge and win-overlay do not currently implement [CNI spec](https://github.com/containernetworking/cni/blob/master/SPEC.md) v0.4.0 due to missing "CHECK" implementation.
* The Flannel VXLAN CNI has the following limitations on Windows:

1. Node-pod connectivity isn't possible by design. It's only possible for local pods with Flannel [PR 1096](https://github.com/coreos/flannel/pull/1096)
2. We are restricted to using VNI 4096 and UDP port 4789. The VNI limitation is being worked on and will be overcome in a future release (open-source flannel changes). See the official [Flannel VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) backend docs for more details on these parameters.

##### DNS {#dns-limitations}

* ClusterFirstWithHostNet는 DNS에서 지원되지 않는다. 윈도우는 '.' 을 FQDN 으로 간주하고 PQDN로 풀어냄은(resolving) 건너 뛴다.
* Linux에는 PQDN을 확인할 때 사용되는 DNS 접미사 목록이 있다. 윈도우에는 해당 포드의 네임 스페이스(예, mydns.svc.cluster.local)와 관련된 DNS 접미사인 1 개의 DNS 접미사만 있다. 윈도우는 FQDN과 서비스 또는 이름을 해당 접미사만으로 확인할 수 있다. 예를 들어 기본 네임 스페이스에 생성된 파드에는 DNS 접미어 **default.svc.cluster.local** 이 있다. 윈도우 파드에서는 **kubernetes.default.svc.cluster.local** 과 **kubernetes** 를 모두 풀어낼 수 있지만 **kubernetes.default** 또는 **kubernetes.default.svc** 와 같은 중간에 있는 것은 아닙니다 .

##### 보안

Secrets are written in clear text on the node's volume (as compared to tmpfs/in-memory on linux). This means customers have to do two things

1. Use file ACLs to secure the secrets file location
2. Use volume-level encryption using [BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-how-to-deploy-on-windows-server)

[RunAsUser](/docs/concepts/policy/pod-security-policy/#users-and-groups)is not currently supported on Windows. The workaround is to create local accounts before packaging the container. The RunAsUsername capability may be added in a future release.

Linux specific pod security context privileges such as SELinux, AppArmor, Seccomp, Capabilities (POSIX Capabilities), and others are not supported.

In addition, as mentioned already, privileged containers are not supported on Windows.

#### API

There are no differences in how most of the Kubernetes APIs work for Windows. The subtleties around what's different come down to differences in the OS and container runtime. In certain situations, some properties on workload APIs such as Pod or Container were designed with an assumption that they are implemented on Linux, failing to run on Windows.

At a high level, these OS concepts are different:

* Identity - Linux uses userID (UID) and groupID (GID) which are represented as integer types. User and group names are not canonical - they are just an alias in `/etc/groups` or `/etc/passwd` back to UID+GID. Windows uses a larger binary security identifier (SID) which is stored in the Windows Security Access Manager (SAM) database. This database is not shared between the host and containers, or between containers.
* File permissions - Windows uses an access control list based on SIDs, rather than a bitmask of permissions and UID+GID
* File paths - convention on Windows is to use `\` instead of `/`. The Go IO libraries typically accept both and just make it work, but when you're setting a path or command line that's interpreted inside a container, `\` may be needed.
* Signals - Windows interactive apps handle termination differently, and can implement one or more of these:
  * A UI thread handles well-defined messages including WM_CLOSE
  * Console apps handle ctrl-c or ctrl-break using a Control Handler
  * Services register a Service Control Handler function that can accept SERVICE_CONTROL_STOP control codes

Exit Codes follow the same convention where 0 is success, nonzero is failure. The specific error codes may differ across Windows and Linux. However, exit codes passed from the Kubernetes components (kubelet, kube-proxy) are unchanged.

##### V1.Container

* V1.Container.ResourceRequirements.limits.cpu and V1.Container.ResourceRequirements.limits.memory - Windows doesn't use hard limits for CPU allocations. Instead, a share system is used. The existing fields based on millicores are scaled into relative shares that are followed by the Windows scheduler. [see: kuberuntime/helpers_windows.go](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/kuberuntime/helpers_windows.go), [see: resource controls in Microsoft docs](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls)
  * Huge pages are not implemented in the Windows container runtime, and are not available. They require [asserting a user privilege](https://docs.microsoft.com/en-us/windows/desktop/Memory/large-page-support) that's not configurable for containers.
* V1.Container.ResourceRequirements.requests.cpu and V1.Container.ResourceRequirements.requests.memory - Requests are subtracted from node available resources, so they can be used to avoid overprovisioning a node. However, they cannot be used to guarantee resources in an overprovisioned node. They should be applied to all containers as a best practice if the operator wants to avoid overprovisioning entirely.
* V1.Container.SecurityContext.allowPrivilegeEscalation - not possible on Windows, none of the capabilities are hooked up
* V1.Container.SecurityContext.Capabilities - POSIX capabilities are not implemented on Windows
* V1.Container.SecurityContext.privileged - Windows doesn't support privileged containers
* V1.Container.SecurityContext.procMount - Windows doesn't have a /proc filesystem
* V1.Container.SecurityContext.readOnlyRootFilesystem - not possible on Windows, write access is required for registry & system processes to run inside the container
* V1.Container.SecurityContext.runAsGroup - not possible on Windows, no GID support
* V1.Container.SecurityContext.runAsNonRoot - Windows does not have a root user. The closest equivalent is ContainerAdministrator which is an identity that doesn't exist on the node.
* V1.Container.SecurityContext.runAsUser - not possible on Windows, no UID support as int.
* V1.Container.SecurityContext.seLinuxOptions - not possible on Windows, no SELinux
* V1.Container.terminationMessagePath - this has some limitations in that Windows doesn't support mapping single files. The default value is /dev/termination-log, which does work because it does not exist on Windows by default.

##### V1.Pod

* V1.Pod.hostIPC, v1.pod.hostpid - host namespace sharing is not possible on Windows
* V1.Pod.hostNetwork - There is no Windows OS support to share the host network
* V1.Pod.dnsPolicy - ClusterFirstWithHostNet - is not supported because Host Networking is not supported on Windows.
* V1.Pod.podSecurityContext - see V1.PodSecurityContext below
* V1.Pod.shareProcessNamespace - this is a beta feature, and depends on Linux namespaces which are not implemented on Windows. Windows cannot share process namespaces or the container's root filesystem. Only the network can be shared.
* V1.Pod.terminationGracePeriodSeconds - this is not fully implemented in Docker on Windows, see: [reference](https://github.com/moby/moby/issues/25982). The behavior today is that the ENTRYPOINT process is sent CTRL_SHUTDOWN_EVENT, then Windows waits 5 seconds by default, and finally shuts down all processes using the normal Windows shutdown behavior. The 5 second default is actually in the Windows registry [inside the container](https://github.com/moby/moby/issues/25982#issuecomment-426441183), so it can be overridden when the container is built.
* V1.Pod.volumeDevices - this is a beta feature, and is not implemented on Windows. Windows cannot attach raw block devices to pods.
* V1.Pod.volumes - EmptyDir, Secret, ConfigMap, HostPath - all work and have tests in TestGrid
  * V1.emptyDirVolumeSource - the Node default medium is disk on Windows. Memory is not supported, as Windows does not have a built-in RAM disk.
* V1.VolumeMount.mountPropagation - mount propagation is not supported on Windows.

##### V1.PodSecurityContext

None of the PodSecurityContext fields work on Windows. They're listed here for reference.

* V1.PodSecurityContext.SELinuxOptions - SELinux is not available on Windows
* V1.PodSecurityContext.RunAsUser - provides a UID, not available on Windows
* V1.PodSecurityContext.RunAsGroup - provides a GID, not available on Windows
* V1.PodSecurityContext.RunAsNonRoot - Windows does not have a root user. The closest equivalent is ContainerAdministrator which is an identity that doesn't exist on the node.
* V1.PodSecurityContext.SupplementalGroups - provides GID, not available on Windows
* V1.PodSecurityContext.Sysctls - these are part of the Linux sysctl interface. There's no equivalent on Windows.

## 도움받기 및 문제 해결 {#troubleshooting}

Your main source of help for troubleshooting your Kubernetes cluster should start with this [section](/docs/tasks/debug-application-cluster/troubleshooting/). Some additional, Windows-specific troubleshooting help is included in this section. Logs are an important element of troubleshooting issues in Kubernetes. Make sure to include them any time you seek troubleshooting assistance from other contributors. Follow the instructions in the SIG-Windows [contributing guide on gathering logs](https://github.com/kubernetes/community/blob/master/sig-windows/CONTRIBUTING.md#gathering-logs).

1. How do I know start.ps1 completed successfully?

    You should see kubelet, kube-proxy, and (if you chose Flannel as your networking solution) flanneld host-agent processes running on your node, with running logs being displayed in separate PowerShell windows. In addition to this, your Windows node should be listed as "Ready" in your Kubernetes cluster.

1. Can I configure the Kubernetes node processes to run in the background as services?

    Kubelet and kube-proxy are already configured to run as native Windows Services, offering resiliency by re-starting the services automatically in the event of failure (for example a process crash). You have two options for configuring these node components as services.

    1. As native Windows Services

        Kubelet & kube-proxy can be run as native Windows Services using `sc.exe`.

        ```powershell
        # Create the services for kubelet and kube-proxy in two separate commands
        sc.exe create <component_name> binPath= "<path_to_binary> --service <other_args>"

        # Please note that if the arguments contain spaces, they must be escaped.
        sc.exe create kubelet binPath= "C:\kubelet.exe --service --hostname-override 'minion' <other_args>"

        # Start the services
        Start-Service kubelet
        Start-Service kube-proxy

        # Stop the service
        Stop-Service kubelet (-Force)
        Stop-Service kube-proxy (-Force)

        # Query the service status
        Get-Service kubelet
        Get-Service kube-proxy
        ```

    1. Using nssm.exe

        You can also always use alternative service managers like [nssm.exe](https://nssm.cc/) to run these processes (flanneld, kubelet & kube-proxy) in the background for you. You can use this [sample script](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1), leveraging nssm.exe to register kubelet, kube-proxy, and flanneld.exe to run as Windows services in the background.

        ```powershell
        register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>

        # NetworkMode      = The network mode l2bridge (flannel host-gw, also the default value) or overlay (flannel vxlan) chosen as a network solution
        # ManagementIP     = The IP address assigned to the Windows node. You can use ipconfig to find this
        # ClusterCIDR      = The cluster subnet range. (Default value 10.244.0.0/16)
        # KubeDnsServiceIP = The Kubernetes DNS service IP (Default value 10.96.0.10)
        # LogDir           = The directory where kubelet and kube-proxy logs are redirected into their respective output files (Default value C:\k)
        ```

        If the above referenced script is not suitable, you can manually configure nssm.exe using the following examples.
        ```powershell
        # Register flanneld.exe
        nssm install flanneld C:\flannel\flanneld.exe
        nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
        nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
        nssm set flanneld AppDirectory C:\flannel
        nssm start flanneld

        # Register kubelet.exe
        # Microsoft releases the pause infrastructure container at mcr.microsoft.com/k8s/core/pause:1.2.0
        # For more info search for "pause" in the "Guide for adding Windows Nodes in Kubernetes"
        nssm install kubelet C:\k\kubelet.exe
        nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=mcr.microsoft.com/k8s/core/pause:1.2.0 --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
        nssm set kubelet AppDirectory C:\k
        nssm start kubelet

        # Register kube-proxy.exe (l2bridge / host-gw)
        nssm install kube-proxy C:\k\kube-proxy.exe
        nssm set kube-proxy AppDirectory c:\k
        nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
        nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
        nssm set kube-proxy DependOnService kubelet
        nssm start kube-proxy

        # Register kube-proxy.exe (overlay / vxlan)
        nssm install kube-proxy C:\k\kube-proxy.exe
        nssm set kube-proxy AppDirectory c:\k
        nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
        nssm set kube-proxy DependOnService kubelet
        nssm start kube-proxy
        ```


        For initial troubleshooting, you can use the following flags in [nssm.exe](https://nssm.cc/) to redirect stdout and stderr to a output file:

        ```powershell
        nssm set <Service Name> AppStdout C:\k\mysvc.log
        nssm set <Service Name> AppStderr C:\k\mysvc.log
        ```

        For additional details, see official [nssm usage](https://nssm.cc/usage) docs.

1. My Windows Pods do not have network connectivity

    If you are using virtual machines, ensure that MAC spoofing is enabled on all the VM network adapter(s).

1. My Windows Pods cannot ping external resources

    Windows Pods do not have outbound rules programmed for the ICMP protocol today. However, TCP/UDP is supported. When trying to demonstrate connectivity to resources outside of the cluster, please substitute `ping <IP>` with corresponding `curl <IP>` commands.

    If you are still facing problems, most likely your network configuration in [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) deserves some extra attention. You can always edit this static file. The configuration update will apply to any newly created Kubernetes resources.

    One of the Kubernetes networking requirements (see [Kubernetes model](/docs/concepts/cluster-administration/networking/)) is for cluster communication to occur without NAT internally. To honor this requirement, there is an [ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) for all the communication where we do not want outbound NAT to occur. However, this also means that you need to exclude the external IP you are trying to query from the ExceptionList. Only then will the traffic originating from your Windows pods be SNAT'ed correctly to receive a response from the outside world. In this regard, your ExceptionList in `cni.conf` should look as follows:

    ```conf
    "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
    ```

1. My Windows node cannot access NodePort service

    Local NodePort access from the node itself fails. This is a known limitation. NodePort access works from other nodes or external clients.

1. vNICs and HNS endpoints of containers are being deleted

    This issue can be caused when the `hostname-override` parameter is not passed to [kube-proxy](/docs/reference/command-line-tools-reference/kube-proxy/). To resolve it, users need to pass the hostname to kube-proxy as follows:

    ```powershell
    C:\k\kube-proxy.exe --hostname-override=$(hostname)
    ```

1. With flannel my nodes are having issues after rejoining a cluster

    Whenever a previously deleted node is being re-joined to the cluster, flannelD tries to assign a new pod subnet to the node. Users should remove the old pod subnet configuration files in the following paths:

    ```powershell
    Remove-Item C:\k\SourceVip.json
    Remove-Item C:\k\SourceVipRequest.json
    ```

1. After launching `start.ps1`, flanneld is stuck in "Waiting for the Network to be created"

    There are numerous reports of this [issue which are being investigated](https://github.com/coreos/flannel/issues/1066); most likely it is a timing issue for when the management IP of the flannel network is set. A workaround is to simply relaunch start.ps1 or relaunch it manually as follows:

    ```powershell
    PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
    PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
    ```

1. My Windows Pods cannot launch because of missing `/run/flannel/subnet.env`

    This indicates that Flannel didn't launch correctly. You can either try to restart flanneld.exe or you can copy the files over manually from `/run/flannel/subnet.env` on the Kubernetes master to` C:\run\flannel\subnet.env` on the Windows worker node and modify the `FLANNEL_SUBNET` row to a different number. For example, if node subnet 10.244.4.1/24 is desired:

    ```env
    FLANNEL_NETWORK=10.244.0.0/16
    FLANNEL_SUBNET=10.244.4.1/24
    FLANNEL_MTU=1500
    FLANNEL_IPMASQ=true
    ```

1. My Windows node cannot access my services using the service IP

    This is a known limitation of the current networking stack on Windows. Windows Pods are able to access the service IP however.

1. No network adapter is found when starting kubelet

    The Windows networking stack needs a virtual adapter for Kubernetes networking to work. If the following commands return no results (in an admin shell), virtual network creation — a necessary prerequisite for Kubelet to work — has failed:

    ```powershell
    Get-HnsNetwork | ? Name -ieq "cbr0"
    Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
    ```

    Often it is worthwhile to modify the [InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) parameter of the start.ps1 script, in cases where the host's network adapter isn't "Ethernet". Otherwise, consult the output of the `start-kubelet.ps1` script to see if there are errors during virtual network creation.

1. My Pods are stuck at "Container Creating" or restarting over and over

    Check that your pause image is compatible with your OS version. The [instructions](https://docs.microsoft.com/en-us/virtualization/windowscontainers/kubernetes/deploying-resources) assume that both the OS and the containers are version 1803. If you have a later version of Windows, such as an Insider build, you need to adjust the images accordingly. Please refer to the Microsoft's [Docker repository](https://hub.docker.com/u/microsoft/) for images. Regardless, both the pause image Dockerfile and the sample service expect the image to be tagged as :latest.

    Starting with Kubernetes v1.14, Microsoft releases the pause infrastructure container at `mcr.microsoft.com/k8s/core/pause:1.2.0`. For more information search for "pause" in the [Guide for adding Windows Nodes in Kubernetes](../user-guide-windows-nodes).

1. DNS resolution is not properly working

    Check the DNS limitations for Windows in this [section](#dns-limitations).

1. `kubectl port-forward` fails with "unable to do port forwarding: wincat not found"

    This was implemented in Kubernetes 1.15, and the pause infrastructure container `mcr.microsoft.com/k8s/core/pause:1.2.0`. Be sure to use these versions or newer ones.
    If you would like to build your own pause infrastructure container, be sure to include [wincat](https://github.com/kubernetes-sigs/sig-windows-tools/tree/master/cmd/wincat)

### 추가 조사
### Further investigation

If these steps don't resolve your problem, you can get help running Windows containers on Windows nodes in Kubernetes through:

* StackOverflow [Windows Server Container](https://stackoverflow.com/questions/tagged/windows-server-container) topic
* Kubernetes Official Forum [discuss.kubernetes.io](https://discuss.kubernetes.io/)
* Kubernetes Slack [#SIG-Windows Channel](https://kubernetes.slack.com/messages/sig-windows)

## 이슈 보고 및 기능 요청

If you have what looks like a bug, or you would like to make a feature request, please use the [GitHub issue tracking system](https://github.com/kubernetes/kubernetes/issues). You can open issues on [GitHub](https://github.com/kubernetes/kubernetes/issues/new/choose) and assign them to SIG-Windows. You should first search the list of issues in case it was reported previously and comment with your experience on the issue and add additional logs. SIG-Windows Slack is also a great avenue to get some initial support and troubleshooting ideas prior to creating a ticket.

If filing a bug, please include detailed information about how to reproduce the problem, such as:

* Kubernetes version: kubectl version
* Environment details: Cloud provider, OS distro, networking choice and configuration, and Docker version
* Detailed steps to reproduce the problem
* [Relevant logs](https://github.com/kubernetes/community/blob/master/sig-windows/CONTRIBUTING.md#gathering-logs)
* Tag the issue sig/windows by commenting on the issue with `/sig windows` to bring it to a SIG-Windows member's attention

{{% /capture %}}

{{% capture whatsnext %}}

We have a lot of features in our roadmap. An abbreviated high level list is included below, but we encourage you to view our [roadmap project](https://github.com/orgs/kubernetes/projects/8) and help us make Windows support better by [contributing](https://github.com/kubernetes/community/blob/master/sig-windows/).

### CRI-ContainerD

{{< glossary_tooltip term_id="containerd" >}}는 최근에 {{< glossary_tooltip text="CNCF" term_id="cncf" >}} 프로젝트를 졸업한 또 다른 OCI 호환 런타임이다. 이는 리눅스에서 현재 테스트되었지만 1.3에서는 윈도우와 Hyper-V를 지원할 예정이다. [[참고](https://blog.docker.com/2019/02/containerd-graduates-within-the-cncf/)]
{{< glossary_tooltip term_id="containerd" >}} is another OCI-compliant runtime that recently graduated as a {{< glossary_tooltip text="CNCF" term_id="cncf" >}} project. It's currently tested on Linux, but 1.3 will bring support for Windows and Hyper-V. [[reference](https://blog.docker.com/2019/02/containerd-graduates-within-the-cncf/)]

CRI-ContainerD 인터페이스는 Hyper-V에 기반하여 샌드박스를 관리할 수 있다. 이는 새로운 다음과 같은 사용 예를 구현할 수 있는 RuntimeClass 기반을 제공한다.
The CRI-ContainerD interface will be able to manage sandboxes based on Hyper-V. This provides a foundation where RuntimeClass could be implemented for new use cases including:

* 추가적인 보안성을 위한 파드 간에 하이퍼바이저 기반 격리화
* 컨테이너를 새로 빌드할 필요없이 새로운 윈도우 서버 버전을 구동할 수 있는 노드를 가능케 하는 하위 호환성
* 파들을 위한 특화된 CPU/NUMA 설정
* 메모리 격리화와 예약
* Hypervisor-based isolation between pods for additional security
* Backwards compatibility allowing a node to run a newer Windows Server version without requiring containers to be rebuilt
* Specific CPU/NUMA settings for a pod
* Memory isolation and reservations

### Hyper-V 격리

v1.10의 실험적인 기능인 존재하는 Hyper-V 격리화 지원은 위에서 언급한 CRI-ContainerD와 RuntimeClass 기능의 기호를 미래에 더 이상 사용되지 않을 것이다. Hyper-V 격리화된 컨테이너를 생성하고 현재 기능을 사용하기 위해서 Kubelet이 기능 게이트는 `HyperVContainer=true`로 파드는 `experimental.windows.kubernetes.io/isolation-type=hyperv` 어노테이션을 포함해서 시작해야 한다. 실험 릴리스에서 이 기능은 파드당 1 컨테이너로 제한된다.
The existing Hyper-V isolation support, an experimental feature as of v1.10, will be deprecated in the future in favor of the CRI-ContainerD and RuntimeClass features mentioned above. To use the current features and create a Hyper-V isolated container, the kubelet should be started with feature gates `HyperVContainer=true` and the Pod should include the annotation `experimental.windows.kubernetes.io/isolation-type=hyperv`. In the experiemental release, this feature is limited to 1 container per Pod.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iis
spec:
  selector:
    matchLabels:
      app: iis
  replicas: 3
  template:
    metadata:
      labels:
        app: iis
      annotations:
        experimental.windows.kubernetes.io/isolation-type: hyperv
    spec:
      containers:
      - name: iis
        image: microsoft/iis
        ports:
        - containerPort: 80
```

### Kubeadm을 통한 배포 및 클러스터 API

Kubeadm은 일반 사용자가 쿠버네티스 클러스터 배포하기 위한 산업 표준이 되었다. 향후 릴리스에서 Kubeadm에서 윈도우 노드를 지원할 예정이다. 또한 윈도우 노드가 올바르게 프로비저닝되도록 클러스터 API에 투자하고 있다.

### 다른 주요 기능 몇 가지
* 그룹 매니지드 서비스 어카운트(GMSA)에 대한 베타 지원
* 여러 CNI
* 여러 스토리지 플러그인

{{% /capture %}}
