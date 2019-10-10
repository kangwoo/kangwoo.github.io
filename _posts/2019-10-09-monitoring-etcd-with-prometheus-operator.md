---
title:  "Prometheus Operator로 etcd 모니터링 하기"
classes: wide
date: 2019-10-09T14:27:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, etcd]
---

## etcd 모니터링 하기
etcd 클러스터는 `/metrics` 라는, 프로메테우스가 수집할 수 있는 매트릭 엔드 포인트를 제공한다.
하지만, Secure Etcd 클러스터인 경우에는 해당 엔드 포인트에 접근하기 위해서는 인증서가 필요하다.

(다른 방법으로는 `/metrics` 엔드 포인트를 다른 포트로 분리하여, 인증서 없이 접근할 수도 있다. `--listen-metrics-urls` 옵션을 참고 바란다.)


## 환경
`helm`을 사용해서 `prometheus-operator`를 설치할 것이다. 
그래서 `prometheus-operator`를 설치할때, `etcd`를 모니터링하도록 변경한 설정 파일을 사용한다.
`kubeEtcd.serviceMonitor`의 값들을 변경한다. `scheme`를 https로 변경하고, 인증서 정보를 등록한다.

## values.yaml 수정하기

### kubeEtcd
```bash
## Component scraping etcd
##
kubeEtcd:
...
  serviceMonitor:
    scheme: https
    insecureSkipVerify: false
    serverName: localhost
    caFile: /etc/prometheus/secrets/etcd-client-cert/etcd-ca
    certFile: /etc/prometheus/secrets/etcd-client-cert/etcd-client
    keyFile: /etc/prometheus/secrets/etcd-client-cert/etcd-client-key
...

```

### prometheus
프로메테우스를 기동할때 `etcd-client-cert`란 이름의 `secret`를 `pod`에 마운트하기 위해서, 
`prometheus.secrets`에 `etcd-client-cert`를 추가해 준다.
그리고, `etcd` 스크랩 설정 추가를 위해서, 
`prometheus.additionalScrapeConfigs`의 `kube-etcd` 부분을 활성화 해준다.

```bash
## Deploy a Prometheus instance
##
prometheus:
...
    secrets:
      - "etcd-client-cert"

...

    additionalScrapeConfigs:
      - job_name: kube-etcd
        kubernetes_sd_configs:
          - role: node
        scheme: https
        tls_config:
          ca_file:   /etc/prometheus/secrets/etcd-client-cert/etcd-ca
          cert_file: /etc/prometheus/secrets/etcd-client-cert/etcd-client
          key_file:  /etc/prometheus/secrets/etcd-client-cert/etcd-client-key
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - source_labels: [__address__]
          action: replace
          target_label: __address__
          regex: ([^:;]+):(\d+)
          replacement: ${1}:2379
        - source_labels: [__meta_kubernetes_node_name]
          action: keep
          regex: .*mst.*
        - source_labels: [__meta_kubernetes_node_name]
          action: replace
          target_label: node
          regex: (.*)
          replacement: ${1}
        metric_relabel_configs:
        - regex: (kubernetes_io_hostname|failure_domain_beta_kubernetes_io_region|beta_kubernetes_io_os|beta_kubernetes_io_arch|beta_kubernetes_io_instance_type|failure_domain_beta_kubernetes_io_zone)
          action: labeldrop
          
...

```


## 인증서 복사하기
`etcd` 인증서를, 프로메테스를 설치할, `monitoring` 네임스페이스에 복사한다.

```bash
POD_NAME=$(kubectl get pods -o=jsonpath='{.items[0].metadata.name}' -l component=kube-apiserver -n kube-system)

kubectl create secret generic etcd-client-cert -n monitoring \
  --from-literal=etcd-ca="$(kubectl exec $POD_NAME -n kube-system -- cat /etc/kubernetes/pki/etcd/ca.crt)" \
  --from-literal=etcd-client="$(kubectl exec $POD_NAME -n kube-system -- cat /etc/kubernetes/pki/etcd/healthcheck-client.crt)" \
  --from-literal=etcd-client-key="$(kubectl exec $POD_NAME -n kube-system -- cat /etc/kubernetes/pki/etcd/healthcheck-client.key)"

```

## `helm`으로 `prometheus-operator` 설치하기
`helm` 설치 명령어로, `proemtheus-operator`를 설치한다. 
수정한 설정값을 적용하기 위해서 `--values values.yaml` 옵션을 사용한다.

```bash
helm install stable/prometheus-operator --name mon --namespace monitoring --values values.yaml --tls
```

## 참고 사항
`helm`으로 `prometheus-operator`를 삭제할때, `crd`는 자동으록 삭제되지 않는다. 
아래 명령어로 직접 삭제해야한다.
```bash
kubectl delete --ignore-not-found customresourcedefinitions \
  prometheuses.monitoring.coreos.com \
  servicemonitors.monitoring.coreos.com \
  podmonitors.monitoring.coreos.com \
  alertmanagers.monitoring.coreos.com \
  prometheusrules.monitoring.coreos.com
``` 

## 참고 링크
- https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/monitoring.md
