Crie uma instância EC2 executando Ubuntu 22.04. Esta instância atuará como seu ambiente de trabalho durante o laboratório.
## Etapa 1: Preparando o Ambiente

Acesse o terminal da instância EC2 usando SSH ou qualquer método preferido.
Após acessar, siga os passos abaixo:

### Atualize o Sistema Operacional

```
sudo apt update
sudo apt upgrade -y
```
### Instale o Docker

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce -y
```

### Instale o kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl /usr/local/bin/kubectl
```

Após a instalação, resete o terminal

```
reset
```

### Instale o Kind
#### Para AMD64 / x86_64 execute o comando abaixo

```
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
```
#### Para ARM64 execute o comando abaixo
```
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-arm64
```

Após executar o comando acima de acordo com a arquitetura do seu Sistema Operacional, execute os comandos abaixo:
```
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### Instalando e configuração do Helm

O Helm é um gerenciador de pacotes para Kubernetes. Ele pode ser usado para instalar e gerenciar aplicativos Kubernetes de forma reproduzível.

Para instalar o Helm execute o comando abaixo:
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get | bash
Depois de instalar o Helm, você precisa configurá-lo. Para fazer isso, execute o seguinte comando:
helm init
```

## Etapa 2: Criando o Cluster utilizando o Kind

```
kind create cluster --name aula5-cluster
```

### Trabalhando com Namespaces

Namespaces ajudam a segmentar seu cluster Kubernetes, permitindo que você isole recursos logicamente.

#### Criando namespace

```
kubectl create namespace aula5-namespace
```

#### Listando os namespaces

```
kubectl get pods --namespace aula5-namespace
```

### Trabalhando com Network Policy

Network Policies permitem restringir o tráfego entre os pods.

Criando um Network Policy

```
vim aula5-network-policy.yaml
```

Cole o conteúdo abaixo dentro do arquivo:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: aula5-network-policy
  namespace: aula5-namespace
spec:
  podSelector:
    matchLabels:
      app: aula5-app
  ingress:
    - from:
      - podSelector:
          matchLabels:
            app: aula5-database
```

Salve o arquivo e saia editor vim igitando `ESC` e na sequênca `:wq!`

Agora execute o comando para criar o Network Policy:

```
kubectl apply -f aula5-network-policy.yaml
```

Verificação das Regras do Network Policy
Este comando lhe dará uma visão das políticas de rede que você definiu.

```
kubectl get networkpolicies --namespace aula5-namespace
```

### Gerenciamento de Configurações e Dados Sensíveis

#### Criando uma secret
Secrets são usados para armazenar informações sensíveis como senhas e chaves.

Execute o comando abaixo para criar uma secret:

```
kubectl create secret generic aula5-secret --from-literal=key1=value1 --from-literal=key2=value2 --namespace aula5-namespace
```

#### Criando um ConfigMap
ConfigMaps são semelhantes aos Secrets, mas para dados não-confidenciais.

```
kubectl create configmap aula5-configmap --from-literal=key1=value1 --from-literal=key2=value2 --namespace aula5-namespace
```

### Trabalhando com Armazenamento Persistente
Persistent Volumes oferecem uma maneira de criar um armazenamento durável para seus pods.

#### Criando um Persistent Volume (PV)

```
vim aula5-pv.yaml
```

Cole o conteúdo abaixo dentro do arquivo:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aula5-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /data/aula5-pv
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - aula5-cluster-control-plane
```

Salve o arquivo e saia editor vim igitando `ESC` e na sequênca `:wq!`
Agora execute o comando para criar o Persistent Volume:

```
kubectl apply -f aula5-pv.yaml
```

#### Atrelando o Persistent Volume ao Persistent Volume Clain
PVCs permitem que os pods solicitem espaço de um Persistent Volume.

```
vim aula5-pvc.yaml
```

Cole o conteúdo abaixo dentro do arquivo:
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: aula5-pvc
  namespace: aula5-namespace
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
Execute o comando para aplicar o PVC:
```

Salve o arquivo e saia editor vim igitando `ESC` e na sequênca `:wq!`
Agora execute o comando para criar o Persistent Volume Claim (PVC):

```
kubectl apply -f aula5-pvc.yaml
```

## Etapa 3: Criando o Metrics Server

O que é o Metrics Server?
O Metrics Server é um agregador de métricas de recursos, como CPU e memória, que são expostas pelo Kubelet em cada nó. Essas métricas são coletadas para fornecer as métricas de API para escalabilidade,
bem como para o comando kubectl top, permitindo que você visualize rapidamente o uso de recursos em seu cluster. É uma peça fundamental para a compreensão da eficiência e gerenciamento de recursos em um cluster Kubernetes.

Execute o comando abaixo:

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

liste os pods para verificar se criou o Metrics Server

```
kubectl get pods -A
```

Saída do comando:

```
kubectl get pods -A
NAMESPACE            NAME                                                      READY   STATUS    RESTARTS   AGE
kube-system          pod/metrics-server-8446d6695-pt75m                        1/1     Running   0          10s
```

## Etapa 4: Criando HPA
O Horizontal Pod Autoscaler (HPA) é um recurso do Kubernetes que dimensiona automaticamente o número de Pods em um Deployment, Replica Set ou StatefulSet com base em uma métrica. A métrica pode ser a utilização da CPU, a utilização da memória, o número de solicitações por segundo ou qualquer outra métrica que seja relevante para a sua aplicação.

Agora iremos criar um deploy de exemplo com o HPA


```
vim aula5-hpa.yaml
```

Cole o conteúdo abaixo dentro do arquivo:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cpu-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cpu-demo
  template:
    metadata:
      labels:
        app: cpu-demo
    spec:
      containers:
      - name: cpu-demo
        image: busybox
        command: ["sleep", "infinity"]
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cpu-demo
  namespace: default
spec:
  maxReplicas: 10
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 80
        type: Utilization
    type: Resource
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cpu-demo
```

Salve o arquivo e saia editor vim igitando `ESC` e na sequênca `:wq!`
Agora execute o comando para criar o Persistent Volume Claim (PVC):

```
kubectl apply -f aula5-hpa.yaml
```

Execute o comando abaixo para listar o Deployment e o HPA que acabamos de criar.
```
kubectl get hpa,pod
NAME                                           REFERENCE             TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/cpu-demo   Deployment/cpu-demo   10%/80%   1         10        1          46m

NAME                           READY   STATUS    RESTARTS   AGE
pod/cpu-demo-55f747c65-6thg6   1/1     Running   0          58m
```

