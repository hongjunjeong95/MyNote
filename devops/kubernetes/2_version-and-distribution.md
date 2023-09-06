# Version and Distribution

## 로컬 쿠버네티스 배포판

<img width="1269" alt="image" src="https://user-images.githubusercontent.com/33750210/232375508-43cb3483-78b9-4cd6-a6d5-11a9e8a4e83b.png" style="zoom: 50%;"  >


<p style="text-align: center;">2021년 5월 자료</p>

- 단일 노드에 빠르게 쿠버네티스 구성 및 테스트 가능
- 일부 기능 제한
  - 클라우드 플랫폼에서만 사용 가능한 기능
    - ALB, NLB, EBS in AWS
  - 여러 노드 환경에서 사용 가능한 리소스
    - DaemonSet, Affinity / Taint / Toleration



## 쿠버네티스 버전 선택

21년 12월 기준 최신 쿠버네티스 버전은 1.22 / AWS EKS 최신 버전은 1.21

운영환경에서 쿠버네티스를 관리한다면

- Major 버전 업데이트의 ChangeLog와 Breaking Changes 꼼꼼히 검토
-  API Resources Deprecation Schedule 검토

<img width="1408" alt="image" src="https://user-images.githubusercontent.com/33750210/232377908-73a653b4-1e54-47c8-b49b-ffec17bc2367.png">

<strong style="color:#eb4444">버전에 따라 지원하는 기능 및 API 리소스 스펙 차이가 있으니 주의할 것</strong>
