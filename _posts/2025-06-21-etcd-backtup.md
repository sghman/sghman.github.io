---
title: Etcd 백업 및 복구 절차
author: gyuhwan
date: 2025-06-21 17:20:00 +0800
categories: [DevOps, k8s, etcd]
tags: [DevOps]
---
# Etcd 백업 및 복구 절차

# 1. 필요한 도구 설치

## 1.1 etcdct 설치

```bash
# 1. etcdctl 다운로드
[root@k8s-master local]# curl -L https://github.com/etcd-io/etcd/releases/download/v3.5.17/etcd-v3.5.17-linux-amd64.tar.gz -o /tmp/etcd.tar.gz

# 2. 다운드한 tar.gz 파일을 압축해제
[root@k8s-master local]# tar xzvf /tmp/etcd.tar.gz -C /tmp

# 3. 실행 파일을 /usr/local/bin 으로 이동
[root@k8s-master local]# mv /tmp/etcd-v3.5.17-linux-amd64/etcd* /usr/local/bin/
```

## 1.2 etcd endpoint 확인

etcd의 클라이언트 URL을 확인하기 위해 아래의 명령어를 실행합니다.

```bash
[root@k8s-master ~]# ps aux | grep etcd | awk -F '--listen-client-urls=' '{print $2}' | awk '{print $1}'
https://127.0.0.1:2379,https://192.168.56.30:2379
```

## 1.3 etcdctl 인증키 등록

### 1.3.1 etcdctl alias 등록(선택 1)

etcdctl 명령어를 간편하게 사용하기 위해 alias를 등록합니다.

```bash
[root@k8s-master ~]# alias etcdctl='ETCDCTL_API=3 etcdctl --endpoints=https://192.168.56.30:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key'
```

이 alias를 등록하면, 매번 ETCDCTL_API=3 및 인증서 경로를 입력할 필요 없이 간편하게 etcdctl 명령어를 사용할 수 있습니다.

### 1.3.2 환경변수 등록(선택 2)

```bash
[root@k8s-master ~]# export ETCDCTL_API=3
[root@k8s-master ~]# export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
[root@k8s-master ~]# export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
[root@k8s-master ~]# export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
[root@k8s-master ~]# export ETCDCTL_ENDPOINTS=https://192.168.56.30:2379
```

이 환경 변수들은 etcdctl 명령어를 사용하기 위해 필요한 인증서, 인증키, Endpoint 정보를 포함합니다.

## 1.4 etcd 상태 확인

```bash
# endpoint health check
[root@k8s-master ~]# etcdctl endpoint health
https://192.168.56.30:2379 is healthy: successfully committed proposal: took = 6.859308ms

# etcd 세부 상태 확인
[root@k8s-master ~]# etcdctl endpoint status --write-out=table
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|          ENDPOINT          |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://192.168.56.30:2379 | 7a3a42e9f89c112c |   3.5.7 |  6.9 MB |      true |      false |         6 |     203898 |             203898 |        |
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
```

# 2. etcd 백업 수행

etcdctl 명령어를 사용하여 스냅샷을 생성합니다.

```bash
[root@k8s-master ~]# etcdctl snapshot save /backup/etcd-backup-$(date +%F).db
...
Snapshot saved at /backup/etcd-backup-2025-01-08.db
```

# 3. etcd 복구 절차

## 3.1 복구 방법 1 :  스냅샷 복원 후 etcd.yaml 수정

### 3.1.1 스냅샷 복원

```bash
# 스냅샷 복원
[root@k8s-master ~]# etcdutl snapshot restore /backup/etcd-backup-2025-01-08.db --data-dir /var/lib/etcd-backup-$(date +%F)
...

# 복원된 디렉터리 확인
[root@k8s-master ~]# ls /var/lib/etcd-backup-2025-01-08/
member
```

### 3.1.2 etcd.yaml 수정

/etc/kubernetes/manifests/etcd.yaml 파일을 열어 hostPath를 스냅샷 복원했던 디렉터리로 변경합니다. etcd는 정적 파드로 yaml 파일이 수정되면 자동으로 설정이 적용됩니다.

```bash
[root@k8s-master ~]# vi /etc/kubernetes/manifests/etcd.yaml

apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.56.30:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
....
  - hostPath:
      path: /var/lib/etcd-backup-2025-01-08     <<--------- 수정
      type: DirectoryOrCreate

```

## 3.2 복구 방법 2 : 스냅샷 복원 후 member 파일 교체

### 3.1.2 스냅샷 복원

```bash
# 스냅샷 복원
[root@k8s-master ~]# etcdutl snapshot restore /backup/etcd-backup-2025-01-08.db --data-dir /var/lib/etcd-backup-$(date +%F)
...

# 복원된 디렉터리 확인
[root@k8s-master ~]# ls /var/lib/etcd-backup-2025-01-08/
member
```

### 3.1.3 member 파일 교체

```bash
[root@k8s-master ~]# mv /var/lib/etcd /var/lib/etcd.bak
[root@k8s-master ~]# mkdir /var/lib/etcd
[root@k8s-master ~]# mv /var/lib/etcd-backup-2025-01-08/member /var/lib/etcd/
```

### 3.1.4 etcd 재시작

```cpp
# etcd container id 확인
[root@k8s-master ~]# crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD
e78fa7ae1adbb       86b6af7dd652c       24 hours ago        Running             etcd                        0                   1af5613b2e3eb       etcd-k8s-master

# etcd 재시작(stop시 자동으로 재실행됨)
[root@k8s-master ~]# crictl --runtime-endpoint unix:///run/containerd/containerd.sock stop e78fa7ae1adbb
```

## 3.2 복구 상태 확인

etcd가 정상적으로 동작하는지 확인합니다.

```bash
[root@k8s-master ~]# etcdctl endpoint health
https://192.168.56.30:2379 is healthy: successfully committed proposal: took = 6.859308ms
```

# 4. 참고 문서

[etcd 설치](https://etcd.io/docs/v3.5/install/)

[https://etcd.io/docs/v3.6/op-guide/recovery/#restoring-a-cluster](https://etcd.io/docs/v3.6/op-guide/recovery/#restoring-a-cluster)

[https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#restoring-an-etcd-cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#restoring-an-etcd-cluster)
