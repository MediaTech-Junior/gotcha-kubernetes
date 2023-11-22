# 5. 앱 스케일링

## Deployment의 replicas 개수 변경하기

Deployment의 replicas 개수를 수동으로 늘리거나 줄일 수 있습니다.
늘리면 그만큼 Pod가 생성되고, 줄이면 일부 Pod가 삭제됩니다.

변경하는 방법은 아래 명령을 입력하면 됩니다. 정말 편리한데요

```bash
kubectl scale {DEPLOYMENT_NAME} --replicas={NEW_REPLICAS}
```

# 6. 앱 업데이트

앱을 무중단으로 배포하는건 서비스 품질을 위해 중요한데요. 무중단 배포 방법중 하나로 Rolling Update가 있습니다.
이 방법은...

1. 일부 파드를 다운시키고, 업데이트합니다.
2. 그동안 트래픽은 남은 파드들이 모두 받습니다.
3. 업데이트가 끝나면, 다른 파드들을 업데이트합니다.

Service는 업데이트중인 파드에는 트래픽을 전달하지 않습니다. 업데이트가 끝났어도, 아직 실행안된 파드에게도 트래픽을 전달히자않죠. 똑똑하지 않나요?

쿠버네티스는 명령어 하나로 Rolling Update를 할 수 있습니다. 롤백도 지원합니다.