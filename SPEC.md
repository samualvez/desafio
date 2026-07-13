# SPEC.md — Sistema de Gerenciamento de Tarefas

## 1. Entendimento do problema

Precisamos construir um sistema onde:

- Existem **Clientes**.
- Cada Cliente pode ter **várias Tarefas** (relação um-para-muitos).
- Um usuário (via frontend simples) deve conseguir:
  1. Ver a lista de clientes.
  2. Ao selecionar um cliente, ver as tarefas dele.
  3. Criar uma nova tarefa para um cliente.
  4. Alterar o status de uma tarefa (ex: PENDENTE → CONCLUÍDA).

O foco do desafio **não é** ter uma UI bonita ou um domínio complexo. O foco é:

- Backend REST bem estruturado (camadas Controller/Service/Repository).
- Persistência com JPA/Hibernate sobre H2 (banco em memória).
- Frontend simples em HTML/CSS/JS puro consumindo a API via `fetch`.
- Capacidade de **explicar** cada decisão técnica na entrevista.

Como é um desafio técnico (não um cliente real com quem eu possa conversar), as "perguntas para o cliente" abaixo são registradas como perguntas que eu faria, e logo depois assumo uma resposta razoável (suposição), para não travar o projeto.

## 2. Perguntas para o cliente

| # | Pergunta | Suposição assumida (ver seção 3) |
|---|----------|-----------------------------------|
| 1 | Tarefas têm quais status possíveis? Só "pendente/concluída" ou mais estados (ex: em andamento, cancelada)? | 3 estados: `PENDENTE`, `EM_ANDAMENTO`, `CONCLUIDA` |
| 2 | Clientes podem ser criados/editados pelo usuário, ou já vêm pré-cadastrados (seed)? | Clientes vêm pré-cadastrados via seed (`data.sql`), sem CRUD de cliente nesta fase |
| 3 | Tarefas podem ser excluídas? | Não é obrigatório no desafio; deixado como melhoria opcional (Fase 4) |
| 4 | Precisa de autenticação/login? | Não. Fora do escopo do desafio |
| 5 | Precisa de paginação de tarefas/clientes? | Não, volume de dados é pequeno (dados de exemplo) |
| 6 | O campo "descrição" da tarefa é obrigatório? Tem limite de tamanho? | Obrigatório, até 255 caracteres |
| 7 | Precisa de data de vencimento (deadline) na tarefa? | Não é obrigatório; pode ser melhoria opcional |
| 8 | O frontend precisa ser responsivo/mobile? | Não é prioridade; CSS simples e legível já basta |

## 3. Suposições

1. Cada Tarefa pertence a exatamente um Cliente (obrigatório, não pode existir tarefa "solta").
2. Status da tarefa é um enum fixo: `PENDENTE`, `EM_ANDAMENTO`, `CONCLUIDA`.
3. Toda tarefa nasce com status `PENDENTE`.
4. Não há exclusão de cliente nem de tarefa nesta primeira versão (poderia ser melhoria opcional).
5. Banco de dados é H2 em memória — os dados são perdidos ao reiniciar a aplicação (aceitável para um desafio).
6. Não há autenticação — API pública localmente, sem controle de acesso.
7. O frontend é servido como arquivos estáticos simples (abertos direto no navegador ou servidos pelo próprio Spring Boot via `src/main/resources/static`).

## 4. Requisitos Funcionais (RF)

- **RF01** — O sistema deve listar todos os clientes cadastrados.
- **RF02** — O sistema deve listar todas as tarefas de um cliente específico.
- **RF03** — O sistema deve permitir criar uma nova tarefa vinculada a um cliente.
- **RF04** — O sistema deve permitir alterar o status de uma tarefa existente.
- **RF05** — O sistema deve validar que uma tarefa só pode ser criada para um cliente existente.
- **RF06** — O sistema deve retornar erros HTTP claros quando um recurso não existir (404) ou quando os dados enviados forem inválidos (400).

## 5. Requisitos Não Funcionais (RNF)

- **RNF01** — API deve seguir convenções REST (verbos HTTP corretos, status codes corretos, recursos no plural).
- **RNF02** — Código organizado em camadas: `Controller` → `Service` → `Repository`, com `Entity` e `DTO` separados.
- **RNF03** — Comunicação Frontend↔Backend em JSON.
- **RNF04** — CORS habilitado para permitir que o frontend (servido separadamente, se necessário) acesse a API.
- **RNF05** — Aplicação deve subir localmente sem configuração manual de banco (H2 em memória, auto-configurado pelo Spring Boot).
- **RNF06** — Código deve ser simples e legível, priorizando clareza didática sobre otimizações prematuras.
- **RNF07** — Sem frameworks de frontend (somente HTML/CSS/JS puro + Fetch API).

## 6. Arquitetura

Arquitetura em camadas (Layered Architecture), padrão típico de aplicações Spring Boot monolíticas:

```
┌─────────────────────────────┐
│   Frontend (HTML/CSS/JS)    │  → fetch() faz chamadas HTTP
└──────────────┬───────────────┘
               │ JSON via HTTP
┌──────────────▼───────────────┐
│      Controller (REST)       │  → recebe requisições, valida entrada, devolve DTO
├──────────────┬───────────────┤
│         Service               │  → regras de negócio (ex: "tarefa só pode existir se cliente existir")
├──────────────┬───────────────┤
│       Repository (JPA)        │  → acesso a dados, sem SQL manual
├──────────────┬───────────────┤
│         Entity (JPA)          │  → mapeamento objeto-relacional
└──────────────┬───────────────┘
               │
        H2 Database (em memória)
```

**Por que essa arquitetura?**
- Separar responsabilidades facilita testes, manutenção e é o padrão esperado em entrevistas de vaga Full Stack Java.
- `Controller` não deve conter lógica de negócio — só orquestra request/response.
- `Service` concentra regras de negócio, podendo ser testado sem depender de HTTP.
- `Repository` isola o acesso a dados — troca de banco não afeta as camadas superiores.
- `DTO` (Data Transfer Object) evita expor a Entity diretamente na API, prevenindo problemas de serialização (ex: referências circulares no relacionamento) e desacoplando o "formato de banco" do "formato de API".

## 7. Modelo de domínio

### Entidades

**Cliente**
| Campo | Tipo | Regras |
|-------|------|--------|
| id | Long | gerado automaticamente (PK) |
| nome | String | obrigatório |
| email | String | obrigatório |

**Tarefa**
| Campo | Tipo | Regras |
|-------|------|--------|
| id | Long | gerado automaticamente (PK) |
| descricao | String | obrigatório, até 255 caracteres |
| status | Enum (`PENDENTE`, `EM_ANDAMENTO`, `CONCLUIDA`) | obrigatório, default `PENDENTE` |
| cliente | Cliente | obrigatório (FK — chave estrangeira) |

### Relacionamento

- **Cliente 1 — N Tarefa** (um cliente tem várias tarefas; uma tarefa pertence a um único cliente).
- No banco relacional: tabela `tarefa` possui uma coluna `cliente_id` (FK) apontando para `cliente.id`.
- No JPA: `@OneToMany` no lado `Cliente` (opcional, lado "inverso") e `@ManyToOne` no lado `Tarefa` (lado "dono" do relacionamento, que guarda a FK).

## 8. Endpoints REST

| Método | Endpoint | Descrição | Corpo (request) | Resposta |
|--------|----------|-----------|------------------|----------|
| GET | `/api/clientes` | Lista todos os clientes | — | `200 OK` + lista de `ClienteDTO` |
| GET | `/api/clientes/{clienteId}/tarefas` | Lista tarefas de um cliente | — | `200 OK` + lista de `TarefaDTO` (ou `404` se cliente não existir) |
| POST | `/api/clientes/{clienteId}/tarefas` | Cria tarefa para um cliente | `{ "descricao": "..." }` | `201 Created` + `TarefaDTO` (ou `404`/`400`) |
| PATCH | `/api/tarefas/{tarefaId}/status` | Altera status da tarefa | `{ "status": "CONCLUIDA" }` | `200 OK` + `TarefaDTO` (ou `404`/`400`) |

**Justificativas de design REST:**
- Recursos aninhados (`/clientes/{id}/tarefas`) porque tarefa não existe sem cliente — expressa a hierarquia de dependência na própria URL.
- `POST` para criação (não idempotente, cria novo recurso).
- `PATCH` (não `PUT`) para alterar status, porque estamos atualizando **parcialmente** o recurso (só o status), não substituindo a tarefa inteira.
- Tarefa recebe rota própria (`/api/tarefas/{id}/status`) fora do aninhamento de cliente, pois a alteração de status não depende do cliente pai — simplifica a URL.

## 9. Estrutura de pastas

```
desafio/
├── SPEC.md
├── TASKS.md
├── PROMPTS.md
├── README.md
├── backend/
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/com/desafio/tarefas/
│       │   │   ├── TarefasApplication.java
│       │   │   ├── cliente/
│       │   │   │   ├── Cliente.java              (Entity)
│       │   │   │   ├── ClienteRepository.java
│       │   │   │   ├── ClienteService.java
│       │   │   │   ├── ClienteController.java
│       │   │   │   └── ClienteDTO.java
│       │   │   ├── tarefa/
│       │   │   │   ├── Tarefa.java                (Entity)
│       │   │   │   ├── StatusTarefa.java          (Enum)
│       │   │   │   ├── TarefaRepository.java
│       │   │   │   ├── TarefaService.java
│       │   │   │   ├── TarefaController.java
│       │   │   │   ├── TarefaDTO.java
│       │   │   │   ├── CriarTarefaDTO.java
│       │   │   │   └── AlterarStatusDTO.java
│       │   │   └── common/
│       │   │       ├── ResourceNotFoundException.java
│       │   │       └── GlobalExceptionHandler.java
│       │   └── resources/
│       │       ├── application.properties
│       │       └── data.sql
│       └── test/  (opcional, melhoria futura)
└── frontend/
    ├── index.html
    ├── style.css
    └── app.js
```

**Por que pacotes por funcionalidade (`cliente/`, `tarefa/`) em vez de por camada (`controllers/`, `services/`)?**
- É a abordagem mais usada em projetos reais modernos ("package by feature"): tudo relacionado a "tarefa" fica junto, facilitando localizar e evoluir o código.
- Alternativa (pacotes por camada) é mais didática para projetos pequenos e muito comum em tutoriais — vamos usar por feature porque escala melhor e é o que se espera em entrevistas mais avançadas, mas isso será explicado e comparado na task correspondente.

## 10. Tecnologias escolhidas

| Tecnologia | Papel | Justificativa |
|------------|-------|----------------|
| Java 21 | Linguagem | LTS mais recente, requisito do desafio |
| Spring Boot | Framework backend | Padrão de mercado para APIs Java, configuração automática (auto-configuration) |
| Spring Web | Camada REST | Fornece `@RestController`, serialização JSON automática via Jackson |
| Spring Data JPA | Persistência | Abstrai SQL repetitivo (CRUD) via interfaces de Repository |
| Hibernate | Implementação JPA | Vem embutido no Spring Data JPA, traduz objetos Java em SQL |
| H2 Database | Banco de dados | Banco em memória, zero configuração, ideal para desafios/testes |
| Maven | Build tool | Gerencia dependências e ciclo de build, padrão do ecossistema Java |
| HTML/CSS/JS puro | Frontend | Exigência do desafio — foco em entender Fetch API sem abstrações de framework |

## 11. Justificativas técnicas gerais

- **DTO em vez de expor Entity direto na API**: evita vazar detalhes internos do banco (como o relacionamento bidirecional, que causaria loop infinito na serialização JSON: Cliente → Tarefas → Cliente → Tarefas...). Também permite moldar exatamente o que a API expõe, independente da estrutura da tabela.
- **H2 em memória**: elimina a necessidade de instalar/configurar um banco externo (Postgres, MySQL) só para o desafio, mantendo o foco no aprendizado de JPA/REST.
- **Tratamento de erros centralizado (`GlobalExceptionHandler`)**: evita repetir blocos try/catch em cada controller; usa `@ExceptionHandler` do Spring para converter exceções em respostas HTTP padronizadas.
- **Enum para status**: garante que só existam status válidos (não aceita string arbitrária), tanto em Java quanto (opcionalmente) no banco.
- **Sem framework de frontend**: atende exigência do desafio e força o entendimento manual do ciclo `fetch → JSON → DOM`, que é justamente o que o desafio quer avaliar.

---

Próximo passo: revisar este documento comigo (o usuário) e, após aprovação, seguir para o `TASKS.md`.
