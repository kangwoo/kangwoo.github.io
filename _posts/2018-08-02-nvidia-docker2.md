---
title:  "Kubernetes 에서 NVIDIA GPU container 사용하기"
classes: wide
date: 2018-08-15T15:13:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, nvidia, gpu]
---

기본적인 docker를 이용하면, GPU 자원을 사용할 수 없습니다. 쿠버네티스 환경에서 NVIDIA GPU를 사용하기 위해서는 `nvidia-docker`를 이용해야 합니다.


# nvidia-docker 설치하기
먼저 NVIDIA Driver가 설치되어 있어야 합니다.
```bash
nvidia-smi
```

## repository 추가
드라이버가 설치되어 있다면, nvidia-docker 설치를 위한 repository를 추가해 줍니다.
### Debian-based distributions
```bash
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
```

### RHEL-based distributions
```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | \
  sudo tee /etc/yum.repos.d/nvidia-docker.repo
```

## nvidia-docker 설치
### Debian-based distributions
```bash
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd
```

### RHEL-based distributions
```bash
sudo yum install -y nvidia-docker2
sudo pkill -SIGHUP dockerd
```

## nvidia-docker 확인
설치가 끌나면, 아래처럼 `--runtime=nvidia` 플래그를 이용해서 GPU를 사용할 수 있습니다. 
아래 명령어를 실행하면 GPU를 사용할 수 있는 상태인지 확인할 수 있습니다.
```bash
$ docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
Mon Jul 15 12:45:56 2019
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.67       Driver Version: 418.67       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla M40           Off  | 00000000:02:00.0 Off |                    0 |
| N/A   35C    P0    63W / 250W |    115MiB / 11448MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  Tesla M40           Off  | 00000000:82:00.0 Off |                    0 |
| N/A   26C    P8    16W / 250W |      0MiB / 11448MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   2  Tesla M40           Off  | 00000000:85:00.0 Off |                    0 |
| N/A   24C    P8    16W / 250W |      0MiB / 11448MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   3  Tesla M40           Off  | 00000000:86:00.0 Off |                    0 |
| N/A   24C    P8    16W / 250W |     11MiB / 11448MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0     21141      C   python                                       104MiB |
+-----------------------------------------------------------------------------+

```

<br/>

# 쿠버네티스에서 nvidia-docker 사용하기
쿠버네티스 환경에서 `nvidia-docker`를 사용하려면, `docker`의 기본 런타임(runtime)을 변경하고, NVIDIA 디바이스 플러그인을 설치해줘야 합니다.

## docker 기본 runtime 변경
정상적으로 `nvidia-docker2`가 설치되었다면, `/etc/docker/daemon.json` 파일이 생성되었을 것입니다.
해당 파일을 열여서 `"default-runtime": "nvidia"`을 추가해주면 됩니다.
```json
{
  "default-runtime": "nvidia", 
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
```
파일을 수정한 후, docker daemon을 재시작 하여야합니다.
```bash
sudo systemctl restart docker
```

## kubernetes-device-plugin 설치
`kubectl`을 이용해서 `nvidia-device-plugin`을 설치해 줍니다.
```bash
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.11/nvidia-device-plugin.yml

```
참고) nvidia-device-plugin.yml
```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nvidia-device-plugin-daemonset
  namespace: kube-system
spec:
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      # Mark this pod as a critical add-on; when enabled, the critical add-on scheduler
      # reserves resources for critical add-on pods so that they can be rescheduled after
      # a failure.  This annotation works in tandem with the toleration below.
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
      labels:
        name: nvidia-device-plugin-ds
    spec:
      tolerations:
      # Allow this pod to be rescheduled while the node is in "critical add-ons only" mode.
      # This, along with the annotation above marks this pod as a critical add-on.
      - key: CriticalAddonsOnly
        operator: Exists
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
      containers:
      - image: nvidia/k8s-device-plugin:1.11
        name: nvidia-device-plugin-ctr
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
        volumeMounts:
          - name: device-plugin
            mountPath: /var/lib/kubelet/device-plugins
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
```

## 참고 : Feature Gates
 `nvidia-device-plugin`을 사용하려면, 해당 노드 kubelet의 DevicePlugins이 "true"로 설정되어 있어야한다.
쿠버네티스 1.10 이상에서는 기본값이 "true"이기 때문에 별도의 설정이 필요없으나, 
혹시라도 1.8이나 1.9를 사용한다면, `KUBELET_EXTRA_ARGS=--feature-gates=DevicePlugins=true` 로 값을 설정해주어야한다.
 - https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/

## GPU 자원(Resource) 이용
GPU 자원을 이용하려면, 리소스 요구사항에 `nvidia.com/gpu`을 사용하면 됩니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: cuda-container
      image: nvidia/cuda:9.0-devel
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPUs
```

# 참고 문서
- <https://github.com/NVIDIA/nvidia-docker>
- <https://github.com/NVIDIA/k8s-device-plugin>
- <https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/>
