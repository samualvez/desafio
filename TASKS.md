# TASKS.md — Roteiro de implementação

> Regra de trabalho: as tasks são entregues **uma por vez**, em ordem. Cada task tem entre 10 e 30 minutos, com teoria completa antes do código, passo a passo, código comentado linha por linha, forma de testar, critérios de aceite, erros comuns e sugestão de commit (Conventional Commits).
>
> Este arquivo é o **índice/checklist**. O conteúdo detalhado de cada task é entregue no chat, uma a uma, conforme avançamos.

## FASE 1 — Planejamento
- [x] 1. Criar SPEC.md
- [x] 2. Criar TASKS.md

## FASE 2 — Backend

- [ ] 3. Criar projeto Spring Boot (Spring Initializr, dependências, import no IntelliJ)
- [ ] 4. Entender a estrutura do projeto gerado (pom.xml, Application.java, resources)
- [ ] 5. Configurar H2 (application.properties, console H2, entender banco em memória)
- [ ] 6. Criar Entity Cliente (JPA, anotações @Entity, @Id, @GeneratedValue)
- [ ] 7. Criar Entity Tarefa + Enum StatusTarefa
- [ ] 8. Criar relacionamento @ManyToOne / @OneToMany entre Tarefa e Cliente
- [ ] 9. Criar ClienteRepository e TarefaRepository (Spring Data JPA)
- [ ] 10. Popular dados iniciais (data.sql) e validar no console H2
- [ ] 11. Criar DTOs (ClienteDTO, TarefaDTO, CriarTarefaDTO, AlterarStatusDTO)
- [ ] 12. Criar ClienteService (regra de negócio: buscar cliente ou lançar erro)
- [ ] 13. Criar TarefaService (criar tarefa, listar por cliente, alterar status)
- [ ] 14. Criar ClienteController (GET /api/clientes)
- [ ] 15. Criar TarefaController (GET/POST tarefas por cliente, PATCH status)
- [ ] 16. Criar tratamento de erros global (ResourceNotFoundException + GlobalExceptionHandler)
- [ ] 17. Testar todos os endpoints manualmente (Postman/Insomnia/navegador)
- [ ] 18. Configurar CORS para o frontend acessar a API

## FASE 3 — Frontend

- [ ] 19. Criar index.html (estrutura básica da página)
- [ ] 20. Criar style.css simples
- [ ] 21. Criar app.js e entender a Fetch API (teoria: fetch, Promise, async/await, JSON)
- [ ] 22. Buscar e exibir lista de clientes (GET /api/clientes)
- [ ] 23. Ao clicar em um cliente, buscar e exibir suas tarefas (GET /api/clientes/{id}/tarefas)
- [ ] 24. Criar formulário para nova tarefa e consumir POST /api/clientes/{id}/tarefas
- [ ] 25. Implementar botão/select para alterar status (PATCH /api/tarefas/{id}/status)
- [ ] 26. Atualizar a tela automaticamente após criar tarefa ou mudar status (sem reload manual)

## FASE 4 — Finalização

- [ ] 27. Criar README.md (como rodar o projeto, decisões, prints)
- [ ] 28. Criar PROMPTS.md (registro de como a IA foi usada no desafio)
- [ ] 29. Revisão geral da arquitetura (checklist de boas práticas)
- [ ] 30. Melhorias opcionais (excluir tarefa, filtro por status, validações extras, testes automatizados)

---

**Próximo passo:** começar a Task 3 — Criar projeto Spring Boot. Me avise quando quiser seguir e eu entrego o conteúdo completo dessa task (teoria + passo a passo + código + testes).
