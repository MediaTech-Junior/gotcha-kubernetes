## Linux 컨테이너
> 컨테이너: 실행에 필요한 모든 파일을 포함한 전체 실행 환경(runtime)에서 애플리케이션을 패키징하고 격리할 수 있는 기술

### Container vs VM
> `K < M < G`
<img width="300" src="https://www.redhat.com/rhdc/managed-files/styles/wysiwyg_full_width/private/virtualization-vs-containers_transparent.png?itok=kpBl0OG5">

- 컨테이너: MB 단위. 특정 작업을 수행하는 단일 기능(e.g. 마이크로서비스)만 컨테이너에 패키징함(경량화). OS 공유
- VM: GB 단위. 자체 OS를 포함하여 리소스의 기능을 동시에 수행할 수 있음
