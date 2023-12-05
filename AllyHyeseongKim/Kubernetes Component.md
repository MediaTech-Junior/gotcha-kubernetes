## Kubernetes 컴포넌트

### 클러스터

<img width="400" src="https://kubernetes.io/images/docs/components-of-kubernetes.svg">

- 클러스터: 애플리케이션 컨테이너를 실행하기 위한 노드 머신
    - 클러스터는 control plane, 컴퓨팅 머신 혹은 노드를 포함
- control plane: 쿠버네티스 노드를 제어하는 프로세스의 컬렉션. 테스크를 할당함
- 노드: control plane에서 할당된 요청 테스크를 수행하는 머신
- 서비스: pod에서 네트워크 서비스로 실행 중인 애플리케이션 노출 방식
- 볼륨: pod 컨테이너에 접근할 수 있는 데이터가 포함된 directory
    - `볼륨의 수명 = pod의 수명`. 컨테이너를 재시작해도 데이터가 보존됨
- 네임스페이스: 가상 클러스터. 네임스페이스를 통해 동일한 물리 클러스터 내에 있는 여러 클러스터를 관리함
