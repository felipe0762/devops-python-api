# API DevOps de Tarefas — Pipeline CI/CD, Docker e AWS

Aplicação REST desenvolvida em **FastAPI** para demonstração de práticas de engenharia DevOps, incluindo testes automatizados, integração contínua (CI), containerização com Docker e estruturação de entrega contínua (CD) para AWS.

## Tecnologias Utilizadas

* **Linguagem & Framework:** Python 3.13, FastAPI, Uvicorn
* **Testes Automatizados:** Pytest, HTTPX
* **Containerização:** Docker (imagem `python:3.13-slim`, usuário não-root e `HEALTHCHECK`)
* **CI/CD:** GitHub Actions

## Endpoints Principais

* `GET /` — Verifica se a API está em execução.
* `GET /health` — Endpoint de monitoramento de saúde (*healthcheck*) da aplicação.
* `GET /tasks` — Lista as tarefas cadastradas.
* `POST /tasks` — Cria uma nova tarefa.
* `PATCH /tasks/{task_id}/done` — Marca uma tarefa como concluída.
* `GET /docs` — Documentação interativa (Swagger UI).

## Como Executar Localmente

### Via Docker (Recomendado)

1. Construa a imagem Docker:
   ```bash
   docker build -t devops-python-api .