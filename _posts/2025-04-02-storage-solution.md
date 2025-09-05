---
title: Ceph, Longhorn, OpenEBS 비교
author: gyuhwan
date: 2025-05-04 13:20:00 +0800
categories: [DevOps, k8s, storage]
tags: [DevOps]
---
# Ceph, Longhorn, OpenEBS

# 1. 소개

Kubernetes 환경에서 Ceph, Longhorn, OpenEBS, 의 주요 특징과 장점을 분석하여 각 상황에 맞는 스토리지 솔루션 도입을 위해 작성했습니다. 확장성, 성능, 복잡성, 설치 편의성 등을 기준으로 평가하며, 현재 사용 중인 **Spheros CMP** 환경에 적합한 솔루션 도입 가능성을 분석합니다.

- **Ceph**: 대규모 환경에서 높은 성능과 확장성을 제공하지만 설정과 관리가 복잡합니다.
- **OpenEBS**: Kubernetes 네이티브 스토리지로 유연한 구성과 제어가 가능합니다
- **Longhorn**: 간편한 설치 및 관리와 직관적인 UI를 제공하며 소규모 환경에 적합합니다.

# 2. 스토리지 솔루션 개요

## 2-1 Ceph

![image.png](/commons/storage/6.png)

### 2-1-1 아키텍처

- 분산 스토리지 시스템: 블록, 파일 객체 스토리지 모두 지원
- 구성 요소:
  - **MON(Monitory)**: 클러스터 상태 관리
  - **OSD(Object Storage Daemon)**: 실제 데이터 저장 및 복제, 장애 발생 시 데이터 복구
  - **MGR(Manager)**:  클러스터 상태를 모니터링하고 메트릭을 수집하며, Ceph의 내부 기능을 관리
  - **RADOS(Reliable Autonomic Distributed Object Store)**: 데이터를 분산 저장하고 복제하여 고가용성을 제공합니다.
  - **Rook Operator**: kubernetes 환경에서 Ceph 클러스터를 부트스트랩하고 관리하며, CSI 드라이버를 통해 스토리지를 Pod에 마운트합니다.

### 2-1-2 확장성

새로운 워커 노드를 추가하거나 워커 노드에 새로운 디스크를 추가 장착하면 Rook Operator가 이를 자동으로 감지하고 해당 raw 디스크를 관리하는 OSD 파드를 생성합니다. 생성된 OSD Pod이 Ceph 클러스터에 추가되면 Ceph 클러스터의 용량이 자동으로 확장됩니다. Ceph는 데이터를 새로운 OSD에 자동으로 재분배하여 클러스터의 균형을 유지하며, 분산 및 복제된 데이터를 통해 **고가용성**을 보장합니다.

![image.png](/commons/storage/1.png)

### 2-1-3 편의성

Rook은 Kubernetes 환경에서 Ceph를 배포, 운영, 관리하기 위한 오픈 소스 오케스트레이션 프레임워크입니다. Rook을 사용하면 복잡한 Ceph 클러스터 구축 과정을 단순화하고, Kubernetes 환경에 특화된 자동화된 관리 기능을 활용할 수 있습니다. [**Rook 공식 홈페이지**](https://rook.io/docs/rook/latest-release/Getting-Started/quickstart/#tldr)에서 제공하는 YAML 파일을 배포하는 것만으로 손쉽게 설치가 가능합니다.

### 2-1-4 활용도

CMP 환경에서 Ceph의 활용 가치가 높은 부분은 현재 NFS와 같은 공유 볼륨을 사용하는 로그 파일 수집 영역입니다. CephFS를 사용하여 공유 볼륨을 구성하면 데이터를 워커 노드 전체에 분산 저장하고 공유할 수 있으며, CRUSH 알고리즘을 통해 데이터가 분산 및 복제되어 저장되므로 특정 노드에 장애가 발생하더라도 데이터 손실 위험을 최소화할 수 있습니다. 이를 통해 로그 데이터의 안정적인 수집 및 보관이 가능하며, 시스템 전체의 가용성을 향상시킬 수 있습니다.

## 2-2 Longhorn

![image.png](/commons/storage/2.png)

### 2-2-1 아키텍처

- Longhorn은 kubernetes 환경에서 클라우드 네이티브 블록 스토리지를 제공하는 경량화된 분산 스토리지 시스템입니다. 모든 볼륨은 전용 스토리지 컨트롤러(Longhorn Engine)와 복제본(Replica)로 구성되며, 데이터는 여러 노드에 걸쳐 자동으로 복제됩니다. 또 간편한 UI를 제공하는 특징이 있습니다.
- 구성 요소:
  - **Longhorn Manager**: Longhorn 시스템의 컨트롤 플레인 역할을 수행합니다. 볼륨 생성, 스냅샷, 백업 등의 작업을 관리하며 DaemonSet으로 실행되어, 모든 노드에 배치됩니다. Longhorn UI를 제공합니다.
  - **Longhorn Engine**: 각 볼륨에 대해 전용 스토리지 컨트롤러 역할을 수행하며, 데이터 복제를 관리합니다. 볼륨 마운트된 Pod당 하나씩 생성되어 관리합니다.
  - **Replicas**: 볼륨 데이터의 복제본을 저장하는 역할을 수행합니다. 각 복제본은 서로 다른 노드에 분산되어 저장되어 데이터 손실에 대비합니다.

### 2-2-2 확장성

Longhorn은 새로운 워커 노드를 추가하거나 기존 노드에 디스크를 추가하면 이를 자동으로 감지하여 복제본을 생성합니다. Longhorn Manager는 kubernetes API와 통신하여 새로운 디스크를 관리합니다.

### 2-2-3 편의성

kubectl, helm 를 사용하여 간편하게 설치가 가능하며, 직관적인 GUI를 통해 관리할 수 있습니다. 스냅샷, 백업 기능을 지원하여 데이터 복호화 복구를 간소화시켜줍니다. **공유 볼륨보다는 단일 볼륨에 권장되는 솔루션**입니다.

### 2-2-4 활용도

CMP 환경에서 DB 관리를 위해 사용하기 적합합니다. 복제본은 설정이 가능하며 다음의 그림과 같이 세 개의 노드에 분산되어 저장이 가능합니다. Longhorn Engine은 복제본 간의 데이터 동기화를 관리하는데. Pod가 데이터를 쓰면 이를 모든 복제본으로 전송하여 동기화합니다. 특정 노드가 장애가 발생하면 Longhorn은 자동으로 새로운 복제본을 생성하여 데이터를 다시 동기화합니다.

![image.png](/commons/storage/3.png)

공유 볼륨으로도 사용은 가능합니다. 모든 노드에 NFS Client를 설치 한뒤, NFS 서버를 Pod로 올리고 볼륨을 마운트 해줍니다. 그리고 사용하려는 Pod에 NFS 서버를 마운트해주면 됩니다.

![image.png](/commons/storage/4.png)

## 2-3 OpenEBS

![image.png](/commons/storage/5.png)

### 2-3-1 아키텍처

- OpenEBS는 kubernetes 네이티브 스토리지 솔루션으로, 다양한 스토리지 엔진을 통해 애플리케이션에 적합한 옵션을 제공합니다.
- 구성 요소:
  - **Control Plane**: OpenEBS의 스토리지 엔진을 관리하고, 볼륨의 생성, 삭제 스냅샷, 복원 등을 수행합니다.
  - **Data Plane(storage engine)**: OpenEBS의 다양한 스토리지 엔진(Jiva, Stor, Mayastor, LocalPV)을 통해 데이터를 저장하고 복제합니다.
    - Jiva: 간단한 개발 및 테스트 환경에 적합한 경량 엔진
    - cStor: 고가용성과 데이터 복제를 지원하며, 프로덕션 환경에 적합
    - Mayastor: NVMe 기반 고성능 스토리지 엔진
    - LocalPV: 로컬 디스크를 직접 사용하여 높은 성능 제공
  - **CSI Driver**: kubernetes와 OpenEBS 간의 통신을 담당하며, 볼륨 생성 및 관리 작업을 수행합니다.

### 2-3-2 확장성

OpenEBS는 새로운 워커 노드를 추가하거나 디스크를 추가하면 이를 자동으로 감지합니다. NDM(Node Device Manager)은 노드에 연결된 디스크를 감지하고 BlockDevice 리소스로 등록합니다. 이는 `kubectl get bd -n openebs` 와 같은 명령어를 통해 추가된 디스크를 확인할 수 있습니다. 애플리케이션 요구사항에 따라 다양한 스토리지 엔진(cStor, Mayastor 등)을 선택할 수 있습니다.

### 2-3-3 편의성

OpenEBS는 Helm 차트 또는 kubectl을 통해 간편하게 설치할 수 있습니다. OpenEBS Operator는 스토리지 클래스를 정의하고, 사용자는 PVC(Persistent Volume Claim)를 통해 원하는 스토리지를 요청할 수 있습니다. 설치 시 단일 볼륨 또는 공유 볼륨에 따라 적합한 스토리지 엔진을 선택할 수 있습니다. OpenEBS는 스냅샷, 백업, 복원 기능을 제공하여 데이터 보호 및 복구를 간소화합니다.

### 2-3-4 활용도

OpenEBS는 제공되는 스토리지 엔진에 따라 단일 볼륨 및 공유 볼륨 모두에 적합합니다. NVMe SSD를 지원하는 Mayastor와 같은 스토리지 엔진을 통해 고성능 스토리지를 제공하며, cStor는 데이터베이스와 같은 스테이트풀 애플리케이션에 적합합니다.

# 3. Ceph, OpenEBS, Longhorn 비교

| **특징** | **Ceph** | **OpenEBS** | **Longhorn** |
| --- | --- | --- | --- |
| **스토리지 유형** | 블록, 파일, 객체 | 블록, 파일 | 블록 |
| 확장성 | 매우 높음 | 높음 | 높음 |
| 복잡도 | 높음 | 중간 | 낮음 |
| kubernetes 통합 | Rook 사용 | 네이티브 지원 | CSI 드라이버 지원 |
| 성능 | 높음 | 높음 | 높음 |
| UI 제공 | 제한적 | 제한적 | GUI 제공 |
| 스냅샷 가능 | 지원 | 지원 | 지원 |
| 편의성 | 복잡한 설정 | 상대적으로 간단 | 설치 및 관리 간단 |
| 설치 지원 | helm, kubectl | helm, kubectl | helm, kubectl |
| 디스크 확장 가능성 | 높음 | 높음 | 높음 |
