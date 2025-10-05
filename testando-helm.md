# Testando PODS com Helm Chart

O Helm Chart é uma ferramenta para criar a declaração de uma infraestrutura kubernetes e criá-la em um cluster. 

## Pré-requisitos

### 1. Usuário atual precisa estar no grupo docker

```bash 
# Ver se está no grupo docker
groups $USER

# Se não estiver
sudo usermod -aG docker $USER

# Conferir se alteração foi feita 
groups $USER

# Abre um novo shell com o grupo docker ativo
newgrp docker
```

### 2. Instalação de um cluster kubernetes (ex: Minikube)

Instruções de instalação: https://minikube.sigs.k8s.io/docs/start/

Para uso em Windows, é possível habilitar o Docker Desktop para criar um cluster kubernetes. 

### 3. Instalação do kubectl para interação com o cluster via CLI

Instruções de instalação: https://kubernetes.io/docs/tasks/tools/ 

### 4. Instalação do Helm

Instruções de instalação: https://helm.sh/docs/intro/install/

Alguns comandos iniciais do Helm Chart:

```bash
# Versão do helm
helm version

# Cria um novo template de helm chart
helm create <nome-do-chart>

# Fazer o deploy no cluster kubernetes
helm install <nome> /path/to/chart

# Remover a aplicação e infraestrutura do cluster kubernetes
helm uninstall <nome>
```

## Instalando o Helm Chart e Interagindo com os pods

Este projeto Helm Chart cria o deploy de um container que utiliza alguns recursos da AWS, emulados pelo Localstack. 

O comportamento esperado é que:

1. um pod seja iniciado com um container localstack.

2. um job seja instanciado para criar alguns recursos no pod localstack (um bucket S3 e uma fila SQS)

3. um cronjob seja iniciado para, a cada minuto, indefinidamente, enviar mensagens à fila SQS 

É possível consumir as mensagens enviadas interagindo manualmente com o pod localstack. 

### Instalando o chart no cluster

```bash
# Ativar o cluster inicializando o minikube
minikube start driver=docker

# Instalar o Helm chart, pode usar o nome que quiser
helm install <nome-do-recurso> /path/to/helm-chart

# Verificar se foi criado
helm list

# Verificar o status
helm status <nome-do-recurso>
``` 

### Interagindo com os pods e observando o comportamento do sistema

```bash
# Listando os pods e seus status
kubectl get pods

# Ver logs do pod
kubectl logs <nome-do-pod>

# Ver apenas jobs
kubectl get jobs

# Exibir detalhes de job e cronjobs
kubectl describe job
kubectl describe cronjob 

# Listar todos os pods com o label localstack
kubectl get pods -l app=localstack

# Ver os logs do pod localstack
kubectl logs <pod-localstack>

# Exibir o balde S3
kubectl exec <pod-localstack> -- awslocal s3 ls

# Exibir URL da fila criada
kubectl exec <pod-localstack> -- awslocal sqs list-queues

# Exibir as mensagens na fila (com invisibilidade temporária desabilitada)
kubectl exec <pod-localstack> -- awslocal sqs get-message --queue-name <URL-da-fila> --max-number-of-messages 10 --visibility-timeout 0 --wait-time-seconds 3 --output json

``` 


> **Sobre o uso do Localstack:**
>
> Neste projeto, os recursos de nuvem da AWS são emulados pelo Localstack. 
>
> O Localstack tem a capacidade de emular a api da AWS, suas respostas e o funcionamento de alguns serviços, como S3 e SQS, e código Lambda. 
>
> O Localstack **NÃO** consegue emular recursos computacionais, como instâncias EC2. A criação das instâncias é emulada, mas não é possível executar nenhum tipo de processamento real nelas.  
