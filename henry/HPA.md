# Horizontal Pod Autoscaling

> 쿠버네티스에서는 작업량에 비례해 자동으로 Pod의 개수를 조절하는
> Horizontal Pod Autoscaling(HPA)이 가능하다.
> 이것은 작업량이 많아지면 하나의 Pod에 더 많은 리소스(CPU나 메모리)를 할당하는 vertical scaling과는 다른 개념이다.
>
> `HorizontalPodAutoscaler`는 미리 설정된 메트릭에 맞춰 스케일링을 진행한다.
## How does a HorizontalPodAutoscaler work?
쿠버네티스는 horizontal pod autoscaling을 간헐적으로 실행되는 control loop로 구현했다. (default interval: 15초)

각 실행마다 컨트롤러 매니저는 `scaleTargetRef`에 정의된 타겟(e.g. `Deployments`, `StatefulSet`)을 찾고, 타겟의 `.spec.selector` label과 일치하는 pod들을 선택한다.
그리고 나서 per-pod 메트릭은 resource metrics API를 통해 나머지 다른 메트릭은 custom metrics API를 통해 수집한다.

보통 `HorizontalPodAutoscaler`를 이용할 때 메트릭 데이터를 받아올 수 있도록 `Metrics Server`를 이용한다.

### Algorithm details
`원하는 Replicas = ceil[현재 Replicas * ( 현재 metric / 원하는 metric )]`

### 메트릭 종류
* Resource metric: CPU, 메모리 사용량 등 pod, node에 미리 정의되어 있는 metric을 의미.
* Custom metric: 유저가 정의한 metric을 의미하며 절댓값을 이용한 Resource Metric과 유사하게 작동.

  e.g. `istio`와 `prometheus`등을 이용해 RPS를 측정할 수 있고 이를 HPA를 위한 custom metric으로 사용할 수 있다.
* External metric: 쿠버네티스와 관련 없는 단일 metric이며, 비교가 이루어지기 전에 Pod 개수로 값을 나눌지 여부를 선택할 수 있음.

## Autoscaling during rolling update
`Deployment`에 오토스케일링 설정을 하면 각 `Deployment`마다 `HorizontalPodAutoscaler`가 생성된다.
`HorizontalPodAutoscaler`는 `Deployment`의 `replicas`필드를 관리하며
디플로이먼트 컨트롤러는 기반이 되는 `ReplicaSets`의 `replicas`를 설정하기 때문에 롤링 업데이트 도중과 이후 pod 개수가 자동으로 조절된다.

`StatefulSet`의 경우에는 `ReplicaSet`과 비슷한 중간 리소스가 없고 `StatefulSet`이 직접 파드 개수를 관리한다.

## Support for resource metrics
오토스케일링 타겟이 되는 파드의 리소스 사용량을 기반으로 스케일링이 가능하다.
```yaml
type: Resource
resource:
  name: cpu
  target:
    type: Utilization
    averageUtilization: 60
```
위의 예시의 경우 파드의 평균 cpu utilization이 60%가 되도록 스케일링이 발생할 것이다.

cf. Utilization: 현재 리소스 사용량 / 파드에 할당된 리소스

### Container resource metrics
> 파드 내 모든 컨테이너의 리소스 사용량이 합해져서 계산되기 때문에 하나의 컨테이너가 리소스를 매우 많이 사용하고 있어도
> 전체 파드의 리소스 사용량이 acceptable한 수준이어서 스케일링이 발생하지 않는 경우도 있다.
컨테이너 별 리소스 메트릭도 제공한다.
예를 들면, 하나의 파드에 웹 애플리케이션과 로깅 사이드카가 존재한다면 사이드카 리소스 사용량은 무시하고 웹 애플리케이션의 리소스 메트릭 기준으로 스케일링도 가능하다.

```yaml
type: ContainerResource
containerResource:
  name: cpu
  container: application
  target:
    type: Utilization
    averageUtilization: 60
```
위의 예시의 경우 모든 파드의 `application` 컨테이너 평균 cpu utilization이 60%가 되도록 스케일링이 발생할 것이다.

## Stability of workload scale
메트릭 변동이 자주 발생하면 HPA에 의해 pod 개수도 변동이 심할 수 있다. 이런 현상을 `thrashing`이나 `flapping`이라고 한다.

## Configurable scaling behavior
`behavior`필드를 이용해 `scaleUp`, `scaleDown` 전략을 설정할 수 있다.

### Scaling policies
여러 스케일링 정책을 설정할 수 있는데 그 중에서 가장 많은 변화를 허용하는 정책이 디폴트(`selectPolicy: Max`)로 적용된다.

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
위의 예시의 첫 번째 policy의 경우 1분에 최대 4개의 파드 scale down을 허용하고,
두 번째 policy의 경우 1분에 최대 현재 replicas의 10%까지 scale down을 허용한다.

cf. 현재 replicas의 개수가 75개라면 첫 번째 policy가 아닌 더 많은 변화를 허용하는 두 번째 policy가 적용되는데 10%인 7.5개가 아니라 올림해 8개의 변화를 허용한다.

### Stabilization window
`flapping`을 방지하기 위해 stabilization window등을 설정할 수 있다.
```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300
```

위의 예시의 경우 메트릭이 변해 scale down이 필요한 상황일 때, 이전 5분 동안의 replicas의 개수를 파악해 그 중 가장 높은 값으로 스케일링 된다.

### Disable scaling
```yaml
behavior:
  scaleDown:
    selectPolicy: Disabled
```
위의 예시의 경우 scale down이 일어나지 않는다.

## Migrating Deployments and StatefulSet to horizontal autoscaling
HPA가 활성화 되어있을 때, `Deployment`와 `StatefulSet`의 `spec.replicas`값을 manifest에서 지우는 것이 권장된다.

만약 그렇지 않으면, 오브젝트의 변경사항이 반영될 때마다 (e.g. `kubectl apply -f deployment.yaml`) 파드 개수가 `spec.replicas`의 값으로 스케일링 된다.
