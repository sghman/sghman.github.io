---
title: Kubespray를 이용한 Kubernetes 클러스터 구축 가이드 (폐쇄망 환경 중심)
author: gyuhwan
date: 2025-04-01 12:40:00 +0800
categories: [DevOps, k8s, Kubespray]
tags: [DevOps]
---
# Kubespray를 이용한 Kubernetes 클러스터 구축 가이드 (폐쇄망 환경 중심)

# 1. 서론

본 문서는 Kubespray를 활용하여 kubernetes 클러스터를 구축하는 과정을 상세히 안내하는 것을 목적으로 합니다. 특히, 폐쇄망 환경에서의 구축 절차와 고려 사항을 중점적으로 다룹니다.

## 1-1 구축 목표: Kubernetes 클러스터(폐쇄망)

- **운영체제**: Rocky Linux 9
- **Kubespray 버전**: v2.24.2
- **Kubernetes 버전**: v1.28.10
- 클러스터 구성: 고가용성(HA)을 갖춘 Kubernetes 클러스터
  - 컨트롤 플레인 노드(마스터 노드): 3대
  - 워커 노드: 2대
  - 부트스트랩 노드: 1대
- 배포 환경: 외부 인터넷 연결이 차단된 폐쇄망 환경

## 1-2 Kubespray 버전 선택 가이드

Kubespray의 릴리즈는 주로 Git 저장소의 **태그(Tag)** 로 관리됩니다. 각 태그는 특정 시점의 Kubespray 코드베이스를 나타내며, 해당 버전에서 지원하는 Kubernetes 버전 범위가 정의되어 있습니다. 일반적으로 Kubespray 릴리즈 노트나 공식 문서에서 각 버전별 주요 특징 및 지원하는 Kubernetes 버전을 확인할 수 있습니다.

| Kubespray Tag | 지원 kubernetes 버전 |
| --- | --- |
| v2.28.0(Latest) | kubernetes v1.32.5 |
| v2.27.0 | kubernetes v1.29.x - v1.31.x |
| v2.26.0 | kubernetes v1.27.x - v1.30.x |
| v2.25.1 | kubernetes v1.29.10 |
| v2.24.2 | kubernetes v1.28.10 |

## 1-3 Kubespray 주요 지원 사항

- **다양한 플러그인 지원**: Calico, Cilium, Flannel 등 다양한 네트워크 플러그인(CNI)과 컨테이너 런타임(containerd, CRI-O) 중에서 선택하여 구성할 수 있습니다.
- **지원 가능한 Linux 배포판**
  - Flatcar Container Linux by Kinvolk
  - Debian (예: Bullseye, Buster, Jessie, Stretch)
  - Ubuntu (예: 16.04, 18.04, 20.04, 22.04)
  - CentOS / RHEL (예: 7, 8, 9)
  - Fedora (예: 35, 36)
  - Fedora CoreOS
  - openSUSE (예: Leap 15.x, Tumbleweed)
  - Oracle Linux (예: 7, 8, 9)
  - AlmaLinux (예: 8, 9)
  - Rocky Linux (예: 8, 9)
  - Kylin Linux Advanced Server V10
  - Amazon Linux 2


---

# 2. 사전 준비 사항

## 2-1 하드웨어 요구 사항

클러스터는 총 3대의 컨트롤 플레인 노드, 2대의 워커 노드, 그리고 1대의  부트스트랩 노드로 구성됩니다. 각 노드의 최소 권장 사양은 다음과 같습니다.

- **컨트롤 플레인 노드 (3대)**
  - CPU: 2 코어 이상
  - RAM: 4GB 이상 (Kubespray `v2.24.x`에서는 2GB 이상을 권장하지만, 안정적인 HA 운영을 위해 4GB 권장)
  - Disk: 40GB 이상
- **워커 노드 (2대)**
  - CPU: 2코어 이상
  - RAM: 4GB 이상
  - Disk: 40GB 이상의 여유 공간
- **부트스트랩 노드 (1대)**
  - CPU: 2코어 이상
  - RAM: 4GB 이상
  - Disk: 50GB 이상의 여유 공간 (Kubespray 소스 코드, 다운로드한 패키지, 컨테이너 이미지 임시 저장 등 고려)

## 2-2 폐쇄망 환경 일반 고려 사항

- **필요한 모든 파일 준비**: Kubespray 소스 코드, RPM패키지, 컨테이너 이미지 아카이브 등
- **필수 RPM 패키지 사전 준비 (Rocky Linux 9 기준)**
  - Python 3.9+ 및 python3-pip
  - Ansible v2.11+ (ansible-core)
  - git

## 2-3 Kubespray 소스 코드 준비

- **Kubespray 소스 코드 다운로드 및 버전 선택**

    ```bash
    root@bootstrap ~]# git clone https://github.com/kubernetes-sigs/kubespray.git
    root@bootstrap ~]# cd kubespray
    root@bootstrap kubespray]# git checkout v2.24.2
    ```


## **2-4  폐쇄망 컨테이너 이미지 준비**

폐쇄망 환경에서는 모든 컨테이너 이미지가 내부 레지스트리에 준비되어 있어야 합니다. 여기서는 **Podman**을 사용하여 레지스트리로 사전에 구축했다고 가정합니다.

### 2-4-1 이미지 목록 생성**(generate_list.sh 활용)**

- **Kubespray 설정 반영**: **inventory/mycluster/group_vars** 아래 원하는 kubernetes 버전(kube_version: v1.28.10) 등을 설정합니다.
- **이미지 목록 생성 스크립트 실행**: Kubespray 소스 코드 내 **contrib/offline/generate_list.sh** 스크립트를 부트스트랩 노드에서 실행합니다.

    ```bash
    root@bootstrap offline]# ./generate_list.sh
    root@bootstrap offline]# ls temp/
    files.list  files.list.template  images.list  images.list.template
    ```


### 2-4-2 이미지 다운로드 및 패키징(manage-offline-container-images.sh 활용)

생성된 **images.list**를 사용하여 인터넷이 가능한 환경에서 이미지를 다운로드하고 패키징합니다.

- **참고 사항**
  - Kubespray v2.24.2 에 포함된 **manage-offline-container-images.sh** 스크립트의 create 기능은 기본적으로 kubectl을 사용하여 실행 중인 클러스터에서 이미지 목록을 추출하도록 설계되어 있습니다.
  - Kubespray 최신 버전(예: v2.28.0)에 포함된 스크립트는 kubectl이 아닌 **IMAGES_FROM_FILE** 에 있는 경로의 파일을 읽어 이미지 목록을 추출합니다.
- **최신 버전의 manage-offline-container-images.sh 활용**

    ```bash
    root@bootstrap offline]# IMAGES_FROM_FILE=./temp/images.list ./manage-offline-container-images.sh create
    ```

- **이미지 파일 위치: container-images**
- **이미지 압축 파일: container-images.tar.gz**

# 3. Kubespray 프로젝트 구조 이해

Kubespray 프로젝트는 다양한 디렉토리와 파일로 구성되어 있습니다. 각 요소는 Kubernetes 클러스터의 설치, 설정, 관리 작업을 수행하는 데 필요한 역할을 합니다.

- **주요 Ansible 플레이북(.yml 파일)**
  - **cluster.yml** : Kubernetest 클러스터 설치를 위한 메인 플레이북입니다.
  - **reset.yml** :  Kubespray로 설치된 Kubernetes 클러스터를 초기화하고 관련 구성 요소들을 제거하는 플레이북입니다.
  - **upgrade-cluster.yml**: 기존 Kubernetes 클러스터의 버전을 업그레이드하는 플레이북입니다.
  - **scale.yml**: 실행 중인 Kubernetes 클러스터에 워커 노드를 추가하는 플레이북입니다.
  - **remove-node.yml**: 클러스터에서 특정 노드를 제거하는 플레이북입니다.

### 3-1 inventory 디렉토리: 클러스터 구성 정의

inventory 디렉토리는 Ansible 인벤토리 파일과 클러스터 설정을 위한 변수 파일들을 저장하는 중요한 위치입니다.

- **inventory/sample**
  - 사용자 정의가 가능한 예시 인벤토리 파일 및 group_vars 디렉토리 구조를 제공합니다. 일반적으로 sample 디렉토리를 복사하여 사용자 전용 인벤토리 디렉토리를 만드는 것이 일반적인 방법입니다.
- **inventory/<mycluster>/inventory.ini**
  - 클러스터를 구성할 서버들의 목록과 서버들이 속할 그룹(kube_control_plane, kube_node, etcd, k8s_cluster 등)을 정의하는 파일입니다.
- **inventory/<mycluster>/group_vars**
  - **all/all.yml** : 클러스터의 모든 노드에 공통적으로 적용되는 변수를 정의합니다.
  - **k8s_cluster/k8s_cluster.yml** : k8s_cluster 그룹에 적용되는 Kubernetes 관련 핵심 변수들을 정의합니다.(kube_version, container_manager)
  - **all/offline.yml**: 사용자가 직접 추가하여 폐쇄망 설치에 필요한 내부 컨테이너 레지스트리 주소, 파일 서버 정보 등의 변수를 정의합니다.

### 3-2 roles 디렉토리: Ansible 역할(Role) 정의

이 디렉토리에는 Kubespray의 핵심 로직이 담긴 Ansible 역할(Role)들이 모여 있습니다.

- **주요 역할**
  - **bootstrap-os**: 노드의 운영체제 환경을 Kubernetes 설치에 적합하도록 준비합니다.(필수 패키지 설치, 커널 파라미터 설정 등)
  - **container-engine**: Containerd, CRI-O 등의 컨테이너 런타임을 설치하고 설정합니다.
  - **etcd**: etcd 클러스터를 설치하고 설정합니다.
  - **kubernetes/control-plane**: Kubernetes 컨트롤 플레인 컴포넌트(API 서버, 스케줄러, 컨트롤러 매니저)를 설치합니다.
  - **kubernetes/node**: Kubernetes 워커 노드 컴포넌트(kubelet, kube-proxy)를 설치합니다.
  - **network_plugin**: Calico, Flannel, Cilium 등 선택한 CNI(Container Network Interface) 플러그인을 설치하고 설정합니다.
  - **kubespray-defaults**: Kubespray 전반에 사용될 수 있는 기본 변수 값들을 일부 포함할 수 있습니다.


### 3-3 requirements.txt: Python 의존성 패키지

Kubespray를 실행하는 Ansible 컨트롤러 노드에 필요한 Python 패키지들의 목록과 버전이 정의된 파일입니다. 설치되는 파일은  netaddr, jmespath, ansible 등이 있습니다. (폐쇄망 환경에서는 이 패키지들도 사전에 다운로드하여 준비해야 합니다.)

```bash
root@bootstrap kubespray]# pip3 install -r requirements.txt
```

### 3-4 contrib 디렉토리: 추가 스크립트 및 도구

Kubespray 사용에 도움이 되는 다양한 추가 스크립트, 도구, 예제 설정 등을 포함하는 디렉토리입니다.

- 폐쇄망 지원: 오프라인 설치를 위한 스크립트가 포함되어 있습니다. (**contrib/offline/**)
- 클라우드 프로바이더별 설정 스크립트나 기타 유틸리티들이 포함되어 있습니다.

1. **Kubespray 프로젝트 구조 이해**
  1. 주요 디렉토리 및 파일 설명
    - **inventory/**: 인벤토리 파일 및 group_vars 위치
    - **roles/**: Ansible 역할 정의
    - **.yml**: 주요 플레이북 (e.g., cluster.yml, scale.yml, reset.yml)
    - **requirements.txt**: Python 의존성 패키지
    - **contrib/**: 추가 스크립트 및 도구 (오프라인/에어갭 관련 도구 포함 4. 가능)

# 4. 환경 설정

Kubespray를 사용하여 Kubernetes 클러스터를 배포하기위해 사전에 각 노드에 대한 몇 가지 환경 설정 작업을 선행합니다.

- **/etc/hosts 파일 설정**: 모든 클러스터 노드에 IP 주소와 호스트명을 등록하여 호스트명으로 접근이 가능하도록 설정합니다.

    ```bash
    # /etc/hosts 파일 예시
    127.0.1.1 bootstrap bootstrap
    192.168.20.40 bootstrap
    192.168.20.41 master-1
    192.168.20.42 master-2
    192.168.20.43 master-3
    192.168.20.44 worker-1
    192.168.20.45 worker-2
    ```

- **SELinux 설정**: Kubespray 실행 중 발생할 수 있는 권한 문제를 방지하기 위해 SELinux 모드를 비활성화합니다.

    ```bash
    root@bootstrap kubespray]# setenforce 0
    root@bootstrap kubespray]# sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    ```

- **방화벽 설정**: Ansible 접속 및 설정의 편의를 위해 firewalld를 중지하고 비활성화 합니다.

    ```bash
    root@bootstrap kubespray]# systemctl stop firewalld
    root@bootstrap kubespray]# systemctl disable firewalld
    ```

- **Python 설치**

    ```bash
    root@bootstrap ~]# dnf install -y python3 python3-pip
    ```

- **Kubespray Python 의존성 설치**:  Kubespray 소스 코드 내 requirements.txt 파일에 명시된 Python 패키지들도 사전 다운로드하여 설치합니다.

    ```bash
    root@bootstrap kubespray]# pip3 install -r requirements.txt
    ```

- **SSH 키 생성**: 부트스트랩 노드에서 다른 노드에서 SSH 접근하기 위한 SSH 키를 생성합니다.

    ```bash
    root@bootstrap kubespray]# ssh-keygen -t rsa -b 4096
    
    # 다른 노드에 공개키 복사(master-1 ~ worker-2)
    root@bootstrap kubespray]# ssh-copy-id root@192.168.25.41
    ```


# 5. Kubespray 설정 커스터마이징

kubespray 디렉토리 내에서 **cp -rfp inventory/sample inventory/mycluster** 명령을 실행하여 ‘**mycluster’**라는 이름의 새로운 인벤토리 설정을 생성하고, 이 디렉토리 내의 파일들을 수정합니다.

## 5-1 인벤토리 파일 작성(inventory/mycluster/inventory.ini)

**inventory.ini** 파일은 클러스터를 구성할 노드들의 정보와 각 노드가 속할 그룹을 정의합니다.

- **호스트 그룹 정의**
  - **[kube_control_plane]**: 컨트롤 플레인 노드(마스터 노드)
  - **[etcd]**: etcd 클러스터 맴버 노드
  - **[kube_node]**: 워커 노드
  - **[k8s_cluster:children]**: kube_control_plane과 kube_node 그룹을 포함하는 상위 그룹

예시: **inventory/mycluster/inventory.ini**

```bash
[all]
master-1 ansible_host=192.168.25.41 ip=192.168.25.41 etcd_member_name=etcd1
master-2 ansible_host=192.168.25.42 ip=192.168.25.42 etcd_member_name=etcd2
master-3 ansible_host=192.168.25.43 ip=192.168.25.43 etcd_member_name=etcd3
worker-1 ansible_host=192.168.25.44 ip=192.168.25.44
worker-2 ansible_host=192.168.25.45 ip=192.168.25.45

[kube_control_plane]
master-1
master-2
master-3

[etcd]
master-1
master-2
master-3

[etcd:children]
kube_control_plane

[calico_rr]

[kube_node]
master-1
master-2
master-3
worker-1
worker-2

[k8s_cluster:children]
kube_control_plane
kube_node
```

## 5-2 내부 컨테이너 레지스트리 연동 설정

폐쇄망 환경에서는 모든 컨테이너 이미지가 사전에 준비된 내부 컨테이너 레지스트리에 저장되어 있어야 합니다.

### 5-2-1 registry_host 변수 사용(group_vars/offline.yml)

```bash
## Global Offline settings
### Private Container Image Registry
registry_host: "192.168.25.40:5000"
```

### 5-2-2 containerd_registries_mirrors 설정(group_vars/all/containerd.yml)

**inventory/mycluster/group_vars/all/containerd.yml**

```bash
# Registries defined within containerd.
containerd_registries_mirrors:
 - prefix: docker.io
   mirrors:
     - host: http://192.168.25.40:5000
      capabilities: ["pull", "resolve"]
      skip_verify: true
```

### 5-2-3 레지스트리 http 접근을 위한 설정(roles/container-engine/containerd/templates/hosts.toml.j2)

```bash
server = "http://{{ item.prefix }}"
```

# 6. Kubernetes 클러스터 배포

이전 단계를 통해 모든 사전 준비와 Kubespray 설정을 커스터마이징이 완료되었다면, 이제 Ansible 플레이북을 실행하여 Kubernetes 클러스터(v1.28.10)를 배포할 차례입니다.

## 6-1 Ansible 플레이북 실행(cluster.yml)

kubespray의 메인 플레이북인 **cluster.yml** 을 실행하여 Kubernetes 클러스터 설치를 시작합니다.

### 6-1-1 클러스터 배포 명령어 실행

부트스트랩 노드의 Kubespray 소스 코드 디렉토리에서 다음 명령어를 실행합니다.

```bash
(kubespray-venv) root@bootstrap-node:~/kubespray$ ansible-playbook \
    --forks 10 \
    -i inventory/mycluster/inventory.ini \
    cluster.yml \
    -b \
    -e ansible_ssh_timeout=50 \
    -vvv \
    2>&1 | tee -a "install_$(date +%Y%m%d_%H%M%S).log"
```

- **주요 옵션 설명**
  - **—forks 10** : Ansible이 동시에 작업을 수행할 병렬 프로세스(스레드) 수를 의미합니다.
  - **-i inventory/mycluster/inventory.ini**: 사용할 인벤토리 파일의 경로를 지정합니다.
  - **cluster.yml**: 실행할 메인 플레이북 파일명입니다.
  - **-b —become-user root**: 원격 노드에서 권한 상승하여 root 사용자로 작업을 수행하도록 지정합니다.
  - **-e annsible_ssh_timeout=50**: Ansible의 SSH 타임아웃을 50초로 설정합니다.
  - **-vvv**: v 개수에 따라 로그 출력 수준이 달라집니다. vvv는 상위 수준으로 상세한 로그를 출력합니다.
  - **2>&1**: 포준 오류(stderr)를 포준(stdout)으로 리디렉션합니다.
  - **tee -a “install_$(date+%Y%m%d_%H%M%S).log”**: 실행 시점의 날짜와 시간을 포함하는 로그 파일을 동적으로 생성합니다.

### 6-1-2 클러스터 정상 배포 확인

- **노드 상태 확인**

    ```bash
    [root@master-1 ~]# kubectl get nodes
    NAME       STATUS   ROLES           AGE   VERSION
    master-1   Ready    control-plane   14h   v1.28.10
    master-2   Ready    control-plane   14h   v1.28.10
    master-3   Ready    control-plane   14h   v1.28.10
    worker-1   Ready    <none>          14h   v1.28.10
    worker-2   Ready    <none>          14h   v1.28.10
    ```

- **시스템 파드 상태 확인**

    ```bash
    [root@master-1 ~]# kubectl get pod -A
    NAMESPACE     NAME                                      READY   STATUS    RESTARTS        AGE
    kube-system   calico-kube-controllers-648dffd99-r6hf9   1/1     Running   0               14h
    kube-system   calico-node-cd7fg                         1/1     Running   0               14h
    kube-system   calico-node-nwn7r                         1/1     Running   1 (14h ago)     14h
    kube-system   calico-node-p4gj4                         1/1     Running   0               14h
    kube-system   calico-node-pbjcl                         1/1     Running   0               14h
    kube-system   calico-node-qr5qk                         1/1     Running   0               14h
    kube-system   coredns-77f7cc69db-f97pc                  1/1     Running   0               14h
    kube-system   coredns-77f7cc69db-x4rkw                  1/1     Running   0               14h
    kube-system   dns-autoscaler-8576bb9f5b-pfgmc           1/1     Running   0               14h
    kube-system   kube-apiserver-master-1                   1/1     Running   2               14h
    kube-system   kube-apiserver-master-2                   1/1     Running   1               14h
    kube-system   kube-apiserver-master-3                   1/1     Running   1               14h
    kube-system   kube-controller-manager-master-1          1/1     Running   3 (4h46m ago)   14h
    ...
    ```


# 7 클러스터 운영 및 관리

Kubespray는 클러스터 설치뿐만 아니라 노드 추가/제거, 업그레이드, 리셋 등 클러스터 생명주기 관리를 위한 다양한 Ansible 플레이북을 제공합니다.

## 7-1 노드 추가(scale.yml)

기존 클러스터에 새로운 워커 노드나 컨트롤 플레인 노드를 추가할 때, **scale.yml** 플레이북을 사용합니다.

### 7-1-1 Inventory 수정

inventory/mycluster/inventory.ini 파일에 새로 추가할 노드의 정보를 추가합니다.

```bash

[all]
master-1 ansible_host=192.168.25.41 ip=192.168.25.41 etcd_member_name=etcd1
master-2 ansible_host=192.168.25.42 ip=192.168.25.42 etcd_member_name=etcd2
master-3 ansible_host=192.168.25.43 ip=192.168.25.43 etcd_member_name=etcd3
worker-1 ansible_host=192.168.25.44 ip=192.168.25.44
worker-2 ansible_host=192.168.25.45 ip=192.168.25.45
worker-3 ansible_host=192.168.25.46 ip=192.168.25.46  <-- 추가
...

[kube_node]
master-1
master-2
master-3
worker-1
worker-2
worker-3 <-- 추가
```

### 7-1-2 `scale.yml` 플레이북 실행

```bash
(kubespray-venv) root@bootstrap-node:~/kubespray$ ansible-playbook \
    -i inventory/mycluster/inventory.ini \
    scale.yml \
    -b \
    -vvv \
    2>&1 | tee -a "scale_$(date +%Y%m%d_%H%M%S).log"
```

## 7-2 노드 제거(remove-node.yml)

클러스터에서 특정 노드를 안전하게 제거할 때, `remove-node.yml` 플레이북을 사용합니다.

### 7-2-1 `remove-node.yml` 플레이북 실행

- **-e node=** 옵션을 사용하여 제거할 노드의 이름을 지정합니다.

    ```bash
    (kubespray-venv) root@bootstrap-node:~/kubespray$ ansible-playbook \
        -i inventory/mycluster/inventory.ini \
        remove-node.yml \
        -e node=worker-3 \ 
        -b \
        -vvv \
        2>&1 | tee -a "remove_node_$(date +%Y%m%d_%H%M%S).log"
    ```


## 7-3 노드 업그레이드(upgrade-cluster.yml)

Kubespray는 기존 Kubernetes 클러스터의 마이너 버전 또는 패치 버전 업그레이드를 지원합니다.

### 7-3-1 업그레이드 준비 단계

- **중요 데이터 백업**
  - Etcd 스냅샷 백업
  - 애플리케이션 데이터 백업
- **새 버전 이미지 준비(폐쇄망 환경)**
- **Kubespray 인벤토리 변수 업데이트**
  - kube_version 변경: inventory/mycluster/group_vars/k8s_cluster.yml 파일에서 kube_version 변수의 값을 업그레이드할 버전으로 명시합니다.
- **Kubespray 버전 간 업그레이드 고려 사항**
  - 순차적 업그레이드: Kubespray Git 저장소에서 다음 릴리즈 태그로 순차적 업그레이드를 권장합니다. 한 번에 많은 버전을 건너뛰는 것은 권장되지 않습니다.
  - Python 의존성 업데이트: 새로운 Kubespray 버전은 **requirements.txt** 파일이 변경되었을 수 있으므로, Python 의존성을 새로 업데이트해야 합니다.

### 7-3-2 `upgrade-cluster.yml` 플레이북 실행

```bash
(kubespray-venv) root@bootstrap-node:~/kubespray$ ansible-playbook \
    -i inventory/mycluster/inventory.ini \
    upgrade-cluster.yml \
    -b \
    -e "serial=1" \ # (권장) 한 번에 하나의 노드만 순차적으로 업그레이드 (특히 컨트롤 플레인)
    -vvv \
    2>&1 | tee -a "upgrade_cluster_$(date +%Y%m%d_%H%M%S).log"
```

## 7-4 클러스터 리셋/삭제(reset.yml)

`reset.yml` 플레이북은 Kubespray로 배포된 Kubernetes 클러스터의 모든 구성 요소와 설정을 노드에서 제거하여 초기 상태로 되돌립니다.

### 7-4-1 `reset.yml` 플레이북 실행

```bash
(kubespray-venv) root@bootstrap-node:~/kubespray$ ansible-playbook \
    -i inventory/mycluster/inventory.ini \
    reset.yml \
    -b \
    -vvv \
    2>&1 | tee -a "reset_cluster_$(date +%Y%m%d_%H%M%S).log"
```

# 8. Kubespray/Ansible 로그 확인 및 디버깅

문제가 발생했을 때 Ansible 플레이북의 실행 로그를 효과적으로 분석하고, 특정 부분만 재실행하거나 점검하는 것은 문제 해결 시간을 단축시킬 수 있습니다.

- **로그 상세 수준 높이기(-v, -vv, -vvv, -vvvv ):** ansible-playbook 실행 시 -v 옵션을 단계별로 추가하여 로그의 상세 수준을 높일 수 있습니다.
  - **-v** : 기본적인 추가 정보
  - **-vv**: 모듈의 입력 파라미터 등 더 많은 정보
  - **-vvv**: 디버깅 시 매우 유용. 각 모듈 실행 시 전달되는 모든 파라미터, 반환된 결과값, SSH 연결 시도에 대한 상세 정보, 실패 시 자세한 오류 메시지 등을 보여줍니다.
  - **-vvvv**: SSH 연결 자체에 대한 매우 상세한 디버깅 정보를 포함합니다.

    ```bash
    ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b -vvv 2>&1 | tee -a "debug_install.log"
    ```

- **실패한 Task 재시도 및 특정 호스트 대상 실행**(**—limit**) : 플레이북 실행 중 특정 노드나 작업에서 오류가 발생했다면, 전체를 다시 실행하는 대신 해당 부분만 재시도합니다.
  - **—limit**: 플레이북 실행 대상을 특정 호스트나 그룹으로 제한합니다.
    - 예시 1: `worker-2` 노드에서만 플레이북 재실행

        ```bash
        ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b --limit=worker-2
        ```

- **특정 작업(Task)만 실행하거나 건너뛰기(—tages, —skip-tags**) : Kubespray의 플레이북과 역할(Role) 내 작업들에는 태그(tag)가 지정되어 있습니다. 이를 활용하여 특정 부분만 실행하거나 건너뛸 수 있습니다.
  - **사용 가능한 태그 목록 확인(—list-tags)**

      ```bash
      (kubespray) [root@bootstrap kubespray]# ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml --list-tags
      playbook: cluster.yml
      
        play #1 (localhost): Check Ansible version    TAGS: [always]
            TASK TAGS: [always, check]
      
        play #2 (kube-master): Add kube-master nodes to kube_control_plane    TAGS: [always]
            TASK TAGS: [always]
      
        play #3 (kube-node): Add kube-node nodes to kube_node TAGS: [always]
            TASK TAGS: [always]
      
        play #4 (k8s-cluster): Add k8s-cluster nodes to k8s_cluster   TAGS: [always]
            TASK TAGS: [always]
      
        play #5 (calico-rr): Add calico-rr nodes to calico_rr TAGS: [always]
            TASK TAGS: [always]
      
        play #6 (no-floating): Add no-floating nodes to no_floating   TAGS: [always]
            TASK TAGS: [always]
      
        play #7 (bastion[0]): Install bastion ssh config      TAGS: []
            TASK TAGS: [always, bastion, localhost]
        ...
      ```

  - **특정 태그 작업만 실행(—tags, -t)**

      ```bash
      ansible-playbook -i inventory/mycluster/inventory-.ini cluster.yml -b --tags=etcd
      ```

  - **특정 태그 작업 건너뛰기(—skip-tags)**

      ```bash
      ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b --skip-tags=apps/problematic-addon
      ```
