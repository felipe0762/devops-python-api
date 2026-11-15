# API DevOps de Tarefas

![Python](https://img.shields.io/badge/Python-3.13-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?logo=fastapi&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Slim-2496ED?logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?logo=githubactions&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-ECR%20%26%20EC2-232F3E?logo=amazonaws&logoColor=white)

API REST de gerenciamento de tarefas desenvolvida em **FastAPI**, estruturada com ciclo completo de **Engenharia DevOps**: controle de versão baseado em *feature branches* e *Pull Requests*, Integração Contínua (CI) com testes automatizados via `pytest`, empacotamento seguro em container Docker e Entrega Contínua (CD) automatizada para **Amazon ECR** e **Amazon EC2**.

---

## Índice

- [Arquitetura do Fluxo DevOps](#arquitetura-do-fluxo-devops)
- [Stack Tecnológica e Boas Práticas](#stack-tecnológica-e-boas-práticas)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Endpoints da API](#endpoints-da-api)
- [Execução Local](#execução-local)
- [Configuração da Pipeline de CD](#configuração-da-pipeline-de-cd)
- [Autor](#autor)

---

## Arquitetura do Fluxo DevOps

```text
[Feature Branch] ──► [Pull Request] ──► [CI: Pytest] ──► [Merge na main]
                                                               │
[Produção: EC2 :8000] ◄── [Deploy via SSH] ◄── [Push Amazon ECR] ◄── [CD Workflow]
```

1. **Versionamento e Governança:** Desenvolvimento isolado em *feature branches* (ex.: `feature/ci-cd-ec2`) com integração controlada na branch `main` mediante revisão e aprovação obrigatória dos testes no *Pull Request*.
2. **Integração Contínua (`.github/workflows/ci.yml`):** Acionada automaticamente em *pushes* e *Pull Requests*, instala as dependências e executa a suíte de testes unitários e de integração com `pytest`.
3. **Entrega Contínua (`.github/workflows/deploy.yml`):** Acionada após *merge* na `main` (ou manualmente via `workflow_dispatch`), constrói a imagem Docker, publica no **Amazon ECR** com rastreabilidade de commit e atualiza o container em produção na **Amazon EC2**.

---

## Stack Tecnológica e Boas Práticas

* **Aplicação & Testes:** Python 3.13, FastAPI, Uvicorn, Pytest e HTTPX.
* **Containerização (`Dockerfile`):**
  * Imagem base enxuta (`python:3.13-slim`) para menor superfície de ataque.
  * Aproveitamento de cache de camadas instalando `requirements.txt` antes do código-fonte.
  * Execução sob usuário não-privilegiado (`appuser`, UID `10001`).
  * Monitoramento ativo de saúde via `HEALTHCHECK` interno no endpoint `/health`.
* **Cloud & Automação:** GitHub Actions (com *GitHub Environments* `DES`), AWS IAM (`LabInstanceProfile`), Amazon ECR e Amazon EC2 (Ubuntu).

---

## Estrutura do Projeto

```text
devops-python-api/
├── .github/
│   └── workflows/
│       ├── ci.yml          # Esteira de Integração Contínua (Pytest)
│       └── deploy.yml      # Esteira de Deploy Contínuo (Amazon ECR & EC2)
├── app/                    # Código-fonte da API FastAPI
├── tests/                  # Suíte de testes automatizados
├── Dockerfile              # Build otimizado e seguro da imagem
├── requirements.txt        # Dependências da aplicação e de testes
└── README.md               # Documentação do repositório
```

---

## Endpoints da API

| Método | Rota | Descrição |
| :--- | :--- | :--- |
| `GET` | `/` | Status inicial da API e redirecionamento para `/docs` |
| `GET` | `/health` | Verificação de saúde (`{"status": "ok", "version": "1.0.0"}`) |
| `GET` | `/docs` | Documentação interativa via Swagger UI |
| `GET` | `/tasks` | Lista todas as tarefas cadastradas |
| `POST` | `/tasks` | Cria uma nova tarefa |
| `PATCH` | `/tasks/{task_id}/done` | Marca uma tarefa específica como concluída |

---

## Execução Local

### 1. Via Ambiente Virtual Python

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate   # Windows

pip install -r requirements.txt
pytest
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

### 2. Via Docker

```bash
docker build -t devops-python-api .
docker run -d --name devops-api -p 8000:8000 devops-python-api
```

Após iniciar a aplicação, acesse:
* **Swagger UI:** `http://localhost:8000/docs`
* **Healthcheck:** `http://localhost:8000/health`

---

## Configuração da Pipeline de CD

O workflow `.github/workflows/deploy.yml` opera sob o ambiente **`DES`** em **Settings > Environments** no GitHub, separando credenciais sensíveis e parâmetros de infraestrutura:

### Secrets (`Environment secrets`)
| Nome | Descrição |
| :--- | :--- |
| `AWS_ACCESS_KEY_ID` | Chave de acesso programático da AWS |
| `AWS_SECRET_ACCESS_KEY` | Chave secreta da AWS |
| `EC2_SSH_PRIVATE_KEY` | Conteúdo integral da chave privada `.pem` da instância EC2 |

### Variables (`Environment variables`)
| Nome | Descrição | Exemplo |
| :--- | :--- | :--- |
| `AWS_REGION` | Região AWS utilizada no ECR e EC2 | `us-east-1` |
| `ECR_REPOSITORY` | Nome do repositório privado no Amazon ECR | `devops-python-api` |
| `EC2_HOST` | Endereço IPv4 público da instância EC2 | `3.88.108.16` |
| `EC2_USER` | Usuário SSH da instância EC2 | `ubuntu` |

### Pré-requisitos na Instância EC2
* **Docker** e **AWS CLI v2** instalados e acessíveis no `PATH` (`/usr/bin/aws`).
* **IAM Role** (`LabInstanceProfile`) vinculada à instância para autenticação no Amazon ECR.
* **Security Group** com regra de entrada liberada na porta TCP `8000`.
