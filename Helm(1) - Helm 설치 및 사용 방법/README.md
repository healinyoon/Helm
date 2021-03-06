# 목차
* Helm 이란?
* Helm 설치하기
* Helm Chart Repository 사용해보기

# Helm 이란?
Helm은 Kubernetes package managing tool이다. Python에서의 pip, Node.js에서의 npm과 같은 역할을 하는 Kubernetes package 설치를 가능하게 하는 tool이라고 보면 된다.

## Helm의 필요성
Kubernetest에 하나의 애플리케이션을 배포하기 위해서는 간단하게 Pod만 배포해서는 사용하기 어려운 경우가 많다. 외부 서비스를 위한 Service, Pod 관리를 위한 Deployment등 추가적인 리소스 배포가 필요하기 때문이다.  
이를 위해 Helm은 애플리케이션 컨테이너 배포는 물론이고, 이에 필요한 Kubernetes 리소스를 모두 패키지 형태로 배포해주는 역할을 한다.

## Helm의 구성도

![](/images/helm.png)  

이미지 출처: https://tech.osci.kr/2019/11/23/86027123/

* Helm Chart  
Helm 패키지이다. 이 패키지에는 Kubernetest 클러스터 내에서 애플리케이션을 동작시키기 위해 필요한 모든 리소스 정의가 포함되어 있다.  

* Helm Chart Repository  
Helm Chart를 저장하고 공유하는 저장소이다.

* Release  
Kubernetes 클러스터에서 구동되는 Chart의 인스턴스이다. 일반적으로 하나의 Chart는 동일한 클러스터 내에 여러번 설치될 수 있다. 설치할 때마다 새로운 Release가 생성된다.

* Helm Client  
Helm Server 모듈과 통신하는 역할을 한다.

* Helm Server(Tiller)  
Helm Client의 요청을 처리하여 Kubernetes에 Chart를 설치하고 릴리즈를 관리한다.


# Helm 설치하기
* [Helm 공식 설치 문서](https://helm.sh/ko/docs/intro/install/)

## Helm 설치
Helm을 설치하는 다양한 방법이 있으나, 여기서는 Helm에서 제공하는 최신 버전을 자동으로 가져와서 설치하는 스크립트를 사용한다.

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
```

```
$ chmod 700 get_helm.sh
```

```
$ ./get_helm.sh
Downloading https://get.helm.sh/helm-v3.4.0-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
```

## Helm Chart Repository 초기화
Helm이 준비되면 Chart Repository를 추가할 수 있다. 
현재 repository를 조회해보면 다음과 같이 아무것도 없다고 출력된다.
```
$ helm repo list
Error: no repositories to show
```

다양한 저장소를 추가할 수 있지만 가장 기본인 `Helm official stable charts`를 추가하였다.
```
$ helm repo add stable https://charts.helm.sh/stable
"stable" has been added to your repositories

$ helm repo list
NAME  	URL
stable	https://charts.helm.sh/stable
```

stable에서 제공하는 chart 리스트를 조회할 수 있다.
```
$ helm search repo stable
NAME                                 	CHART VERSION	APP VERSION            	DESCRIPTION
stable/acs-engine-autoscaler         	2.2.2        	2.1.1                  	DEPRECATED Scales worker nodes within agent pools
stable/aerospike                     	0.3.3        	v4.5.0.5               	A Helm chart for Aerospike in Kubernetes
stable/airflow                       	7.13.2       	1.10.12                	Airflow is a platform to programmatically autho...
stable/ambassador                    	5.3.2        	0.86.1                 	DEPRECATED A Helm chart for Datawire Ambassador

(중략)
```

## Helm Chart Repository 업데이트
Repository로 부터 최신 Chart를 다운로드 하기 위해서는 애플리케이션 설치 이전에 repository를 업데이트해주는 것이 좋다.
```
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```

# Helm Chart Repository 사용해보기

## Helm Chart 확인

Helm Chart를 찾아 애플리케이션을 설치하는 여러 가지 방법이 있지만, 시작하기 가장 좋은 방법은 위에서 구성한 official stable chart 중 하나를 사용하여 설치하는 것이다.

여기서는 `mysql`을 예시로 설치해본다.

설치하려는 MySQL chart의 간단한 정보는 다음 명령어를 통해 확인 가능하다.
```
$ helm show chart stable/mysql
apiVersion: v1
appVersion: 5.7.30
description: Fast, reliable, scalable, and easy to use open-source relational database
  system.
home: https://www.mysql.com/
icon: https://www.mysql.com/common/logos/logo-mysql-170x115.png
keywords:
- mysql
- database
- sql
maintainers:
- email: o.with@sportradar.com
  name: olemarkus
- email: viglesias@google.com
  name: viglesiasce
name: mysql
sources:
- https://github.com/kubernetes/charts
- https://github.com/docker-library/mysql
version: 1.6.7
```

만약 해당 chart에 대한 모든 정보를 확인하고 싶다면 다음 명령어를 사용하자(상당히 많은 정보가 출력된다).
```
$ helm show all stable/mysql
```

## Helm Chart를 이용하여 애플리케이션 설치

이제 helm chart를 이용하여 MySQL 애플리케이션을 설치해보자.
```
$ helm install stable/mysql --generate-name
NAME: mysql-1604311201
LAST DEPLOYED: Mon Nov  2 10:00:04 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql-1604311201.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql-1604311201 -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h mysql-1604311201 -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/mysql-1604311201 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```

위의 예에서 `stable/mysql` chart가 설치되었고, 이름은 `mysql-1604311201`인 것을 확인할 수 있다. 

Helm chart로 설치한 애플리케이션을 조회해보자.
```
$ helm ls
NAME            	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
mysql-1604311201	default  	1       	2020-11-02 10:00:04.300938829 +0000 UTC	deployed	mysql-1.6.7	5.7.30
```

당연히 `kubectl`로도 조회가 가능하다.
```
$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/mysql-1604311201-654cf77476-w58m7   0/1     Pending   0          35m

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP    55d
service/mysql-1604311201   ClusterIP   10.111.137.120   <none>        3306/TCP   35m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-1604311201   0/1     1            0           35m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-1604311201-654cf77476   1         1         0       35m
```

애플리케이션 설치를 위해 필요한 모든 kubernetes 리소스가 함께 설치된 것을 볼 수 있다.

이렇게 Helm chart를 사용하여 설치를 하게 되면 `새로운 release`가 된것이다. 즉 하나의 Helm chart로 여러번의 설치를 할 수 있는데, 이때 각각의 설치는 덮어씌어지는 것이 아니라 독립적으로 관리되고 업그레이드된다.

그렇다면 실제로 그렇게 동작하는지 테스트해보자. 위와 동일한 방식으로 mysql을 하나 더 설치한다.
```
$ helm install stable/mysql --generate-name
NAME: mysql-1604313417
LAST DEPLOYED: Mon Nov  2 10:37:00 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql-1604313417.default.svc.cluster.local
```

`helm ls`로 조회해보면 새로운 release가 생성되었음을 확인할 수 있고
```
$ helm ls
NAME            	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
mysql-1604311201	default  	1       	2020-11-02 10:00:04.300938829 +0000 UTC	deployed	mysql-1.6.7	5.7.30
mysql-1604313417	default  	1       	2020-11-02 10:37:00.578674443 +0000 UTC	deployed	mysql-1.6.7	5.7.30
```

`kubectl get all`로 조회해도 마찬가지로 독립적인 애플리케이션이 설치되었음을 알 수 있다.
```
$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/mysql-1604311201-654cf77476-w58m7   0/1     Pending   0          39m
pod/mysql-1604313417-9685cfbd6-srxtl    0/1     Pending   0          2m53s

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP    55d
service/mysql-1604311201   ClusterIP   10.111.137.120   <none>        3306/TCP   39m
service/mysql-1604313417   ClusterIP   10.104.229.201   <none>        3306/TCP   2m53s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-1604311201   0/1     1            0           39m
deployment.apps/mysql-1604313417   0/1     1            0           2m53s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-1604311201-654cf77476   1         1         0       39m
replicaset.apps/mysql-1604313417-9685cfbd6    1         1         0       2m53s
```

## Helm Chart를 이용하여 설치한 애플리케이션 제거
Helm chart를 이용하여 설치한 애플리케이션을 제거하는 방법은 아주 간단하다.
```
$ helm uninstall mysql-1604311201
release "mysql-1604311201" uninstalled

$ helm uninstall mysql-1604313417
release "mysql-1604313417" uninstalled

$ helm ls
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```


# 출처
* https://bcho.tistory.com/1335
* https://helm.sh/docs/intro/install/
* https://docs.helm.sh/docs/intro/quickstart/
* https://tech.osci.kr/2019/11/23/86027123/