# Helm 이란?
Helm은 Kubernetes package managing tool이다. Python에서의 pip, Node.js에서의 npm과 같은 역할을 하는 Kubernetes package 배포를 가능하게 하는 tool이라고 보면 된다.

### Helm의 필요성
Kubernetest에 하나의 애플리케이션을 배포하기 위해서는 간단하게 Pod만 배포해서는 사용하기 어려운 경우가 많다. 외부 서비스를 위한 Service, Pod 관리를 위한 Deployment등 추가적인 리소스 배포가 필요하기 때문이다.  
이를 위해 Helm은 애플리케이션 컨테이너 배포는 물론이고, 이에 필요한 Kubernetes 리소스를 모두 패키지 형태로 배포해주는 역할을 한다.

### Helm의 구성도

![](/images/helm.png)  
이미지 출처: https://tech.osci.kr/2019/11/23/86027123/

* Helm Chart: 템플릿화된 Kubernetes 리소스 설치 스크립트(yaml)
* Helm Chart Repository: Helm Chart의 모든 메터데이터가 저장된 저장소
* Helm Client: Helm Server 모듈과 통신
* Helm Server(Tiller): Helm Client의 요청을 처리하여 Kubernetes에 Chart를 설치하고 릴리즈를 관리

# Helm 설치
* [Helm 공식 설치 문서](https://helm.sh/ko/docs/intro/install/)

Helm을 설치하는 다양한 방법이 있으나, 여기서는 Helm에서 제공하는 최신 버전을 자동으로 가져와서 설치하는 스크립트를 사용한다.



# 출처
https://bcho.tistory.com/1335