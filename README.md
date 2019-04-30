# PKS / Kubernetes Workshop

## Getting Started

### If you're completely unprepared

* download docker at https://www.docker.com/products/docker-desktop
* download kubectl at https://kubernetes.io/docs/tasks/tools/install-kubectl/
* download helm at https://helm.sh/docs/using_helm/#installing-helm

### If you don't have your own Kubernetes Cluster

* Log into Kubernetes by going to http://gangway.workshop.paulczar.wtf/.  After logging in using the credentials supplied to you you can download a kubeconfig file, or copy and paste a series of commands.

> you'll need to accept a self signed cert during the auth process ... sorry about that :)

* Log into Harbor Registry by going to https://harbor.workshop.paulczar.wtf/

* Create a project for your username in harbor here - https://harbor.workshop.paulczar.wtf/harbor/projects

* Log Docker into Harbor by running:  `$ docker login harbor.workshop.paulczar.wtf` and follow the prompts.

* Verify Kubernetes is working:  `$ kubectl version` you should see both a local and server version.

* Verify you can download a docker image: `$ docker pull harbor.workshop.paulczar.wtf/library/nginx:stable-alpine`

### Download example applications

    git clone https://github.com/paulczar/pksw6p.git

    cd pks-workshop

    export K8S_USER=<username>

    export TILLER_NAMESPACE=<username>

------
# STOP HERE and wait for workshop
------

## docker

Build and run the Hello world app

    cd hello

    docker build -t $K8S_USER/hello -f Dockerfile.cp .

    docker run --name hello -d -p 8080:8080 $K8S_USER/hello

Have a look at what Containers are running:

```console
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                    NAMES
fe3dfa017ff7        paulczar/hello      "java -jar /app.jar"   6 seconds ago       Up 5 seconds        0.0.0.0:8080->8080/tcp   hello
```

Browse to it via a web browser (or curl):

```
$ curl localhost:8080
hello development
```

## docker compose

```
cd ../coffee

cat docker-compose.yaml

docker-compose up -d

docker-compose ps

docker ps

docker-compose logs -f source processor sink

docker-compose stop
```

## Kubernetes

### Pods and Deployments

    kubectl run hello --image=paulczar/hello \
      --port=8080

    kubectl get all

    kubectl scale --replicas=3 \
      deployment/hello

    kubectl get all

    kubectl patch deployment hello \
      --patch '
    spec:
      template:
        spec:
          containers:
            - name: hello
              env:
                - name: MESSAGE
                  value: HELLO PKS WORKSHOP
    '
    kubectl get all

    kubectl port-forward deployment/hello 8080

    curl localhost:8080

### Services

    kubectl expose deployment \
      hello --type=LoadBalancer \
      --port 80 --target-port 8080

### ConfigMaps and Secrets

    kubectl create configmap hello \
     --from-literal='message=Hello Nashville'

    kubectl get configmap hello -o yaml

    kubectl create secret generic hello --from-literal='message=Hello Nashville'

    kubectl get secret hello -o yaml

### Cleanup
    kubectl delete deployment hello
    kubectl delete service hello
    kubectl delete configmap hello
    kubectl delete secret hello

## Kube with manifests

    kubectl apply -f deploy

    kubectl port-forward deployment/hello 8080

    curl localhost:8080

    kubectl edit configmap hello

    curl localhost:8080

    kubectl delete -f deploy

## Helm

    mkdir ../helm
    cd ../helm

    helm version

### Helm Hello

    helm create hello
    cd hello

edit values.yaml

```yaml
image:
  repository: paulczar/hello
  tag: latest

message: "Hello World!"
```

edit templates/deployment.yaml

```yaml
        - name: {{ .Chart.Name }}
          env:
            - name: MESSAGE
              value: "{{ .Values.message }}"
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
```

> delete the section for `livenessCheck`.

```bash
helm install --name dev --set message="Hello Development" .

helm install --name prod --set message="Hello Production" .

kubectl port-forward deployment/prod-hello 8080:8080
kubectl port-forward deployment/dev-hello 8081:8080

curl localhost:8080

curl localhost:8081

helm delete --purge dev
helm delete --purge prod
```
