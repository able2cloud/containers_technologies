Crie uma instância EC2 executando Ubuntu 22.04. Esta instância atuará como seu ambiente de trabalho durante o laboratório.
Etapa 1: Preparando o Ambiente

Acesse a instância EC2 usando SSH ou qualquer método preferido.
Atualize a lista de pacotes e o sistema operacional com os seguintes comandos:


sudo apt update
sudo apt upgrade -y


sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce -y

Instale o kubectl

sudo apt install -y kubectl

Ou
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.23.0/bin/linux/amd64/kubectl
sudo chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl


Install kind:


# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

 Instalação e configuração do Helm

O Helm é um gerenciador de pacotes para Kubernetes. Ele pode ser usado para instalar e gerenciar aplicativos Kubernetes de forma reproduzível.

Para instalar o Helm, abra um terminal e execute o seguinte comando:

curl https://raw.githubusercontent.com/helm/helm/master/scripts/get | bash
Depois de instalar o Helm, você precisa configurá-lo. Para fazer isso, execute o seguinte comando:
helm init




----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Vire root:
sudo su -


Laboratório Kubernetes usando Kind: Trabalhando com Recursos Avançados
Configuração Inicial
Criação do Cluster
Neste primeiro passo, você criará um cluster usando Kind.


kind create cluster --name aula5-cluster


Trabalhando com Namespaces
1. Criação de um Namespace
Namespaces ajudam a segmentar seu cluster Kubernetes, permitindo que você isole recursos logicamente.


kubectl create namespace aula5-namespace
2. Verificação dos Pods no Namespace
Este comando mostra se há algum pod rodando no namespace que acabamos de criar.


kubectl get pods --namespace aula5-namespace
Criação e Explicação de Network Policy
1. Criação de um Network Policy
Network Policies permitem restringir o tráfego entre os pods.

Salve o YAML abaixo como aula5-network-policy.yaml.


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
Execute o comando para aplicar o Network Policy:


kubectl apply -f aula5-network-policy.yaml
2. Verificação das Regras do Network Policy
Este comando lhe dará uma visão das políticas de rede que você definiu.


kubectl get networkpolicies --namespace aula5-namespace
Gerenciamento de Configurações e Dados Sensíveis
1. Criação e uso de um Secret
Secrets são usados para armazenar informações sensíveis como senhas e chaves.


kubectl create secret generic aula5-secret --from-literal=key1=value1 --from-literal=key2=value2 --namespace aula5-namespace
2. Criação e uso de um ConfigMap
ConfigMaps são semelhantes aos Secrets, mas para dados não-confidenciais.


kubectl create configmap aula5-configmap --from-literal=key1=value1 --from-literal=key2=value2 --namespace aula5-namespace
Trabalhando com Armazenamento Persistente
1. Criação de um Persistent Volume (PV)
Persistent Volumes oferecem uma maneira de criar um armazenamento durável para seus pods.

Salve o YAML abaixo como aula5-pv.yaml.


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

Execute o comando para aplicar o Persistent Volume:

kubectl apply -f aula5-pv.yaml
2. Criação de um Persistent Volume Claim (PVC)
PVCs permitem que os pods solicitem espaço de um Persistent Volume.

Salve o YAML abaixo como aula5-pvc.yaml.


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


kubectl apply -f aula5-pvc.yaml

O que é o Metrics Server?
O Metrics Server é um agregador de métricas de recursos, como CPU e memória, que são expostas pelo Kubelet em cada nó. Essas métricas são coletadas para fornecer as métricas de API para escalabilidade, 
bem como para o comando kubectl top, permitindo que você visualize rapidamente o uso de recursos em seu cluster. É uma peça fundamental para a compreensão da eficiência e gerenciamento de recursos em um cluster Kubernetes.

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml

liste os pods: kubectl get pods -A

se os pods do metrics server estiverem rodando, execute kubectl top pods -A

