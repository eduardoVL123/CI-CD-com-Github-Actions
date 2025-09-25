# Projeto de CI/CD: Pipeline Automatizado com GitHub Actions, Docker e ArgoCD

Este projeto demonstra a criação de um pipeline completo de Integração Contínua (CI) e Entrega Contínua (CD) para uma aplicação web simples em FastAPI.

O objetivo é automatizar todo o ciclo de vida da aplicação: desde o `commit` de uma alteração no código, passando pela construção e publicação de uma imagem Docker, até a implantação automática em um cluster Kubernetes local gerenciado via GitOps pelo ArgoCD.

## ✨ Conceitos Principais

* **Integração Contínua (CI):** O código é automaticamente testado e "empacotado" em uma imagem Docker a cada alteração, garantindo que novas mudanças não quebrem a aplicação.Para isso, utilizamos o **GitHub Actions**.
* **Entrega Contínua (CD) com GitOps:** O estado do cluster Kubernetes é definido em um repositório Git (a "fonte da verdade").O **ArgoCD** garante que o ambiente de produção espelhe exatamente o que está definido nos manifestos desse repositório. 

## 🛠️ Stack de Tecnologias

* **Aplicação:** Python com FastAPI 
* **Contêineres:** Docker 
* **Orquestração:** Kubernetes (via Rancher Desktop) 
* **CI (Automação de Build/Push):** GitHub Actions 
* **CD (GitOps):** ArgoCD 
* **Registry de Imagens:** Docker Hub 
## 🏗️ Arquitetura do Projeto

Este projeto utiliza dois repositórios Git, seguindo uma prática comum em ambientes GitOps:

1.  **`hello-app`**: Repositório da aplicação.
    * Contém o código-fonte da aplicação FastAPI (`main.py`). 
    * Contém o `Dockerfile` para construir a imagem. 
    * Contém o workflow do GitHub Actions (`.github/workflows/`) que automatiza o build e o push para o Docker Hub.
2.  **`hello-manifests`**: Repositório de manifestos.
    * Contém os manifestos Kubernetes (`deployment.yaml`, `service.yaml`) que descrevem como a aplicação deve ser executada no cluster.
    * Este é o repositório que o ArgoCD monitora.
    <img width="285" height="60" alt="image" src="https://github.com/user-attachments/assets/db1eb49c-45a5-4c6f-a6d3-22f66c5e4b02" />

    * ## 📋 Pré-requisitos

Para executar este projeto, você precisará de:

* Conta no GitHub (repositórios públicos)
* Conta no Docker Hub com um token de acesso 
* Rancher Desktop com Kubernetes habilitado 
* `kubectl` configurado 
* Git, Python 3 e Docker instalados localmente

## 🚀 Guia de Execução

### Fase 1: Configuração Inicial

1.  **Crie os Repositórios:** Crie os dois repositórios (`hello-app` e `hello-manifests`) no GitHub. 
2.  **Configure os Segredos:** No repositório `hello-app`, vá em `Settings > Secrets and variables > Actions` e crie os segredos: 
    * `DOCKER_USERNAME`: Seu nome de usuário do Docker Hub. 
    * `DOCKER_PASSWORD`: Seu token de acesso do Docker Hub. 
3.  **Crie o Repositório no Docker Hub:** Crie manualmente um repositório público no Docker Hub chamado `hello-app`.

### Fase 2: Deploy da Aplicação

1.  **Clone os Repositórios:** Clone ambos os repositórios para a sua máquina local.
2.  **Adicione os Arquivos:**
    * No `hello-app`, adicione os arquivos `main.py`, `requirements.txt`, `Dockerfile` e o workflow `.github/workflows/ci-pipeline.yml`. 
    * No `hello-manifests`, adicione os arquivos `deployment.yaml` e `service.yaml`. 
3.  **Primeiro Push (CI):** Faça o `push` dos arquivos para o repositório `hello-app`. Isso irá acionar o GitHub Actions, que construirá e enviará a imagem para o Docker Hub com uma tag única baseada no hash do commit.
4.  **Atualize o Manifesto:** Pegue a tag da imagem recém-criada no Docker Hub e atualize a linha `image:` no seu arquivo `deployment.yaml` local. 
5.  **Segundo Push (CD):** Faça o `push` dos arquivos de manifesto para o repositório `hello-manifests`.

### Fase 3: Configuração do ArgoCD

1.   **Instale o ArgoCD:** Se ainda não o fez, instale o ArgoCD no seu cluster local. 
2.  **Acesse a Interface:** Use `kubectl port-forward svc/argocd-server -n argocd 8080:443` e acesse `https://localhost:8080`.
3.  **Crie o App:** Crie um novo App no ArgoCD com as seguintes configurações: 
    * **Repository URL:** URL do seu repositório `hello-manifests`. 
    * **Path:** `.`
    * **Destination Cluster:** `https://kubernetes.default.svc`
    * **Namespace:** `default`
    * **Sync Policy:** `Automatic`
4.  **Sincronize:** O ArgoCD irá detectar os manifestos e implantar a aplicação.O status deve se tornar `Healthy` e `Synced`. 
  <img width="1580" height="438" alt="image" src="https://github.com/user-attachments/assets/473f1e14-2b9d-44d5-abe0-ddc9590309dc" />

### Fase 4: Verificação Final

1.  **Verifique o Pod:** No terminal, `kubectl get pods` deve mostrar o pod com status `Running`.
2.  **Acesse a Aplicação:** 
    ```bash
    # Crie o túnel de conexão para o serviço
    kubectl port-forward svc/hello-app-service 8081:80
    ```
    Acesse `http://localhost:8081` no navegador.A mensagem `{"message":"Hello World"}` deve aparecer. 
    <img width="1218" height="164" alt="image" src="https://github.com/user-attachments/assets/1cf7c60b-0c23-4d0b-81eb-d5a922f07c9f" />

