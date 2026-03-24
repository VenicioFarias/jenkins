🚀 Guia Completo: CI/CD Jenkins → Kubernetes
Vou te guiar passo a passo para configurar um pipeline completo de CI/CD do Jenkins para Kubernetes.

📋 Índice
1. Pré-requisitos
2. Configuração do Kubernetes
3. Configuração do Jenkins
4. Manifestos Kubernetes
5. Jenkinsfile
6. Troubleshooting

1️⃣ Pré-requisitos
No seu cluster Kubernetes:

# Verificar se o namespace existe
```kubectl get namespace projeto-apis-pefoce

# Se não existir, criar
kubectl create namespace projeto-apis-pefoce```

Ferramentas necessárias:

Jenkins 2.541.2 ✅
kubectl configurado
Docker Registry (GitLab Registry ou Docker Hub)
Acesso ao cluster Kubernetes

2️⃣ Configuração do Kubernetes
2.1 Criar Service Account para o Jenkins
```
bash# Criar arquivo: k8s-setup/jenkins-sa.yaml
----------------------------------------------
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-deployer
  namespace: projeto-apis-pefoce

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: jenkins-deployer-role
  namespace: projeto-apis-pefoce
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "secrets"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "statefulsets"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-deployer-binding
  namespace: projeto-apis-pefoce
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins-deployer-role
subjects:
  - kind: ServiceAccount
    name: jenkins-deployer
    namespace: projeto-apis-pefoce

---
# Token de longa duração para o Jenkins
apiVersion: v1
kind: Secret
metadata:
  name: jenkins-deployer-token
  namespace: projeto-apis-pefoce
  annotations:
    kubernetes.io/service-account.name: jenkins-deployer
type: kubernetes.io/service-account-token

```
2.2 Aplicar configurações


```
# Aplicar o Service Account
kubectl apply -f k8s-setup/jenkins-sa.yaml

# Obter o token (guarde esse token!)
kubectl get secret jenkins-deployer-token -n projeto-apis-pefoce -o jsonpath='{.data.token}' | base64 --decode

# Obter o CA Certificate
kubectl get secret jenkins-deployer-token -n projeto-apis-pefoce -o jsonpath='{.data.ca\.crt}' | base64 --decode > k8s-ca.crt

# Obter a URL do cluster
kubectl cluster-info
```
2.3 Criar Secret para Docker Registry
```
# Para GitLab Registry
kubectl create secret docker-registry gitlab-registry \
  --docker-server=172.27.39.88:5000 \
  --docker-username=seu-usuario \
  --docker-password=seu-token \
  --docker-email=seu-email@exemplo.com \
  -n projeto-apis-pefoce

# Verificar
kubectl get secret gitlab-registry -n projeto-apis-pefoce
```

---

## 3️⃣ Configuração do Jenkins

### 3.1 Instalar Plugins Necessários

Vá em: **Manage Jenkins → Plugins → Available**

Instale:
- ✅ Kubernetes CLI Plugin
- ✅ Kubernetes Plugin
- ✅ Docker Pipeline
- ✅ GitLab Plugin
- ✅ Pipeline

### 3.2 Configurar Credenciais do Kubernetes

**Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

#### Credencial 1: Kubernetes Token
```
Kind: Secret text
Scope: Global
Secret: [cole o token que você obteve acima]
ID: k8s-token
Description: Kubernetes Service Account Token
```

#### Credencial 2: Kubeconfig (Alternativa)
```
Kind: Secret file
Scope: Global
File: [upload do seu ~/.kube/config]
ID: kubeconfig
Description: Kubernetes Config
```

#### Credencial 3: Docker Registry
```
Kind: Username with password
Username: seu-usuario
Password: seu-token
ID: gitlab-registry-login
Description: GitLab Registry Credentials
```

3.3 Configurar Cloud Kubernetes (Opcional - para agents dinâmicos)
Manage Jenkins → Clouds → New cloud → Kubernetes
Name: kubernetes
Kubernetes URL: https://seu-cluster-k8s:6443
Kubernetes Namespace: projeto-apis-pefoce
Credentials: k8s-token