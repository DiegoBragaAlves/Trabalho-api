# Trabalho Final — Integração DevOps

## Implementação de uma Pipeline CI/CD com GitHub Actions, Docker, Amazon ECR e EC2

**Repositório:** [Trabalho-api](https://github.com/DiegoBragaAlves/Trabalho-api)

## 1. Descrição do projeto

Este projeto demonstra a implementação de uma esteira de Integração Contínua e Entrega Contínua (CI/CD) para uma API desenvolvida em Python utilizando FastAPI.

O objetivo é automatizar os processos de validação do código, execução de testes, construção de imagens Docker, publicação no Amazon Elastic Container Registry (ECR) e implantação da aplicação em uma instância Amazon EC2.

A aplicação disponibiliza endpoints HTTP para gerenciamento de tarefas e verificação de funcionamento. Os dados são armazenados em memória, sem necessidade de banco de dados.

## 2. Tecnologias utilizadas

| Tecnologia | Finalidade |
|---|---|
| Python 3.13 | Linguagem de programação |
| FastAPI | Desenvolvimento da API |
| pytest | Testes automatizados |
| Ruff | Análise e formatação do código |
| Git e GitHub | Versionamento do projeto |
| GitHub Actions | Automação CI/CD |
| Docker | Containerização da aplicação |
| Amazon ECR | Armazenamento das imagens Docker |
| Amazon EC2 | Hospedagem da aplicação |
| AWS IAM | Gerenciamento de permissões |
| SSH | Conexão para implantação na EC2 |

## 3. Estrutura do projeto

```text
Trabalho-api/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy.yml
│       ├── deploy-ec2.yml
│       └── aws.yml
├── app/
│   ├── __init__.py
│   └── main.py
├── tests/
│   └── test_api.py
├── docs/
├── .dockerignore
├── .python-version
├── Dockerfile
├── compose.yaml
├── pyproject.toml
├── requirements.txt
├── requirements-dev.txt
└── README.md
```

O arquivo `ci.yml` é responsável pela integração contínua. O arquivo `deploy.yml` contém a pipeline de testes, publicação no Amazon ECR e implantação na EC2.

O arquivo `aws.yml` mantém apenas uma execução manual informativa referente ao antigo fluxo ECS.

O arquivo `deploy-ec2.yml` corresponde ao fluxo anterior, que constrói a imagem diretamente na EC2, e deverá ser desativado para evitar implantações concorrentes.

## 4. Execução local

### Pré-requisitos

- Git
- Python 3.13
- Docker

Clone o repositório:

```bash
git clone https://github.com/DiegoBragaAlves/Trabalho-api.git
cd Trabalho-api
```

### Linux e macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements-dev.txt
uvicorn app.main:app --reload
```

### Windows PowerShell

```powershell
py -3.13 -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements-dev.txt
uvicorn app.main:app --reload
```

A aplicação estará disponível em:

- API: http://localhost:8000
- Documentação Swagger: http://localhost:8000/docs
- Verificação de saúde: http://localhost:8000/health

## 5. Endpoints da API

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/` | Informações da API |
| GET | `/health` | Estado e versão da aplicação |
| GET | `/tasks` | Listar tarefas |
| POST | `/tasks` | Criar tarefa |
| PATCH | `/tasks/{id}/done` | Concluir tarefa |

Exemplo de verificação:

```bash
curl http://localhost:8000/health
```

Resposta observada na versão implantada:

```json
{
  "status": "ok",
  "version": "1.0.0"
}
```

## 6. Testes automatizados e qualidade

O projeto utiliza pytest para validar o funcionamento da API e Ruff para verificar a qualidade e a formatação do código.

Comandos de execução local:

```bash
ruff check .
ruff format --check .
pytest
```

Essas verificações ajudam a identificar problemas antes da integração das alterações na branch principal.

## 7. Containerização com Docker

A aplicação é executada em um contêiner Docker, garantindo maior padronização entre os ambientes.

### Construção da imagem

```bash
docker build -t devops-python-api:local .
```

### Execução do contêiner

```bash
docker run -d \
  --name devops-python-api \
  -p 8000:8000 \
  devops-python-api:local
```

### Verificação

```bash
docker ps
curl http://localhost:8000/health
```

O Docker permite empacotar a aplicação e suas dependências em uma imagem que pode ser executada em diferentes ambientes.

## 8. Integração contínua — CI

A integração contínua é implementada utilizando o GitHub Actions.

O workflow `ci.yml` valida alterações realizadas no projeto, incluindo Pull Requests destinados à branch `main`.

As verificações incluem:

1. Obtenção do código-fonte.
2. Configuração do ambiente Python.
3. Instalação das dependências.
4. Verificação da qualidade e formatação com Ruff.
5. Execução dos testes automatizados com pytest.
6. Registro do resultado no GitHub Actions.

O processo permite identificar falhas antes da integração do código.

## 9. Entrega contínua — CD

A entrega contínua é implementada pelo arquivo `.github/workflows/deploy.yml`.

O workflow é acionado automaticamente após alterações na branch `main` e também permite execução manual.

### Etapas da pipeline

1. Baixar o código do GitHub.
2. Configurar o ambiente Python.
3. Instalar as dependências.
4. Executar os testes automatizados com pytest.
5. Autenticar na AWS utilizando credenciais armazenadas no GitHub Secrets.
6. Realizar login no Amazon ECR.
7. Construir uma imagem Docker.
8. Identificar a imagem com a tag correspondente ao SHA do commit.
9. Publicar a imagem no repositório privado `devops-python-api`.
10. Conectar à instância EC2 por SSH.
11. Autenticar o Docker da EC2 no Amazon ECR.
12. Baixar a imagem publicada utilizando `docker pull`.
13. Substituir o contêiner anterior.
14. Iniciar a aplicação atualizada.
15. Verificar o funcionamento pelo endpoint `/health`.

O job de implantação depende da aprovação dos testes automatizados. Caso os testes falhem, a publicação e a implantação não são executadas.

## 10. Arquitetura de implantação

O projeto utiliza a seguinte arquitetura:

```text
Desenvolvedor
     |
     v
GitHub — Branch de desenvolvimento
     |
     v
Pull Request para main
     |
     v
GitHub Actions — CI
     |
     v
Testes automatizados
     |
     v
Merge na branch main
     |
     v
GitHub Actions — CD
     |
     v
Construção da imagem Docker
     |
     v
Amazon ECR
     |
     v
Amazon EC2 — Docker Pull
     |
     v
Execução do contêiner
     |
     v
API FastAPI — Porta 8000
```

O Amazon ECR funciona como registro privado das imagens Docker.

A instância EC2 utiliza a função IAM `LabRole`, com permissões para leitura de imagens do ECR.

A conexão administrativa utilizada pelo workflow ocorre por SSH, com autenticação por chave.

## 11. Segurança e credenciais

As credenciais da AWS e do acesso SSH são armazenadas nos GitHub Secrets, evitando sua inclusão no código-fonte.

Secrets utilizados:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
EC2_HOST
EC2_HOST_KEY
EC2_SSH_PRIVATE_KEY_B64
EC2_USER
```

As credenciais da AWS Academy são temporárias e podem precisar de atualização quando a sessão do laboratório expira.

Nenhuma senha, chave privada ou token deve ser publicado no repositório.

A porta SSH 22 foi liberada para permitir a conexão dos runners do GitHub Actions. Essa configuração deve ser revista para ambientes permanentes, utilizando uma estratégia de acesso mais restritiva.

## 12. Aplicação publicada na AWS

A aplicação foi implantada em uma instância Amazon EC2 na região `us-east-1`.

**Endereço utilizado durante os testes:**

http://54.242.133.1:8000

**Endpoint de verificação:**

http://54.242.133.1:8000/health

Resposta confirmada:

```json
{
  "status": "ok",
  "version": "1.0.0"
}
```

O endereço IP é público e pode mudar caso a instância seja interrompida e reiniciada.

## 13. Verificação da versão implantada

A versão em execução pode ser verificada acessando:

```text
http://54.242.133.1:8000/health
```

O campo `version` identifica a versão apresentada pela aplicação.

Também é possível verificar o contêiner na instância EC2:

```bash
sudo docker ps
```

Para consultar os logs:

```bash
sudo docker logs devops-python-api
```

E para verificar a imagem utilizada:

```bash
sudo docker inspect devops-python-api \
  --format '{{.Config.Image}}'
```

No fluxo de publicação pelo ECR, a imagem é identificada pelo SHA do commit correspondente à execução do GitHub Actions.

## 14. Evidências da implementação

As evidências devem incluir:

- Execução bem-sucedida da pipeline de CI.
- Pull Request aprovado e integrado à branch `main`.
- Execução bem-sucedida da pipeline de publicação e implantação.
- Imagem publicada no Amazon ECR com sua tag.
- Instância Amazon EC2 em execução.
- Contêiner Docker em execução na EC2.
- Endpoint `/health` acessível.
- Demonstração de atualização automática para uma segunda versão.

As capturas de tela e os links dos Pull Requests serão reunidos para entrega no Moodle.

### Pull Requests

**Pull Request inicial:** [PR #1](https://github.com/DiegoBragaAlves/Trabalho-api/pull/1)

**Pull Request de atualização:** pendente de criação e validação.

## 15. Resultados alcançados

A implementação demonstrou a integração de ferramentas de desenvolvimento e infraestrutura em uma esteira automatizada.

Foram validados os testes automatizados, a construção de imagens Docker, a publicação no Amazon ECR e a implantação da API na EC2.

Durante a configuração, foram identificados e solucionados problemas relacionados às dependências de teste, credenciais temporárias da AWS Academy e conectividade SSH.

O projeto demonstra como práticas DevOps podem tornar o processo de entrega de software mais automatizado, rastreável e reproduzível.

## 16. Referências

- [GitHub Actions](https://docs.github.com/pt/actions)
- [FastAPI](https://fastapi.tiangolo.com/)
- [Docker](https://docs.docker.com/)
- [Amazon ECR](https://docs.aws.amazon.com/ecr/)
- [Amazon EC2](https://docs.aws.amazon.com/ec2/)
- [AWS IAM](https://docs.aws.amazon.com/iam/)
