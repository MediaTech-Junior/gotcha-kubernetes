# Horizontal Pod Autoscaling

## 리소스 메트릭

스케일링 기준으로 리소스 사용량을 지정할 수 있습니다. 리소스의 최대 가용량을 k8s가 알고 있기 때문에, 리소스 사용률을 계산할 수 있습니다.

예를 들어 CPU 사용량 60%를 기준으로 스케일링하게 하고 싶다면...

```yaml
type: Resource
resource:
  name: cpu
  target:
    type: Utilization
    averageUtilization: 60
```

리소스는 사용률을 스케일링 기준으로 정해 사용할 수 있습니다. 다만 커스텀 메트릭의 경우에는 사용률을 지원하지 않습니다.

## 컨테이너 리소스 메트릭

리소스 메트릭은 모든 컨테이너 리소스를 합산/평균내어 계산됩니다. 그래서 만약 하나의 컨테이너가 많은 리소스를 사용하더라도, 전체 컨테이너 리소스 사용량 기준에서는 높지 않을 수 있죠.

그래서 스케일링이 되지 않을 수 있습니다. 컨테이너 리소스 메트릭은 컨테이너에 HPA 사양을 지정합니다.

```yaml
type: ContainerResource
containerResource:
  name: cpu
  container: application
  target:
    type: Utilization
    averageUtilization: 60
```

`application` 컨테이너에 CPU 사용률 60% 정책을 적용한 예시입니다.

## Scaling 정책

정책을 설정해 scale down 속도를 조절할 수 있습니다.

```yaml
behavior:
  scaleDown:
    policies:
    - type: Pods
      value: 4
      periodSeconds: 60
    - type: Percent
      value: 10
      periodSeconds: 60
```

두 가지 방법으로(파드 수, 비율) 정책을 설정할 수 있습니다. 정책이 이렇게 여러 개인 경우에는 개수가 더 많은 정책이 사용됩니다.

예를 들어서 replica가 20개라면? -> Pods는 4개, Percent는 2개이므로 Pods 정책이 선택되어 4개가 down됩니다. `periodSeconds`가 60초이므로 60초 주기로 반복합니다.

scale down을 해야하는 상황이 오면 이 정책에 따라 점진적으로 파드를 줄여나갑니다.

## 안정화 윈도우

파드의 메트릭은 계속 변합니다. 그래서 스케일링이 계속 발생할텐데요. 기준 메트릭을 잘못 지정하면, 매번 스케일링이 일어날 수도 있습니다.
파드를 삭제하고 생성하고... 불필요한 리소스를 낭비합니다.

그래서 안정화 윈도우는 일정 시간을 두고, 가장 높은 replica를 기록해서 그 값으로 스케일링합니다.

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300
```

이렇게 설정하면 5분동안 메트릭을 기록하면서, 가장 높은 replica값으로 스케일링합니다. (보수적으로 스케일링)

