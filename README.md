# 🚀 Projeto CI/CD com Docker, GitHub Actions, Kubernetes e ArgoCD

## 📌 Visão Geral
Este repositório contém um pipeline completo de **Integração Contínua (CI)** e **Entrega Contínua (CD)** utilizando as seguintes tecnologias:

- **Docker** → Containerização da aplicação.
- **GitHub Actions** → Automação do build e push da imagem.
- **Docker Hub** → Registro de imagens.
- **Kubernetes** → Orquestração de containers.
- **ArgoCD** → Continuous Delivery e sincronização automática dos manifests.

---

## 🛠️ Estrutura do Projeto
```
Projeto-CI-CD/
│
├── app/                     # Código da aplicação (Flask)
│   ├── main.py
│   ├── requirements.txt
│
├── hello-manifests/         # Manifests do Kubernetes
│   ├── deployment.yaml
│   ├── service.yaml
│
├── .github/workflows/       # Pipeline GitHub Actions
│   ├── ci-cd.yml
│
├── Dockerfile               # Dockerfile da aplicação
└── README.md                # Este arquivo
```

---

## 🐳 Docker

### Criar a imagem localmente
```bash
docker build -t seu-usuario-dockerhub/hello-app:latest .
```

### Rodar localmente
```bash
docker run -p 5000:5000 seu-usuario-dockerhub/hello-app:latest
```

---

## ⚙️ GitHub Actions

**Arquivo:** `.github/workflows/ci-cd.yml`

**Fluxo:**
1. Build da imagem Docker.  
2. Login no Docker Hub com secrets.  
3. Push da imagem para o repositório Docker Hub.  
4. Atualização automática dos manifestos no repositório `hello-manifests`.  

**Configuração de Secrets:**
- `DOCKER_USERNAME` → Seu usuário do Docker Hub.  
- `DOCKER_PASSWORD` → Token de acesso gerado no Docker Hub.  

---

## ☸️ Kubernetes

### Manifests

**Arquivo `deployment.yaml`:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - name: hello-app
        image: seu-usuario-dockerhub/hello-app:latest
        ports:
        - containerPort: 5000
```

**Arquivo `service.yaml`:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  selector:
    app: hello-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
```

---

## 🎯 ArgoCD

### Instalação (via kubectl)
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Acesso ao ArgoCD UI
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Acesse em: **https://localhost:8080**  

- Usuário: `admin`  
- Senha inicial:
```bash
kubectl -n argocd get secret argocd-secret -o jsonpath="{.data.admin\.password}" | base64 -d
```

---

## 🚀 Fluxo Final do CI/CD

### 1. Correções na Pipeline do GitHub Actions
- O workflow `ci-cd.yml` foi ajustado para garantir que a imagem seja construída e enviada corretamente para o Docker Hub com a tag `latest`.  
- Login ao Docker Hub usando o secret `DOCKER_PASSWORD`.  
- Nome da imagem padronizado: `docker.io/ruanmarduk/hello-app`.  
- Imagem construída com duas tags: `commit SHA` e `latest`.  
- Atualização do `deployment.yaml` com a nova tag usando `sed`.  
- Push automático dos manifests atualizados para o repositório `hello-manifests`.  
- Workflow configurado com permissões **Read and write** para evitar erros de `Permission denied`.  

### 2. Configuração da Aplicação no ArgoCD
- Aplicação `hello-app` criada no ArgoCD, apontando para o repositório de manifestos.  
- O ArgoCD monitora o repositório e aplica atualizações automaticamente no cluster.  
- Criado um secret `regcred` no Kubernetes com as credenciais do Docker Hub.  
- `deployment.yaml` atualizado com:
```yaml
imagePullSecrets:
  - name: regcred
```

### 3. Verificação do Deploy e Acesso à Aplicação
- **Status da Aplicação:** No ArgoCD, a aplicação aparece como `Synced` e `Healthy`.  
- **Pods em Execução:**  
  ```bash
  kubectl get pods
  ```
  Resultado esperado: pods `Running`.  
- **Acesso Local:**  
  ```bash
  kubectl port-forward svc/hello-service 8080:80
  ```
  Abra [http://localhost:8080](http://localhost:8080) → deve retornar:  
  ```json
  {"message": "Hello World"}
  ```

---
