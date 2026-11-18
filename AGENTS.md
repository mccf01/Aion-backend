# Aion - Instruções para Agentes de IA

> Beyond the Ordinary

## Objetivo

Este projeto é o backend do Aion, um sistema para gerenciamento de escalas de Testemunho Público.

O objetivo é manter uma arquitetura limpa, modular, segura e de fácil manutenção.

A IA deve priorizar qualidade, legibilidade, segurança e consistência arquitetural acima da velocidade de implementação.

Sempre respeite os padrões já existentes no projeto.

A identidade oficial do projeto é **Aion — Beyond the Ordinary**. Ao trabalhar com documentações institucionais ou elementos de apresentação do sistema, respeite essa identidade.

---

# Stack

## Backend

- Node.js
- TypeScript
- Express
- Prisma ORM
- PostgreSQL
- Docker
- Jest

## Banco de Dados

- PostgreSQL
- Prisma ORM

## Infraestrutura

- Docker Compose
- MailHog para testes de e-mail

---

# Arquitetura

O projeto utiliza arquitetura em camadas.

Toda requisição deve seguir obrigatoriamente o fluxo abaixo.

```text
Routes
   ↓
Middlewares
   ↓
Controllers
   ↓
Services
   ↓
Repositories
   ↓
Prisma ORM
   ↓
PostgreSQL
```

Nunca pule camadas.

Nunca acesse Prisma fora dos Repositories.

Nunca implemente regras de negócio nos Controllers.

---

# Controllers

Responsabilidades:

- Receber Request
- Extrair parâmetros
- Chamar Services
- Retornar Response

Controllers NÃO devem:

- acessar Prisma
- conter regras de negócio
- realizar autenticação
- realizar autorização
- conter consultas SQL
- acessar banco diretamente

Controllers devem permanecer pequenos e objetivos.

---

# Services

Toda regra de negócio pertence aos Services.

Exemplos:

- cadastro
- login
- recuperação de senha
- regras de recorrência
- geração de escalas
- validação de datas
- verificação de conflitos
- autorização baseada em RBAC, quando depender do contexto da operação
- prevenção de IDOR
- validação de propriedade dos recursos

Sempre que possível, Services devem ser independentes do Express.

---

# Repositories

Repositories são a única camada autorizada a acessar o Prisma.

Nunca utilize Prisma diretamente em:

- Controllers
- Services
- Middlewares

Repositories devem conter apenas acesso aos dados.

Não coloque regras de negócio ou decisões de autorização nesta camada.

---

# Middlewares

Cada Middleware deve possuir apenas uma responsabilidade.

Exemplos:

- auth.middleware.ts
- role.middleware.ts
- validation.middleware.ts
- error.middleware.ts
- rateLimit.middleware.ts

Responsabilidades relacionadas à segurança:

- autenticação das requisições
- verificação geral de permissões e papéis
- validação de entradas
- tratamento global de erros
- limitação de requisições

Regras de autorização que dependam do contexto da operação, como propriedade de recursos e prevenção de IDOR, devem ser verificadas nos Services.

Evite Middlewares específicos para entidades.

---

# Prisma

Utilizar Prisma apenas através dos Repositories.

Sempre utilizar tipagem forte.

Evitar consultas duplicadas.

Priorizar métodos reutilizáveis.

Sempre preferir recursos nativos do Prisma antes de escrever SQL manual.

---

# Segurança

Este projeto prioriza segurança desde a arquitetura.

Sempre considerar:

- JWT
- Cookies HttpOnly
- SameSite
- Secure
- bcrypt
- RBAC
- Least Privilege
- IDOR
- SQL Injection
- XSS
- CSRF
- Rate Limiting

Nunca remover validações existentes.

Nunca enfraquecer autenticação.

Nunca armazenar senha em texto puro.

Nunca expor stack traces em produção.

---

# RBAC

O sistema utiliza Role Based Access Control.

Atualmente existem quatro papéis.

## PUBLICADOR

Responsável pela utilização comum do sistema.

Pode:

- consultar suas escalas
- consultar programações
- atualizar seu próprio perfil
- confirmar participação
- cancelar participação quando permitido

Não pode acessar funcionalidades administrativas.

---

## SERVO

Possui todas as permissões do PUBLICADOR.

Na primeira release, possui as mesmas permissões operacionais do ANCIÃO para o gerenciamento de escalas.

Pode:

- criar escalas
- editar escalas
- remover escalas
- criar e editar usuários
- gerenciar locais
- executar tarefas administrativas delegadas

Não pode:

- remover ou desativar usuários
- acessar funcionalidades exclusivas de administração técnica do sistema

As permissões do SERVO poderão ser modificadas, ampliadas ou restringidas conforme a evolução do Aion e a implementação de novas funcionalidades.

---

## ANCIÃO

Possui todas as permissões do SERVO.

Além disso, pode:

- remover ou desativar usuários

Na primeira release, possui as mesmas permissões operacionais do SERVO para o gerenciamento de escalas e locais.

Funcionalidades administrativas mais complexas poderão ser disponibilizadas exclusivamente ao ANCIÃO conforme a evolução do sistema.

---

## ROOT

Possui todas as permissões do ANCIÃO, além de acesso às funcionalidades de administração técnica da plataforma.

Destina-se exclusivamente à administração técnica e ao gerenciamento completo do Aion.

Pode:

- acessar todas as funcionalidades do sistema
- acessar telas e recursos de administração técnica
- alterar parâmetros globais e configurações da plataforma
- gerenciar configurações relacionadas ao servidor por meio das funcionalidades administrativas disponibilizadas pelo sistema
- executar tarefas administrativas exclusivas
- ativar e desativar o modo de manutenção
- realizar operações administrativas independentemente das restrições aplicadas aos demais perfis

O ROOT representa uma exceção deliberada à hierarquia convencional de permissões do sistema.

O acesso técnico do ROOT deve ser restrito às funcionalidades administrativas implementadas e autorizadas pelo Aion. Isso não concede automaticamente acesso direto à infraestrutura, ao terminal, ao banco de dados ou às credenciais externas de hospedagem.

Operações críticas devem respeitar as boas práticas de segurança, validação, auditoria e proteção da disponibilidade do sistema.

A IA nunca deve conceder permissões de ROOT a outros usuários sem autorização explícita.

Para os demais papéis, sempre aplicar o princípio do menor privilégio.

---

## Evolução das permissões

As permissões descritas refletem as regras definidas para a primeira release do Aion.

Com a evolução do sistema e a implementação de funcionalidades mais complexas, as permissões dos perfis SERVO e ANCIÃO poderão ser diferenciadas, ampliadas ou restringidas conforme as necessidades operacionais e as decisões de negócio.

A IA não deve antecipar essas mudanças nem implementar restrições ou permissões futuras que ainda não tenham sido formalmente definidas.

---

# IDOR

Sempre verificar propriedade dos recursos.

Nunca confiar apenas no ID enviado pelo cliente.

Sempre validar:

- usuário autenticado
- role
- dono do recurso

PUBLICADOR acessa apenas recursos autorizados.

SERVO possui acesso administrativo limitado.

ANCIAO possui acesso administrativo completo sobre o domínio.

ROOT possui acesso aos recursos e funcionalidades do sistema.

As verificações de propriedade devem respeitar as permissões de cada papel e as regras de negócio existentes.

---

# Código

Priorizar:

- simplicidade
- reutilização
- legibilidade
- baixo acoplamento
- alta coesão

Evitar:

- código duplicado
- funções gigantes
- arquivos enormes
- comentários desnecessários
- lógica repetida

Preferir funções pequenas.

---

# TypeScript

Sempre utilizar tipagem explícita.

Evitar:

```typescript
any
```

Preferir:

- interfaces
- types
- enums

Utilizar strict mode.

---

# Organização

Cada módulo deve possuir estrutura semelhante.

```text
module/

controllers/
services/
repositories/
routes/
schemas/
types/
```

Manter organização consistente entre todos os módulos.

---

# Dependências

Antes de adicionar uma nova biblioteca:

- verificar se existe solução nativa
- verificar se o projeto já utiliza biblioteca equivalente
- evitar dependências desnecessárias

Não instalar bibliotecas sem necessidade.

---

# Banco de Dados

Nunca alterar modelos Prisma sem necessidade.

Caso seja necessário:

- atualizar schema.prisma
- executar prisma generate
- criar migration quando apropriado

Nunca apagar dados automaticamente.

---

# Qualidade

Antes de finalizar qualquer tarefa, executar as verificações de qualidade.

Executar:

```bash
npm run lint
npm run build
npm run test
```

Corrigir todos os erros de lint e compilação.

Sempre verificar:

- imports
- tipagem
- lint
- arquivos órfãos
- testes relacionados às alterações

Caso existam testes, garantir que sejam executados e aprovados antes de finalizar a tarefa.

---

# Docker

Assumir que o ambiente utiliza Docker Compose.

Caso os containers não estejam iniciados:

```bash
docker compose up -d
```

Banco utilizado:

- PostgreSQL

Ferramenta de e-mail:

- MailHog

Nunca assumir instalação local do PostgreSQL.

---

# Implementações

Ao criar novas funcionalidades:

1. Criar Repository
2. Criar Service
3. Criar Controller
4. Criar Routes
5. Criar validações
6. Atualizar documentação quando necessário

Sempre seguir o padrão arquitetural existente.

---

# Refatorações

Ao modificar código existente:

- preservar comportamento
- evitar breaking changes
- reutilizar código existente
- manter padrão arquitetural
- manter compatibilidade com os demais módulos

---

# Consistência Arquitetural

Ao implementar novas funcionalidades:

- respeitar a arquitetura existente
- reutilizar componentes antes de criar novos
- evitar duplicação de lógica
- manter nomenclaturas consistentes
- seguir os padrões adotados pelo projeto

Caso uma alteração estrutural significativa seja necessária, explicar o motivo antes de implementá-la.

Nunca introduzir um novo padrão quando já existir um padrão consolidado no projeto.

---

# Estilo

Priorizar código limpo.

Quando houver duas soluções equivalentes:

Escolher sempre a mais simples, mais legível e mais consistente com a arquitetura do projeto.

---

# Em caso de dúvida

Nunca invente regras de negócio.

Caso alguma informação esteja ausente ou ambígua:

- explique o problema
- proponha alternativas
- aguarde confirmação antes de realizar alterações estruturais

Nunca tome decisões arquiteturais importantes automaticamente.