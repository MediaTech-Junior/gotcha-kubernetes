# 2. 앱 배포하기

## Deployment

**앱 배포 방법을 추상화한 것**입니다.

쿠버네티스에서는 컨트롤러가 시스템의 상태를 관찰하고, 복구합니다. 
Deployment는 **컨트롤러에게 "이 앱은 이런 상태여야해!" 라고 알려주는 역할**이라고 볼 수 있습니다.

예를 들어서 Deployment에 최소 Pod수를 3으로 명시하면, 그 앱은 최소 3개의 Pod로 구동됨을 보장합니다.
운영 도중에 앱이 하나 다운되어서 Pod가 2개가 되어도, 컨트롤러가 이를 감지해서 Pod를 다시 띄웁니다.

다른 예시로 앱을 업데이트하려면, Deployment를 수정해주기만 하면 됩니다. `kubectl set`을 사용해서 이미지를 변경할 수 있습니다.

재미있는 점은 이미지를 바꾸면 모든 파드에 즉시 적용되는 것이 아니라, **자동으로 롤링 업데이트를 진행해준다는 점입니다.** 심지어 롤백도 지원합니다.

## 앱 배포하고 확인해보기

```bash
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```
이 명령을 실행하고 `kubectl get deployment`로 확인해보면, deployment가 생성된 것을 확인할 수 있습니다.

앱이 실행되고 있기는 한데... 접근해보려면 운영 환경에서는 서비스를 만들어야합니다. 서비스까지 만들기는 좀 복잡하니, 잠깐 테스트 용도로 쓰기 좋은 `kubectl proxy`를 사용해볼 수 있습니다.

`kubectl proxy`로 API 프록시 서버를 띄우고, 아래 명령으로 파드가 잘 떠있는지 확인해보세요.

```bash
curl http://localhost:8001/api/v1/namespaces/{NAMESPACE}/pods/{POD_NAME}:8080/proxy
```

> ### kubectl proxy가 뭐지?
> 
> 쿠버네티스의 API 서버는 사용하려면 인증이 필요합니다. (`~/.kube/config`에 있는 TLS 키를 사용)
> 
> `kubectl`을 사용하면 인증을 자동으로 수행해주지만, curl을 사용해서 api를 쓰려면 직접 인증해야하는데요.
> `kubectl proxy`를 사용하면 인증을 대신 수행해주는 프록시 서버를 foreground에서 구동합니다.
> 
> API서버는 파드를 생성할 때, 자동으로 그 파드 정보를 확인할 수 있는 엔드포인트도 만들어주는데요. `kubectl proxy`를 사용해서 원하는 파드에 접근해볼 수도 있습니다.
> 이 예제에서도 이걸 위해서 `kubectl proxy`를 쓴 것이구요!
> 
> `kubectl proxy`를 실행하면, 기본 8001번 포트로 프록시 서버를 실행합니다. 여기로 curl명령을 보내면 api서버로 프록싱하면서 인증까지 자동으로 수행해줍니다.
> 
> 하지만 보안상의 이유로 **테스트 용도, 로컬에서만 사용**하는 것을 권장합니다. 외부로 앱을 노출하려면 서비스를 사용하세요~