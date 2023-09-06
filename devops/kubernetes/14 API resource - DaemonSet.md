# API Resource - DaemonSet

## DaemonSet이란?

각 노드마다 꼭 실행되어야 하는 워크로드(로그 수집, 메트릭 수집, 네트워크 구성 등)

- 클러스터 상의 모든 노드에 동일한 파드를 하나씩 생성
- 로그 수집 / 메트릭 수집 / 네트워크 구성 등의 목적으로 많이 사용
  - 로그 수집 : filebeat / fluentbit 등
  - 메트릭 수집 : node-exporter / metricbeat / telegraf 등
  - 네트워크 구성 : kube-proxy / calico 등
- Deployment와 마찬가지로 Label Selector 기반으로 동작
- nodeSelector / Affinity / Toleration 등을 통해 실행되어야 할 노드 목록을 필터링 가능

<img width="978" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/6be1f80a-70bc-419d-aedc-3d8b8db9baa0" style="zoom:50%;" >



