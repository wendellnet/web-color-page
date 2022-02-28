# Projeto Kubernets/ArgoCD
    Criando um processo para automatizar deploys, usando kubernetes + argocd
    Utilizei o Kind para rodar o cluster localmente

    Espero que minhas anotações lhe auxiliem.
    Convido a acompanharem esses procedimentos no canal do 
    [Fabricio-KubeDev](https://www.youtube.com/watch?v=4CtHL0XXdMM/)

# Criar um cluster kubernets    
    1: kind create cluster --name dataops
    2: kind create cluster --name dataops --config ./kind_config.yaml

    comandos para auxiliar:
    - kind get clusters
    - kubectl cluster-info --context dataops
    - kind delete cluster --name dataops
    - kubectl get deployment
    - kubectl get pods
    - kubectl get namespace
    - kubectl get svc
    - kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"

# Instalação do ArgoCd

    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    kubectl get all -n argocd
    kubectl port-forward service/argocd-server 8080:80 -m argocd

    comando para recuperar a senha do ARGO CD, utilizar um decode base 64
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
    
    criar uma aplicação no ARGOCD, configurar o git
    kubectl get applications -n argocd

# Clone o projeto
    git clone git@github.com:wendellnet/web-color-page.git

    kubectl apply -f .\k8s\deployment.yaml
    kubectl port-forward svc/webcolor 8080:80
    
    kubectl delete -f .\k8s\deployment.yaml

# Argocd no application.yaml
    comando para gerar o application.yaml:-> kubectl get applications web-color -n argocd -o yaml > application.yaml
```
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
    name: web-color
    namespace: argocd
    spec:
    destination:
        namespace: web-color
        server: https://kubernetes.default.svc
        
    project: default
    source:
        path: homolog
        repoURL: https://github.com/wendellnet/web-color-page.git
        targetRevision: HEAD
    syncPolicy: 
        syncOptions:
        - CreateNamespace=true
        automated:
        selfHeal: true
        prune: true
```

# Configurando dois ambientes com pastas de homologação e produção
    reconfigurar o application.yaml com dois manifestos
    criar pasta homolog e prod
    modificar o application.yaml com duas aplicações
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-color-homolog
  namespace: argocd
spec:
  destination:
    namespace: web-color-homolog
    server: https://kubernetes.default.svc
    
  project: default
  source:
    path: homolog
    repoURL: https://github.com/wendellnet/web-color-page.git
    targetRevision: HEAD
  syncPolicy: 
    syncOptions:
    - CreateNamespace=true
    automated:
      selfHeal: true
      prune: true
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-color-prod
  namespace: argocd
spec:
  destination:
    namespace: web-color-prod
    server: https://kubernetes.default.svc
    
  project: default
  source:
    path: prod
    repoURL: https://github.com/wendellnet/web-color-page.git
    targetRevision: HEAD
  syncPolicy: 
    syncOptions:
    - CreateNamespace=true
    automated:
      selfHeal: true
      prune: true
```

```
    git add .
    git commit -am "adicionando dois manifestos"
    git push origin main
```
    kubectl apply -f .\application.yaml
    kubectl port-forward svc/webcolor 8081:80 -n web-color-homolog
    kubectl port-forward svc/webcolor 8082:80 -n web-color-prod