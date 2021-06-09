# Kubernetes 101

---

## Kubernetes 소개

---

* 쿠버네티스의 전신 : 구글의 사내 프로젝트 경험을 바탕으로, 쿠버네티스를 오픈소스로 2014년 공개

  * Borg
  * Omega

* 쿠버네티스는 인프라를 추상화함으로써, 컨테이너 어플리케이션을 쉽게 배포하고 관리하도록 돕는다. 

* 쿠버네티스 장점

  * 어플리케이션 배포 단순화

  특정 하드웨어 리소스(예를 들어, SSD)를 필요로 할 경우, Labeling을 활용해서 노드 정보를 명시하지 않아도 자동으로 배포

  * 하드웨어 활용도 극대화

  여러 어플리케이션 구성 요소를 노드의 가용 리소스에 fit하게 match.

  * Health Check & Recovery

  노드를 모니터링하고 노드가 죽을 경우, 다른 노드로 일정을 자동 재조정. 운영하는 사람은 장애가 발생한 노드만 처리하면 됨.

  * Auto Scaling

  개별 어플리케이션의 부하를 지속적으로 모니터링할 필요 없이, 자동으로 리소스를 모니터링하고 인스턴스 수를 조정함

  * 어플리케이션 개발 단순화

  컨테이너가 개별 환경을 제공함으로써 개발 및 수정이 편해진다. 새로운 버전을 출시했는데 버그나 이슈를 발견할 경우 롤 아웃을 할 수 있다.

---

* 쿠버네티스 Cluster Architecture

  * 마스터 노드(관리자 역할), 워커 노드(실제 실행 담당)
  * Master Node
    * kube-apiserver
    * etcd
    * kube-scheduler
    *  controller-manager
  * Worker Node
    * kubelet
    * kube-proxy
    * Container Runtime(Pod)

  ![kube-architecture](../images/kubernetes1.png)

---

## Kubernetes 실습 on GCP

---

* Kubernetes 클러스터 만들기
  * Region만 바꾸고 진행. 마스터 버전은 default 값
* nginx 실행해보기(feat. Google Cloud Shell)

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --type=LoadBalancer
```

* Go를 활용해 앱 실행하기

  * hostname을 확인할 수 있는 간단한 프로그램 작성

  ```go
  package main
  
  import (
      "fmt"
      "github.com/julienschmidt/httprouter"
      "net/http"
      "log"
      "os"
  )
  
  func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
      hostname, err := os.Hostname()
      if err == nil {
          fmt.Fprint(w, "Welcome! " + hostname +"\n")
      } else {
          fmt.Fprint(w, "Welcome! ERROR\n")
      }
  }
  
  func main() {
      router := httprouter.New()
      router.GET("/", Index)
  
      log.Fatal(http.ListenAndServe(":8080", router))
  }
  ```

  * Go 언어 설치 및 앱 실행

  ```bash
  apt install golang
  go get github.com/julienschmidt/httprouter
  go build main.go
  ./main
  ```

  * dockerfile 작성

  ```dockerfile
  FROM golang:1.11
  WORKDIR /usr/src/app
  COPY main /usr/src/app
  CMD ["/usr/src/app/main"]
  ```

  * 도커 빌드/푸시

  ```bash
  sudo docker build -t oror1024/http-go
  sudo docker login
  sudo docker push oror1024/http-go
  ```

  * 컨테이너에 앱 배포

  ```bash
  sudo docker run -d -p 8080:8080 --rm http-go
  ```

  * 쿠버네티스에 배포

  ```bash
  kubectl create deployment http-go --image=oror1024/http-go
  kubectl expose deployment http-go --name http-go-svc --type=LoadBalancer --port=8080
  ```

  

  

  

