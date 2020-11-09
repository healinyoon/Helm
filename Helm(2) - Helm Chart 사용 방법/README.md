# 목차
* Helm Chart 기본 명령어 사용하기
* Helm Chart 커스터마이징 해보기
* 더 다양한 설치 방법들 살펴보기
* Helm Chart 릴리즈 업그레이드 및 실패 복구하기
* Helm Chart 설치 / 업그레이드 / 롤백에 관한 유용한 옵션 살펴보기
* Helm uninstall 하기
* Helm 저장소 작업하기
* Helm Chart 직접 만들기


# Helm Chart 기본 명령어 사용하기

## `helm search`: Chart 검색
Helm은 강력한 검색 명령어를 제공한다. 아래의 두 명령어는 서로 다른 유형의 Repository Source로부터 검색하는데 사용할 수 있다.

1. `helm search hub`  
여러 저장소에있는 helm chart를 포괄하는 `[helm hub](https://artifacthub.io/)`에서 검색한다.

2. `helm search repo`  
`helm repo add`를 사용하여 local helm client에 추가한 저장소에서 검색한다. 검색은 local data 상에서 이루어지며, 퍼블릭 네트워크 접속이 필요하지 않다.

### `helm search hub` 예시  
`helm search hub` 명령어를 사용하면 공개적으로 사용 가능한 chart를 찾아볼 수 있다.
```
$ helm search hub wordpress
URL                                               	CHART VERSION	APP VERSION	DESCRIPTION
https://hub.helm.sh/charts/bitnami/wordpress      	9.9.1        	5.5.3      	Web publishing platform for building blogs and ...
https://hub.helm.sh/charts/seccurecodebox/old-w...	2.1.0        	4.0        	Insecure & Outdated Wordpress Instance: Never e...
https://hub.helm.sh/charts/presslabs/wordpress-...	0.10.5       	0.10.5     	Presslabs WordPress Operator Helm Chart
```

### `helm search repo` 사용 예시  
`helm search repo` 명령어를 사용하면 기존에 추가된 저장소에서 사용 가능한 chart를 찾아볼 수 있다.
```
$ helm search repo wordpress
NAME            	CHART VERSION	APP VERSION	DESCRIPTION
stable/wordpress	9.0.3        	5.3.2
```

> (참고) `helm search`는 fuzzy string matcing 알고리즘을 사용하기 때문에 검색시 **단어** 또는 **구문의 일부분**만 입력해도 된다.
```
$ helm search repo word
NAME            	CHART VERSION	APP VERSION	DESCRIPTION
stable/wordpress	9.0.3        	5.3.2      	DEPRECATED Web publishing platform for building...

$ helm search repo wordp
NAME            	CHART VERSION	APP VERSION	DESCRIPTION
stable/wordpress	9.0.3        	5.3.2      	DEPRECATED Web publishing platform for building...
```

## `helm install`: Chart 패키지 설치
사용하려는 Chart를 찾은 후에는 `helm install {사용자 지정 release name} {chart name}` 명령어를 사용하여 설치해준다. 
```
$ helm install my-mariadb stable/mariadb
WARNING: This chart is deprecated
NAME: my-mariadb
LAST DEPLOYED: Tue Nov  3 05:07:12 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
This Helm chart is deprecated

(중략)
```

Chart를 설치하면 새로운 relase 인스턴스가 생성되는 것을 알아두자. 위의 명령어에서 release의 이름은 `my-mariadb`이다. 만약 별도의 사용자 지정 release name 없이 helm이 자동으로 생성해주는 이름을 그대로 사용하려면 `--generate-name` 옵션을 사용하면 된다.

Helm은 release시 설치되는 모든 리소스가 running 상태로 변할 때까지 기다리지 않는다. 따라서 release의 리소스 상태 추척을 계속하거나, 구성 정보를 확인하고 싶을 때는 `helm status` 명령어를 사용하자.
```
$ helm status my-mariadb
NAME: my-mariadb
LAST DEPLOYED: Tue Nov  3 05:40:42 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
This Helm chart is deprecated
```
그런데.. `helm status` 명령어를 사용해도 각 리소스 상태가 출력되지 않는다. 원래 Helm version2에서는 아래와 같이 출력되는데, version3에서 부터 리소스 출력이 사라졌다.
> (Helm version2에서의 `helm status` 출력 예시)
```
$ helm status mychart
LAST DEPLOYED: Fri Jun 28 11:16:50 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME     READY  UP-TO-DATE  AVAILABLE  AGE
mychart  1/1    1           1          4m2s

==> v1/Pod(related)
NAME                     READY  STATUS   RESTARTS  AGE
mychart-b9488ff5c-b6xzz  1/1    Running  0         4m2s

==> v1/Service
NAME     TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)  AGE
mychart  ClusterIP  10.110.150.104  <none>       80/TCP   4m2s
```

원인을 찾아보니, **"The main reason for this is that by doing this in Helm, it leads to code duplication (we basically end up copying kubectl code), possible bugs, and maintenance overhead in the Helm codebase. So I think it would be better to keep it out([상세 내용 확인하기](https://github.com/helm/helm/issues/5952))."** 라고 논의되고 있다. 해결하기 위한 노력([참고](https://github.com/helm/helm/pull/7728))이 이루어지고 있지만, 아직까지는 지원되지 않는다.

아쉽지만 `kubectl get all` 명령어로 조회하자.
```
$ kubectl get all | grep my-mariadb
pod/my-mariadb-master-0   0/1     Pending   0          44m
pod/my-mariadb-slave-0    0/1     Pending   0          44m
service/my-mariadb         ClusterIP   10.104.91.81   <none>        3306/TCP   44m
service/my-mariadb-slave   ClusterIP   10.105.42.35   <none>        3306/TCP   44m
statefulset.apps/my-mariadb-master   0/1     44m
statefulset.apps/my-mariadb-slave    0/1     44m
```

# Helm Chart 커스터마이징 해보기
Chart의 기본 구성 옵션만 사용하여, 대부분의 경우 선호하는 구성을 사용하기 위한 chart 커스터마이징 방법을 살펴보자.
Chart에 어떤 옵션이 구성 가능한지 보기 위해 `helm show values` 명령어를 사용한다.

```
$ helm show values stable/mariadb
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagePullSecrets
##
# global:
#   imageRegistry: myRegistryName
#   imagePullSecrets:
#     - myRegistryKeySecretName
#   storageClass: myStorageClass

## Use an alternate scheduler, e.g. "stork".
## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
##
# schedulerName:

## Bitnami MariaDB image
## ref: https://hub.docker.com/r/bitnami/mariadb/tags/
##
image:
  registry: docker.io
  repository: bitnami/mariadb
  tag: 10.3.22-debian-10-r27
  ## Specify a imagePullPolicy
  ## Defaults to 'Always' if image tag is 'latest', else set to 'IfNotPresent'
  ## ref: http://kubernetes.io/docs/user-guide/images/#pre-pulling-images
  ##
  pullPolicy: IfNotPresent
  ## Optionally specify an array of imagePullSecrets.
  ## Secrets must be manually created in the namespace.
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  # pullSecrets:
  #   - myRegistryKeySecretName

  ## Set to true if you would like to see extra information on logs
  ## It turns BASH and NAMI debugging in minideb
  ## ref:  https://github.com/bitnami/minideb-extras/#turn-on-bash-debugging
  debug: false

(중략)

```

YAML 형식의 파일에 있는 설정들을 override하여 패키지 설치시 함께 반영할 수 있다. 이때, 설치시 override할 설정 목록을 전달하는 방법은 두 가지 방법이 있다.  
1. `--values(또는 -f)`: override할 YAML 파일을 지정한다. 여러 번 지정할 수 있지만 가장 오른쪽에 있는 파일이 우선시된다.
2. `--set`: 명령줄 상에서 override를 직접 지정한다.

둘 중 우선 순위는 `--values`가 더 높다. `helm get values {release-name}` 명령어로 해당 release에 대한 커스터마이징 값을 조회할 수 있다. 그 외에도 `helm get` 명령어는 클러스터에서 릴리즈 정보를 확인할 때 유용하다.

## `--values` 옵션을 사용한 chart 커스터마이징

아래 명령어는 `user0`이라는 기본 MariaDB 사용자를 생성하고, 이 사용자에게 새로 생성된 `user0db` 데이터베이스에 대한 접근 권한을 부여한다(그 외 나머지 모든 기본 설정은 해당 chart의 설정을 따르게 된다).

```
$ echo '{mariadbUser: user0, mariadbDatabase: user0db}' > config.yaml

$ helm install -f config.yaml stable/mariadb --generate-name
WARNING: This chart is deprecated
NAME: mariadb-1604900489
LAST DEPLOYED: Mon Nov  9 05:41:33 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
This Helm chart is deprecated
```

위의 커스터마이징 옵션 명령어가 잘 반영되었음을 `helm get values {release-name}`으로 확인할 수 있다.
```
$ helm get values mariadb-1604900489
USER-SUPPLIED VALUES:
mariadbDatabase: user0db
mariadbUser: user0
```

## `--set` 옵션을 사용한 chart 커스터마이징
`--set`에 명시된 오버라이드 사항들은 configMap으로 보관되며, `helm upgrade`를 실행할 때, `--reset-values`를 명시하여 제거할 수 있다.

### `--set`의 형식과 제한점

* 0개 이상의 `키=값` 쌍을 인자로 받는다. `--set name=value`의 형태이며, YAML 형식으로는 아래와 같다.
```
name: value
```

* 여러개의 값들은 `,(쉼표)`로 구분된다. 따라서 `--set a=b, c=d`는 다음과 같다.
```
a: b
c: d
```

* 더 복잡한 표현되 지원한다. 예를 들어 `--set outer.inner=value`는 다음과 같이 표현된다.
```
outer:
  inner: value
```

* 리스트는 `{, }`를 사용하여 표현할 수 있다. 예를 들어 `--set name={a, b, c}`는 다음과 같이 표현된다.
```
name:
  - a
  - b
  - c
```

* Helm version >= 2.5.0 부터는 리스트의 index 연산이 가능하다. 예를 들어 `--set servers[0].port=80`와 `--set servers[0].port=8080, servers[0].host=example`은 다음과 같다.
```
servers:
  - port: 80
```

```
servers:
  - port: 8080
    host: example
```

* `--set` 인자에 특수문자를 포함해야하는 경우 `\(백슬래시)`을 사용하면 된다. `--set name=value1\,value2`는 다음과 같이 표현된다.
```
name: "value1,value2"
```

* 위와 비슷한 예로 `toYaml` 기능으로 annotation, label, node selector를 파싱하는 chart에서 종종 사용되는 `.(마침표)`를 포함해야하는 경우도 `\(백슬래시)`를 사용하면 된다. `--set nodeSelector."kubernetes\.io/role"=master`는 다음과 같이 표현된다.
```
nodeSelector:
  kubernetes.io/role: master
```

# 더 다양한 설치 방법들 살펴보기
`helm install {source}` 명령어를 사용하여 여러 source에서 helm chart 패키지를 설치할 수 있다. Source의 목록으로는 다음과 같은 것들이 있다.
* Helm Chart Repository
* local에 있는 chart 압축 파일: `helm install foo foo-0.1.1.tgz`
* local에 있는 압축 해제된 chart 디렉토리: `helm install foo path/to/foo`
* url `helm install foo https://example.com/charts/foo-1.2.3.tgz`

# Helm Chart 릴리즈 업그레이드 및 실패 복구하기

### 릴리즈 업그레이드
새로운 버전의 chart 패키지로 업그레이드 or chart의 구성을 변경하고자 할때 `helm upgrade` 명령어를 사용할 수 있다.   
업그레이드는 기존에 릴리즈된 리소스를 사용자가 입력한 정보에 따라 진행된다. Kubernetes chart는 크고 복잡할 수 있기 때문에 Helm은 최소한의 개입으로 업그레이드를 수행하려고 한다. 따라서 최근 릴리즈 이후로 변경된 것들만 업그레이드하게 된다.

업그레이드에 사용할 YAML 파일을 생성한다.
> upgrade-chart.yaml
```
mariadbUser: user2
```

아래의 명령어를 통해 업그레이드를 수행한다. 
```
$ helm upgrade -f upgrade-chart.yaml mariadb-1604900489 stable/mariadb
WARNING: This chart is deprecated
Release "mariadb-1604900489" has been upgraded. Happy Helming!
NAME: mariadb-1604900489
LAST DEPLOYED: Mon Nov  9 07:00:08 2020
NAMESPACE: default
STATUS: deployed
REVISION: 2
NOTES:
This Helm chart is deprecated
```

`REVISION: 2`를 통해 위에서 릴리즈한 버전에서 업그레이드 되었음을 확인할 수 있다.  

values도 수정되었는지 확인해자.
```
$ helm get values mariadb-1604900489
USER-SUPPLIED VALUES:
mariadbUser: user2
```
위와 같이 `mariadbUser: user2`로 변경된 것을 볼 수 있다.

### 릴리즈 롤백
릴리즈가 계획대로 되지 않는다면, `helm rollback {release} {revision}` 명령어를 사용하여 롤백할 수 있다.

아래의 예시는 mariadb-1604900489를 revision 1로 롤백하는 명령어다. 여기서 `revision`은 incremental revision으로 설치, 업그레이드, 롤백 등이 실행될 때마다 1씩 증가한다. 특정 릴리즈의 revision을 확인하고 싶을 땐 `helm history {release}` 명령어를 사용하자.
```
$ helm history mariadb-1604900489
REVISION	UPDATED                 	STATUS    	CHART         	APP VERSION	DESCRIPTION
1       	Mon Nov  9 05:41:33 2020	superseded	mariadb-7.3.14	10.3.22    	Install complete
2       	Mon Nov  9 07:00:08 2020	superseded	mariadb-7.3.14	10.3.22    	Upgrade complete
3       	Mon Nov  9 07:02:30 2020	superseded	mariadb-7.3.14	10.3.22    	Upgrade complete
4       	Mon Nov  9 09:04:50 2020	deployed  	mariadb-7.3.14	10.3.22    	Rollback to 1
```

# Helm Chart 설치 / 업그레이드 / 롤백에 관한 유용한 옵션 살펴보기
아래의 온션은 설치 / 업그레이드 / 롤백에 유용한 옵션들이다. 이 외에도 다양한 명령어 옵션들이 있으니 `helm {command} --help`를 참고하자.

* `--timeout`: 쿠버네티스 명령어가 완료되기를 기다려주는 시간 값(초) 기본값은 5m0s
* `--wait`: 릴리스가 성공적이었다고 기록하기 전에, 모든 포드들이 준비 상태가 되고 PVC들이 연결되고 디플로이먼트가 최소한(Desired - maxUnavailable)의 준비 상태 포드 수를 갖추며 서비스들이 IP 주소(LoadBalancer라면 인그레스)를 가질 때까지 기다린다. --timeout 값만큼 기다릴 것이다. 타임아웃되면 릴리스는 FAILED로 기록될 것이다. 참고: 롤링 업데이트 전략의 일부로서 replicas 설정은 1이고 maxUnavailable 설정은 0이 아닌 디플로이먼트의 경우, --wait는 최소 준비 상태 포드 수를 만족하면 준비 상태로 응답할 것이다.
* `--no-hooks`: 명령어에 대한 훅(hook) 작동을 생략함
* `--recreate-pods`(upgrade와 rollback에만 적용가능): 이 플래그는 모든 포드들의 재생성을 일으킬 수 있다 (디플로이먼트에 속한 포드들은 제외). (헬름 3에서 사용 중단(DEPRECATED))

# Helm uninstall 하기
Kubernetes cluster에서 릴리즈를 Uninstall 하려면 `helm uninstall {release}` 명령어를 사용하면 된다. 기본적으로 **릴리즈는 바로 삭제되기 때문에 uninstall된 리소스를 롤백하는 것은 불가능하다는 것을 늘 주의해야 한다**.

Helm3부터는 uninstall시 릴리즈의 기록도 함께 제거한다. 따라서 제거한 릴리즈의 기록은 남기고 싶다면 다음과 같이 `--keep-history` 옵션을 사용해야 한다.
```
$ helm uninstall mariadb-1604900489 --keep-history
release "mariadb-1604900489" uninstalled
```

`helm list --uninstalled` 명령어를 통해 제거된 릴리즈만 확인 가능하다.
```
$ helm list --uninstalled
NAME              	NAMESPACE	REVISION	UPDATED                                	STATUS     	CHART         APP VERSION
mariadb-1604900489	default  	4       	2020-11-09 09:04:50.655567649 +0000 UTC	uninstalled	mariadb-7.3.1410.3.22
```

`helm list --all` 명령어는 실패하거나, 삭제(`--keep-history`를 사용한 경우에만)된 기록을 포함하여 helm이 가지고 있는 모든 릴리즈 기록을 출력한다.

# Helm 저장소 작업하기
Helm3부터 기본 chart 저장소를 제공하지 않는다. `helm repo`는 저장소를 추가, 목록 조회, 제거하는 명령어를 제공한다.

### 저장소 추가
```
$ helm repo add {저장소 명} {저장소 URL}
```

### 저장소 목록 조회
```
$ helm repo list
```

### 저장소 업데이트
Chart repository는 자주 업데이트되므로, `helm repo update` 명령어를 실행하여 언제든 Helm client를 업데이트 할 수 있다.

### 저장소 제거
```
$ helm repo remove {저장소 명}
```

# Helm Chart 직접 만들기

`helm create` 명령어로 빠르게 Helm chart를 구성해볼 수 있다.
```
$ helm create my-first-chart
Creating my-first-chart
$ ls  | grep my-first-chart
my-first-chart
```

위와 같이 `my-first-chart`가 생성된 것을 확인할 수 있다. 생성된 chart를 수정하여 나만의 템플릿을 생성할 수 있다.

Chart를 수정한 후에 `helm lint` 명령어를 통해 형식에 오류가 없는 지 검증할 수 있다.
```
$ helm lint my-first-chart/
==> Linting my-first-chart/
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```
아직 아무것도 수정하지 않았기 때문에 당연히 오류가 발생한다.

Chart를 패키징할 때는 `helm package` 명령어를 사용한다.
```
$ helm package my-first-chart/
Successfully packaged chart and saved it to: /home/ldccai/Helm/Helm(2) - Helm Chart 사용 방법/my-first-chart-0.1.0.tgz
```

마지막으로 패키징한 Chart 패키지를 install하면 나만의 chart가 릴리즈 된다.
```
$ helm install my-first-chart ./my-first-chart-0.1.0.tgz
```

[다음 장에서는 Chart의 구조와 수정하는 방법에 대해 자세히 알아보겠다.](https://helm.sh/docs/topics/charts/)




# 출처
* https://helm.sh/ko/docs/intro/using_helm/