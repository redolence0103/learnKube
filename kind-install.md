# KIND(K8S in Docker) 기반의 K8S Cluster 환경 설정
<img width="100" src="![docker-process-docker-limit drawio](https://user-images.githubusercontent.com/90162116/141114099-ac264413-d995-4782-8888-89a9412d3259.png)"
## 서버(vm) 환경 설정 Docker 설치
```
sudo sysctl -w net.ipv4.ip_forward=1
sudo swapoff -a
sudo apt update -y
sudo apt install -y docker.io
sudo usermod -aG docker $USER
```
## Go 설치
```
wget https://golang.org/dl/go1.16.7.linux-amd64.tar.gz
sudo tar zfx go1.16.7.linux-amd64.tar.gz -C /usr/local
export PATH=$PATH:/usr/local/go/bin
```
## kubectl 설치
```
curl -LO https://dl.k8s.io/release/v1.21.0/bin/linux/amd64/kubectl
```
## kind 설치 및 k8s cluster 생성
```
GO111MODULE="on" go get sigs.k8s.io/kind@v0.11.1
sudo mv $PWD/go/bin/kind /usr/local/go/bin

cat << EOF > ./kind.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF

kind create cluster --config kind.yaml
```
## metallb 설치 및 설정

- kube-proxy를 ipvs mode로 설정
```
kubectl edit configmap -n kube-system kube-proxy
  .....
mode: "ipvs"
ipvs:
  strictARP: true
  .....
```
- metallb 설치
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
```
- metallb Layer2 설정
```
cat << EOF > ./config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.18.70.1-172.18.70.254
EOF

kubectl apply -f config.yaml
```
## helm 설치 (version 3)
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```
## ingress-nginx 설치
```
helm repo add stable https://charts.helm.sh/stable
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm repo list
helm show values ingress-nginx/ingress-nginx > /tmp/ingress-nginx.yaml
```
- hostNetwork 사용을 위한 config 설정 
```
vi /tmp/ingress-nginx.yaml
  .....
hostNetwork: True
  .....
  hostPort:
    enabled: True
  .....
kind: DaemonSet
  .....
:wq!
```
- ingress-nginx 설치
```
kubectl create ns ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace=ingress-nginx --values /tmp/ingress-nginx.yaml
```
## cluster에 matrix server 설치
- matrix server download
```
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
```
- kubelet과 비보안 통신 설정
```
vi components.yaml
  .....
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
  .....
:wq!

kubectl create -f components.yaml
```
#emptyDir
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs-fortune
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-rs-fortune
  template:
    metadata:
      labels:
        app: myapp-rs-fortune
    spec:
      containers:
      - name: web-server
        image: nginx:alpine
        volumeMounts:
        - name: web-fortune
          mountPath: /usr/share/nginx/html
          readOnly: true
        ports:
        - containerPort: 80
      - name: html-generator
        image: ghcr.io/c1t1d0s7/fortune
        volumeMounts:
        - name: web-fortune
          mountPath: /var/htdocs
      volumes:
      - name: web-fortune
        emptyDir: {}
```
## github jenkins token
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
