---
title:  "Prometheus를 사용해서 NVIDIA GPU 모니터링 하기"
classes: wide
date: 2019-09-08T15:13:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, nvidia, gpu, prometheus]
---

# Node의 GPU 모니터링 하기
`prometheus`를 사용해서 노드들의 정보를 수집하고 있다면, 아마 `node-exporter'를 사용하고 있을 것이다.
`NVIDIA`에서는 `dcgm-exporter`라는 GPU 정보 출력용 이미지를 제공하고 있다.
이 `dcgm-exporter`과 `node-exporter'를 결합하여 사용하면, GPU 정보를 수집할 수 있다.

우선 일반 노드와 GPU 노드를 분리하기 위해서 `taint`와 `label`을 설정해주었다.

```bash
kubectl taint nodes ${node} nvidia.com/gpu=:NoSchedule

kubectl label nodes ${node} "nvidia.com/brand=${label}"
kubectl label nodes ${node} hardware-type=NVIDIAGPU
```
	

# 기존 `node-exporter`에 `dcgm-exporter` 추가하기
`dcgm-exporter`가 GPU 정보를 파일로 남기고, `prometheus`는 그 파일을 읽어서 그 정보를 같이 출력한다.

```bash
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: node-exporter
      app.kubernetes.io/instance: gpu-node-exporter
      app.kubernetes.io/part-of: prometheus
      app.kubernetes.io/managed-by: argo-system
  template:
    metadata:
      labels:
        app.kubernetes.io/name: node-exporter
        app.kubernetes.io/instance: gpu-node-exporter
        app.kubernetes.io/part-of: prometheus
        app.kubernetes.io/managed-by: argo-system
    spec:
      nodeSelector:
        hardware-type: NVIDIAGPU
      containers:
      - args:
        - --path.procfs=/host/proc
        - --path.sysfs=/host/sys
        - "--collector.textfile.directory=/run/prometheus"
        image: prom/node-exporter:v0.18.1
        imagePullPolicy: IfNotPresent
        name: prometheus-node-exporter
        ports:
        - containerPort: 9100
          hostPort: 9100
          name: metrics
          protocol: TCP
        resources:
          limits:
            cpu: 500m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /host/proc
          name: proc
          readOnly: true
        - mountPath: /host/sys
          name: sys
          readOnly: true
        - name: collector-textfiles
          readOnly: true
          mountPath: /run/prometheus
      - image: nvidia/dcgm-exporter:1.4.6
        name: nvidia-dcgm-exporter
        securityContext:
          runAsNonRoot: false
          runAsUser: 0
        volumeMounts:
          - name: collector-textfiles
            mountPath: /run/prometheus
      dnsPolicy: ClusterFirst
      hostNetwork: true
      hostPID: true
      restartPolicy: Always
      serviceAccount: prometheus-node-exporter
      serviceAccountName: prometheus-node-exporter
      terminationGracePeriodSeconds: 30
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
      - effect: NoSchedule
        key: node-role.kubernetes.io/ingress
        operator: Exists
      - effect: NoSchedule
        key: nvidia.com/gpu
        operator: Exists
      volumes:
      - hostPath:
          path: /proc
          type: ""
        name: proc
      - hostPath:
          path: /sys
          type: ""
        name: sys
      - name: collector-textfiles
        emptyDir:
          medium: Memory
      - name: pod-gpu-resources
        hostPath:
          path: /var/lib/kubelet/pod-resources
  updateStrategy:
    type: OnDelete


```


# 참고 문서
- <https://docs.nvidia.com/datacenter/kubernetes/kubernetes-upstream/index.html#kubernetes-gpu-monitoring-agent>
