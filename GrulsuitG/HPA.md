# Horizontal Pod Autoscaling

쿠버네티스에서는 요구되는 리소스에 맞춰 자동으로 스케일링 할 수 있다.

- 수평적 확장: 파드의 수를 늘리는 것
- 수직적 확장: 리소스를 더 많이 할당하는 것

요구량이 줄어들면 자동으로 축소되며, HPA 는 스케일될 수 있는 오브젝트에만 적용 가능하다.

## 작동방식

HPA 는 주기적으로 실행되는 제어루프로 구현됩니다.  
(주기 설정 방법 : `--horizontal-pod-autoscaler-sync-period`, default : 15s)

컨트롤러 매니저는 HPA 정의에 지정된 메트릭에 대한 리소스 사용률을 조회한다.

- 파드 단위 리소스 메트릭  
  % 단위, raw value 모두 사용 가능하며 스케일하는데 사용되는 비율을 생성한다.
  만약 적절하게 리소스 요청이 되지 않았다면 autoscaler 는 동작하지 않는다.

- 파드 단위 사용자 정의 메트릭
  raw value 로만 정의될 수 있다.

- 오브젝트 메트릭 및 외부 메트릭
  단일 값을 통해 확장여부를 결정하고, 해당 값을 파드의 개수로 나눌 지 선택할 수 있다.

## 알고리즘

```
desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )]
```

오차범위 설정 : `horizontal-pod-autoscaler-tolerance` (기본값: 0.1)

특정 파드에 메트릭 누락( ex) unready, unhealthy, initializing, etc...)이 발생하면 처리를 미루고, 최종 스케일 량을 조정하는데 사용된다.

이런 누락된 메트릭에 대한 값들은 보수적으로 측정된다

- scale down : 100% 사용한다고 가정
- scale up : 0%를 사용한다고 가정

## 스케일링 정책

```
behavior:
  scaleDown:
    policies:
    - type: Pods #1
      value: 4
      periodSeconds: 60
    - type: Percent #2
      value: 10
      periodSeconds: 60
```

1번 정책: 1분동안 4개를 스케일 다운할 수 있다.  
2번 정책: 1분동안 10%를 스케일 다운할 수 있다.

기본적으로 가장 많은 변경을 허용하는 정책이 선택되기 때문에
레플리카가 40개가 넘어야 2번정책이 선택된다.
