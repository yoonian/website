---
reviewers:
title: 네트워크 정책 선언
content_template: templates/task
---
{{% capture overview %}}
이 문서는 쿠버네티스 [네트워크 정책 API](/docs/concepts/services-networking/network-policies/)를 사용하여 파드가 서로 통신하는 방법을 제어하는 네트워크 정책(NetworkPolicy)을 선언하는데 도움된다.
{{% /capture %}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

네크워크 정책을 지원하는 네트워크 제공자가 구성되었는지 확인한다. 다음을 포함하여 네트워크 정책을 지원하는 여러 네트워크 공급자가 있다.

* [캘리코(Calico)](/docs/tasks/administer-cluster/network-policy-provider/calico-network-policy/)
* [시리움(Cilium)](/docs/tasks/administer-cluster/network-policy-provider/cilium-network-policy/)
* [큐브 라우터(Kube-router)](/docs/tasks/administer-cluster/network-policy-provider/kube-router-network-policy/)
* [로마나(Romana)](/docs/tasks/administer-cluster/network-policy-provider/romana-network-policy/)
* [위브 넷(Weave Net)](/docs/tasks/administer-cluster/network-policy-provider/weave-network-policy/)

{{< note >}}
위의 목록은 권장 사항이나 기본 설정이 아니라, 제품 이름별로 알파벳순으로 정렬된 것이다. 이 예제는 이러한 공급자 중 하나를 사용하는 쿠버네티스 클러스터에 유효하다.
{{< /note >}}
{{% /capture %}}

{{% capture steps %}}

## `nginx` 디플로이먼트를 생성하고 서비스로 공개한다.

쿠버네티스 네트워크 정책의 작동 방식을 확인하려면, `nginx` 디플로이먼트를 생성해서 시작하자.

```console
kubectl run nginx --image=nginx --replicas=2
```
```
deployment.apps/nginx created
```

그리고 서비스를 통해 노출한다.

```console
kubectl expose deployment nginx --port=80
```

```
service/nginx exposed
```

이는 기본 네임스페이스에서 두 개의 `nginx` 포드를 실행하고, 이를 `nginx`라는 서비스로 노출한다.

```console
kubectl get svc,pod
```

```
NAME                        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/kubernetes          10.100.0.1    <none>        443/TCP    46m
service/nginx               10.100.0.16   <none>        80/TCP     33s

NAME                        READY         STATUS        RESTARTS   AGE
pod/nginx-701339712-e0qfq   1/1           Running       0          35s
pod/nginx-701339712-o00ef   1/1           Running       0          35s
```

## 다른 파드에서 접근하여 서비스를 시험하자

다른 파드에서 새 `nginx` 서비스로 접근할 수 있어야 한다. 시험하려면 기본 네임스페이스의 다른 파드에서 서비스에 접근하자. 네임스페이스의 격리를 활성화하지 않았는지 확인한다.

busybox 컨테이너를 시작하고 `wget`을 `nginx` 서비스에 사용한다.

```console
kubectl run busybox --rm -ti --image=busybox /bin/sh
```

```console
Waiting for pod default/busybox-472357175-y0m47 to be running, status is Pending, pod ready: false

Hit enter for command prompt

/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.100.0.16:80)
/ #
```

## `nginx` 서비스를 제한하기

레이블에 `access: true`인 파드에서만 쿼리할 수 있도록, `nginx` 서비스를 제한하기 원한다고 가정하자. 이렇게 하려면, 이런 파드에서만 접속을 허용하도록 `네트워크 정책`을 생성하자.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      run: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"
```

## 서비스에 정책을 할당한다

kubectl을 이용하여 위에 nginx-policy.yaml에서 네트워크 정책을 만든다.

```console
kubectl apply -f nginx-policy.yaml
```

```
networkpolicy.networking.k8s.io/access-nginx created
```

## access 레이블이 정의되지 않았을 때에 서비스 접근 시험하기
올바른 레이블없는 파드에서 nginx 서비스에 접근하려 하면, 요청 시간이 초과될 것이다.

```console
kubectl run busybox --rm -ti --image=busybox /bin/sh
```

```console
Waiting for pod default/busybox-472357175-y0m47 to be running, status is Pending, pod ready: false

Hit enter for command prompt

/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.100.0.16:80)
wget: download timed out
/ #
```

## 레이블을 정의하고 다시 시험하기

올바른 레이블을 갖는 파드를 생성하면, 요청이 허용되었음을 확인할 수 있다.

```console
kubectl run busybox --rm -ti --labels="access=true" --image=busybox /bin/sh
```

```console
Waiting for pod default/busybox-472357175-y0m47 to be running, status is Pending, pod ready: false

Hit enter for command prompt

/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.100.0.16:80)
/ #
```
{{% /capture %}}


