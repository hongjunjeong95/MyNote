# Datadog Agent Install With Helm Chart

## Git Repository

[![img](https://github.githubassets.com/favicon.ico)https://github.com/The-DataHunt/helm-dh-platform](https://github.com/The-DataHunt/helm-dh-platform) - Github 계정 연결 

## 기본 설치 / 업그레이드

1. Datadog Directory 접속

2. values.yaml 설정 (기본 - 로그, APM 활성화)

   1) 로그 수집 활성화

      ```
      logs:    # datadog.logs.enabled -- Enables this to activate Datadog Agent log collection    ## ref: https://docs.datadoghq.com/agent/basic_agent_usage/kubernetes/#log-collection-setup    enabled: true    # datadog.logs.containerCollectAll -- Enable this to allow log collection for all containers    ## ref: https://docs.datadoghq.com/agent/basic_agent_usage/kubernetes/#log-collection-setup    containerCollectAll: true
      ```

   2) APM 활성화

      ```
      apm:    # datadog.apm.socketEnabled -- Enable APM over Socket (Unix Socket or windows named pipe)    ## ref: https://docs.datadoghq.com/agent/kubernetes/apm/    socketEnabled: true    # datadog.apm.portEnabled -- Enable APM over TCP communication (port 8126 by default)    ## ref: https://docs.datadoghq.com/agent/kubernetes/apm/    portEnabled: true
      ```

3. install.sh / upgrade.sh 실행 ([API_KEY](https://app.datadoghq.com/organization-settings/api-keys?_gl=1*1p0upnw*_gcl_aw*R0NMLjE2ODE4MDQ1OTkuQ2p3S0NBandfX2loQmhBREVpd0FYRWF6SmhYNVRkZ3RRWUxVM1VzLVFwb1laWlJhSERad3k0WndwREZ2ejhSejlXbDVqNVNacnQydHRCb0NrdmtRQXZEX0J3RQ..*_ga*MjA0ODMzMTA5MS4xNjgyNDcxOTUy*_ga_KN80RDFSQK*MTY4Mjk5MzA0NC4xMS4wLjE2ODI5OTMwNDUuNTkuMC4w*_fplc*JTJCJTJGdUhTUktwcUpvcDBkOFdoWDVNN0VadUNocTYlMkJ1RlAwdXlmY3RxNFZrM1dqRXhTZTJwbnhxaHdYSSUyRmw4bnN2YkZKSlJaU0lSTEtZNTdlb2t5Vm1QVmJLQ1NaRmx4OVFvWXgxJTJGS2g5OWR3UzdiT2h3c1ZzTzFKZmhPbXRjQSUzRCUzRA..), [APP_KEY](https://app.datadoghq.com/organization-settings/application-keys))

   <img width="873" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/260dcb86-5a6e-4134-9690-a3c4afe1168d">

## 설치 후 Pod 리스트

<img width="1063" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/71db6057-cbf7-4097-8d64-b5c4689f9ecc">

## 설치 후 Service 리스트

<img width="1646" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/b301516b-91c3-469f-bcda-e8a1933fc2b5">

#### reference

[![img](https://docs.datadoghq.com/favicon.ico)Install the Datadog Agent on Kubernetes](https://docs.datadoghq.com/containers/kubernetes/installation/?tab=helm) 

 