# Contexto do Projeto — API REST de Gestão de Treinamentos (F`****`G`*****`)

> Documento de onboarding e referência arquitetural. Antes de propor ou implementar uma mudança, valide-a contra estas decisões.

## Objetivo

Construir uma API REST para gestão de treinamentos corporativos, com uma arquitetura sólida, escalável e fácil de manter. A qualidade e a coerência arquitetural têm prioridade sobre a velocidade inicial de desenvolvimento.

O projeto evolui incrementalmente: decisões arquiteturais devem ser discutidas e validadas antes da implementação.

## Stack

- Backend: TypeScript, Node.js e Express.
- Banco de dados: MySQL e Prisma ORM.
- Validação: Zod.
- Documentação: OpenAPI e Swagger UI.
- Logs: Pino e, posteriormente, `pino-http`.

### Documentação de API

Zod é a fonte única da verdade. O fluxo é:

```text
Zod → OpenAPI → Swagger UI
```

- Não utilizar `swagger-jsdoc`.
- Utilizar `@asteasolutions/zod-to-openapi` e manter `swagger-ui-express`.
- Centralizar a configuração em `src/config/openapi.ts`.

## Filosofia arquitetural

O projeto usa **DDD Light**:

- O domínio guia a arquitetura, os casos de uso prevalecem sobre CRUD e o código usa a linguagem do negócio.
- A organização é feita por features/módulos de domínio.
- Não usar DDD completo, Aggregates, Value Objects, Domain Events nem Factories complexas.
- Não criar classes CRUD genéricas nem abstrações prematuras; preferir composição a herança.

## Estrutura geral

```text
src/
  app.ts
  server.ts
  config/
  core/
  modules/
  routes/
  shared/
```

- `config/`: configurações, como `env.ts` e `openapi.ts`.
- `core/`: infraestrutura, como logger, erros, middlewares, HTTP e validação.
- `modules/`: conceitos do domínio, como `usuarios`, `treinamentos`, `instrutores` e `auth`.

## Convenções

- Pastas e arquivos: `kebab-case`.
- Classes: `PascalCase`.
- Funções e variáveis: `camelCase`.
- Constantes: `UPPER_SNAKE_CASE`.
- Usar exclusivamente o alias `@`, por exemplo: `import { env } from '@/config/env';`.
- Não criar aliases específicos como `@config`, `@core` ou `@modules`.

## TypeScript

Configurações adotadas:

```text
strict = true
exactOptionalPropertyTypes = true
noUncheckedIndexedAccess = true
noImplicitOverride = true
module = NodeNext
moduleResolution = NodeNext
target = ES2023
```

O objetivo é detectar erros o mais cedo possível.

## Módulos e casos de uso

Cada módulo é organizado por responsabilidade e cada caso de uso tem seu próprio service:

```text
usuarios/
  usuario.controller.ts
  usuario.repository.ts
  usuario.routes.ts
  usuario.schema.ts
  services/
    create-user.service.ts
    update-user.service.ts
    deactivate-user.service.ts
```

Não criar um `usuario.service.ts` abrangente. Um arquivo deve ter uma responsabilidade e um módulo deve representar um conceito do domínio.

## Camadas

O Prisma deve permanecer encapsulado. Services não acessam Prisma diretamente:

```text
Controller → Service → Repository → Prisma
```

- Controllers são finos: recebem a requisição, validam, chamam o service e retornam a resposta.
- Regras de negócio pertencem aos services, que representam casos de uso, como `CreateTrainingService`, `FinishTrainingService`, `LoginService` e `CreateUserService`.
- Schemas Zod ficam em arquivos `*.schema.ts`, nunca dentro de controllers.
- Não usar DTOs: os tipos são inferidos dos schemas Zod, por exemplo `z.infer<typeof createUserSchema>`.

## Ambiente, logs e erros

### Variáveis de ambiente

`src/config/env.ts` é o único ponto de acesso. Nunca usar `process.env` diretamente. A validação é feita com Zod e a aplicação deve falhar na inicialização se faltar uma variável obrigatória.

### Logger

Não usar `console.log`. Os logs devem ser estruturados com Pino e adequados a produção e observabilidade.

```text
core/logger/
  options.ts      # monta a configuração do Pino
  index.ts        # cria e exporta a instância do logger
  http-logger.ts  # registra requisições HTTP automaticamente
```

### Erros

Fluxo previsto:

```text
Controller → asyncHandler → Service → throw AppError → Error Middleware → logger → resposta HTTP
```

Cada erro deve ser registrado uma única vez.

## Respostas de API

Sucesso:

```json
{ "success": true, "data": {}, "meta": {} }
```

Erro:

```json
{ "success": false, "error": { "code": "...", "message": "..." } }
```

## Qualidade e forma de trabalho

Testes fazem parte da arquitetura. Para cada funcionalidade: implementar, escrever testes, executá-los e só então considerar a tarefa concluída. TDD não é obrigatório; testes contínuos são.

Em toda decisão, responder:

- Por que esta decisão foi tomada?
- Qual problema ela resolve?
- Como contribui para a consistência arquitetural?

## Estado atual e próximos passos

Concluídos: definição da stack e DDD Light, estrutura inicial, TypeScript, ESLint/Prettier, Zod como fonte única da verdade, OpenAPI/Swagger UI, `env.ts` e a arquitetura de logger Pino.

Próximas etapas: finalizar Pino e `pino-http`; implementar tratamento global de erros (`AppError`, `asyncHandler` e middleware); seguir com Prisma, OpenAPI e o primeiro módulo de negócio, `Auth`.
