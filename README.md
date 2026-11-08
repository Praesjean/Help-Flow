# Help Flow | Sistema de Gerenciamento de Chamados

O Help Flow é uma aplicação distribuída desenvolvida para o gerenciamento de chamados, usuários e indicadores de atendimento.

O projeto utiliza uma arquitetura baseada em microserviços, com Backend for Frontend (BFF), microfrontend e integração com serviços Serverless do Microsoft Azure.

A solução foi desenvolvida como parte da disciplina de Arquitetura de Soluções Cloud, com o objetivo de aplicar conceitos modernos de arquitetura de software e computação em nuvem.

## 1. Funcionalidades

O sistema contempla as seguintes funcionalidades:

### Gerenciamento de usuários

- Cadastro de usuários.
- Autenticação por e-mail e senha.
- Controle de acesso por perfil.
- Consulta, atualização e exclusão de usuários.

### Gerenciamento de chamados

- Abertura de chamados.
- Consulta de chamados.
- Atualização de informações.
- Controle de status.
- Classificação por prioridade.
- Exclusão conforme permissões de acesso.

### Indicadores e métricas

- Quantidade total de chamados.
- Chamados abertos e em andamento.
- Chamados resolvidos e fechados.
- Percentual de resolução.
- Distribuição de chamados por prioridade.

## 2. Tecnologias utilizadas

### Frontend

- React
- JavaScript
- Vite

### Backend

- Node.js 22
- Express 5
- Axios
- JSON Web Token (JWT)
- Swagger / OpenAPI
- Jest e Supertest

### Bancos de dados

- MongoDB Atlas
- Azure SQL Database

### Cloud e infraestrutura

- Microsoft Azure
- Azure Functions
- Azure Application Insights
- Docker
- Azure Functions Core Tools
- Azurite

## 3. Arquitetura geral

O Help Flow utiliza uma arquitetura distribuída composta por um microfrontend, um Backend for Frontend, dois microserviços independentes e uma Azure Function.

Cada componente possui responsabilidades específicas e pode ser desenvolvido e executado de forma independente.

### Visão geral da arquitetura

```text
             Microfrontend React
                     |
                     v
              API Gateway
             (previsto)
                     |
                     v
                BFF Node.js
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Chamados    Usuários   Azure Function
          |          |          |
          v          v          v
      MongoDB    Azure SQL   Indicadores
       Atlas                calculados
```

O BFF centraliza a comunicação entre o frontend e os serviços de backend, enquanto cada microserviço mantém sua própria responsabilidade e persistência de dados.

## 4. Componentes do sistema

O projeto está dividido em cinco componentes principais.

### 4.1 Microfrontend

Responsável pela interface de interação com os usuários.

Desenvolvido em React com Vite, disponibiliza as funcionalidades de gerenciamento de chamados e usuários por meio de uma aplicação web.

**Repositório:** `microfrontend`

### 4.2 Backend for Frontend (BFF)

Responsável por intermediar a comunicação entre o microfrontend e os microserviços.

Desenvolvido em Node.js com Express, centraliza as requisições, encaminha operações aos serviços e disponibiliza o endpoint de agregação de dados.

**Endpoint principal:**

`GET /aggregated-data`

**Porta local:** `3000`

**Repositório:** `bff`

### 4.3 Microserviço de Chamados

Responsável pelo gerenciamento dos chamados do sistema.

Desenvolvido em Node.js com Express, utiliza MongoDB Atlas para persistência e JWT para autenticação e autorização.

Implementa operações de criação, consulta, atualização e exclusão de chamados.

**Porta local:** `3001`

**Banco de dados:** MongoDB Atlas

**Repositório:** `microservice-chamados`

### 4.4 Microserviço de Usuários

Responsável pelo gerenciamento de usuários e autenticação da aplicação.

Desenvolvido em Node.js com Express, utiliza Azure SQL Database para persistência, bcrypt para proteção de senhas e JWT para autenticação.

**Porta local:** `3002`

**Banco de dados:** Azure SQL Database

**Repositório:** `microservice-usuarios`

### 4.5 Azure Function de Métricas

Responsável pelo processamento e cálculo de indicadores operacionais dos chamados.

Desenvolvida em Node.js utilizando HTTP Trigger, recebe os dados encaminhados pelo BFF e retorna as métricas calculadas.

**Método:** POST

**Rota:** `/api/getticketmetrics`

**Repositório:** `azure-function`

**Endpoint publicado:**

https://help-flow-metrics-c0dddcd0fvfpdkdp.brazilsouth-01.azurewebsites.net/api/getticketmetrics

## 5. Estrutura do repositório

O repositório principal organiza os componentes por meio de Git Submodules.

```text
Help-Flow/
├── azure-function/
├── bff/
├── microfrontend/
├── microservice-chamados/
├── microservice-usuarios/
├── .gitignore
└── README.md
```

Cada diretório de componente possui seu próprio repositório Git e README com informações específicas de implementação, configuração e execução.

O repositório principal concentra a documentação geral e a referência das versões dos componentes utilizados no projeto.

## 6. Fluxo de comunicação

O fluxo de comunicação do sistema ocorre da seguinte maneira:

1. O usuário interage com a interface React.
2. O frontend encaminha as requisições ao BFF.
3. O BFF identifica o serviço responsável pela operação.
4. As requisições são encaminhadas aos microserviços de Chamados ou Usuários.
5. Os microserviços consultam seus respectivos bancos de dados.
6. Quando necessário, o BFF encaminha os dados dos chamados à Azure Function.
7. A função processa os indicadores e retorna os resultados.
8. O BFF consolida as informações e retorna a resposta ao frontend.

### Endpoint de agregação

O endpoint `GET /aggregated-data` permite reunir informações dos diferentes componentes em uma única resposta JSON.

O acesso exige um token JWT válido.

As informações retornadas variam conforme o perfil do usuário autenticado:

- **ADMIN:** recebe dados de chamados e usuários.
- **ATENDENTE:** recebe dados de chamados e usuários.
- **CLIENTE:** recebe apenas os chamados autorizados, sem consultar a listagem de usuários.

As regras específicas de autorização continuam sendo aplicadas pelos microserviços.

## 7. Bancos de dados

O projeto utiliza o padrão Database per Service, mantendo a persistência independente em cada microserviço.

| Componente               | Banco de dados           | Responsabilidade          |
| ------------------------ | ------------------------ | ------------------------- |
| Microserviço de Chamados | MongoDB Atlas            | Persistência dos chamados |
| Microserviço de Usuários | Azure SQL Database       | Persistência dos usuários |
| BFF                      | Não possui banco próprio | Comunicação e agregação   |

Essa abordagem reduz o acoplamento entre os serviços e permite que cada microserviço gerencie seus próprios dados.

## 8. Segurança e autenticação

A autenticação do sistema utiliza JSON Web Token (JWT).

O microserviço de Usuários é responsável por validar as credenciais e emitir os tokens utilizados na aplicação.

As senhas são protegidas utilizando bcrypt, evitando o armazenamento de senhas em texto puro.

### Perfis de acesso

- ADMIN
- ATENDENTE
- CLIENTE

### Medidas de segurança

- Autenticação JWT.
- Autorização baseada em perfis.
- Proteção de senhas com bcrypt.
- Validação de tokens nos serviços.
- Helmet para cabeçalhos de segurança.
- CORS.
- Configurações sensíveis armazenadas em variáveis de ambiente.

## 9. Execução local

### Pré-requisitos

- Node.js 22
- npm
- Git
- Docker, quando necessário
- Acesso aos bancos de dados configurados
- Azure Functions Core Tools
- Azurite para execução local da Azure Function, quando necessário

### Clonar o repositório

```bash
git clone --recurse-submodules https://github.com/StefanyCarlos23/Help-Flow.git
```

Caso o repositório já tenha sido clonado sem os submódulos:

```bash
git submodule update --init --recursive
```

### Configurar os componentes

Cada componente possui seu próprio arquivo `.env` ou arquivo de configuração local.

Configure as variáveis necessárias de acordo com o README de cada repositório.

Não publique arquivos `.env`, `local.settings.json`, senhas, tokens ou chaves de autenticação.

### Ordem recomendada de inicialização

Para executar o sistema localmente, recomenda-se iniciar os componentes nesta ordem:

1. Microserviço de Usuários — porta `3002`.
2. Microserviço de Chamados — porta `3001`.
3. BFF — porta `3000`.
4. Microfrontend.
5. Azure Function, quando necessário para os indicadores.

Cada componente deve ter suas dependências instaladas e configurações de ambiente ajustadas antes da execução.

Consulte o README individual de cada componente para obter os comandos específicos de inicialização.

## 10. Docker e implantação

Os componentes do Help Flow possuem suporte à execução em contêineres por meio de Docker.

As imagens podem ser construídas individualmente utilizando os Dockerfiles presentes nos respectivos repositórios.

A comunicação entre os contêineres exige uma configuração de rede adequada, permitindo que o BFF alcance os microserviços.

Quando executados em uma rede Docker compartilhada, os serviços devem utilizar os nomes dos contêineres ou serviços como endereços de comunicação.

A Azure Function possui implantação independente no Microsoft Azure, utilizando o plano Flex Consumption e a região Brazil South.

## 11. Documentação e testes

Os componentes backend possuem documentação e testes próprios.

| Componente               | Documentação         | Testes                                      |
| ------------------------ | -------------------- | ------------------------------------------- |
| BFF                      | Swagger / OpenAPI    | Jest e Supertest                            |
| Microserviço de Chamados | Swagger UI           | Jest e Supertest                            |
| Microserviço de Usuários | Swagger UI           | Jest e Supertest                            |
| Azure Function           | Endpoint HTTP        | Testes realizados durante o desenvolvimento |
| Microfrontend            | README do componente | Conforme implementação                      |

### Documentação Swagger

As interfaces Swagger dos serviços backend podem ser acessadas localmente:

| Serviço  | Endereço                            |
| -------- | ----------------------------------- |
| BFF      | http://localhost:3000/api-docs      |
| Chamados | http://localhost:3001/api-docs      |
| Usuários | Configurado no próprio microserviço |

## 12. Padrões arquiteturais

O Help Flow foi desenvolvido utilizando diferentes padrões e conceitos de arquitetura de software.

### Microservices

Separação das funcionalidades em serviços independentes, cada um com suas próprias responsabilidades.

### Backend for Frontend (BFF)

Camada intermediária responsável por adaptar e consolidar as informações disponibilizadas pelos serviços para o frontend.

### Microfrontend

Organização da interface de usuário em uma aplicação frontend independente.

### Clean Architecture

Separação das responsabilidades entre API, Application, Domain e Infrastructure nos microserviços.

### Vertical Slice Architecture

Organização das funcionalidades por casos de uso, facilitando a manutenção e evolução dos componentes.

### Database per Service

Cada microserviço mantém seu próprio banco de dados, evitando dependências diretas entre as estruturas de persistência.

### Serverless Computing

Utilização de Azure Functions para executar o processamento de indicadores sob demanda.

### Event-Driven Architecture

Conceito arquitetural contemplado na proposta do projeto para integração entre componentes distribuídos.

## 13. Contexto acadêmico

Projeto desenvolvido para a disciplina de Arquitetura de Soluções Cloud da Pontifícia Universidade Católica do Paraná (PUCPR).

O objetivo é desenvolver e demonstrar uma aplicação distribuída utilizando microserviços, BFF, microfrontend, bancos de dados independentes e serviços Serverless do Microsoft Azure.

A solução busca aplicar conceitos de arquitetura moderna, separação de responsabilidades, escalabilidade e integração entre serviços.
