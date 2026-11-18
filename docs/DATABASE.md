# Aion - Database

> Beyond the Ordinary

## Objetivo

Este documento descreve a arquitetura do banco de dados utilizada pelo Aion.

Seu objetivo é documentar as decisões relacionadas à persistência de dados, modelagem das entidades, convenções adotadas e boas práticas utilizadas durante o desenvolvimento.

Este documento complementa o `DOMAIN.md`, descrevendo **como o domínio é representado no banco de dados**.

---

# Tecnologias

O projeto utiliza as seguintes tecnologias para persistência de dados:

- PostgreSQL
- Prisma ORM

O PostgreSQL é responsável pelo armazenamento das informações.

O Prisma atua como camada de acesso ao banco de dados, abstraindo consultas SQL e fornecendo tipagem forte para toda a aplicação.

---

# Objetivos da Modelagem

A modelagem do banco considera os seguintes princípios:

- Integridade referencial.
- Normalização dos dados.
- Facilidade de manutenção.
- Escalabilidade.
- Clareza nos relacionamentos.
- Compatibilidade com Prisma ORM.

Sempre que possível, evitar duplicação de informações.

---

# Convenções

## Identificadores

As entidades utilizam UUID como chave primária.

Exemplo:

```text
id
```

O uso de UUID evita colisões entre registros e facilita futuras integrações.

---

## Datas

Como convenção, as entidades persistentes devem possuir, quando aplicável:

```text
createdAt
updatedAt
```

Esses campos permitem o acompanhamento básico da criação e atualização dos registros.

---

## Nomenclatura

Utilizar nomes claros e consistentes.

Exemplos:

- usuario
- local
- ponto
- escala
- agendamento

Evitar abreviações desnecessárias.

---

# Entidades

A modelagem contempla as seguintes entidades e estruturas principais. Essa relação não é definitiva e poderá ser ampliada conforme a evolução do sistema.

## Usuario

Responsável pela autenticação e identificação dentro do sistema.

Possui um nível de permissão representado pelo enum `Role`.

Relacionamentos principais:

- Possui um nível de permissão.
- Pode ser responsável por Agendamentos.
- Pode participar de Agendamentos.
- Pode receber convites para participar de Agendamentos.

---

## Role

Representa os níveis de permissão disponíveis no sistema.

É definido como um enum no `schema.prisma`, não como uma entidade independente.

Valores atualmente suportados:

- PUBLICADOR
- SERVO
- ANCIAO
- ROOT

Cada usuário possui exatamente um nível de permissão.

---

## Local

Representa uma localização geográfica geral utilizada para a realização das atividades do sistema.

Exemplo: uma praça.

Relacionamentos:

- Possui vários Pontos.

---

## Ponto

Representa um ponto físico específico pertencente a um Local.

Exemplo: Entrada principal.

O Ponto representa uma localização mais específica dentro do Local, não apenas uma descrição textual.

Relacionamentos:

- Pertence a um Local.
- Possui exatamente uma Escala associada.

---

## Escala

Representa a estrutura de disponibilidade de agendamento associada a um Ponto.

Contém as datas e os horários disponíveis para que os usuários possam realizar Agendamentos.

Relacionamentos:

- Está associada a um único Ponto.
- Organiza várias Disponibilidades.

Cada Ponto possui sua própria Escala. Não há necessidade de múltiplas Escalas para um mesmo Ponto na primeira versão.

A possibilidade de diferentes tipos de Escala, como escalas recorrentes, fica reservada para releases futuras.

---

## Disponibilidade

Representa uma data e um intervalo de horário disponibilizados dentro de uma Escala.

Exemplo:

- Data: 14/11/2026.
- Horário: 09:00 às 11:00.

Relacionamentos:

- Pertence a uma Escala.
- Pode possuir um Agendamento associado.

As disponibilidades serão cadastradas individualmente. O planejamento poderá acontecer semanalmente ou quinzenalmente, mas essa periodicidade ainda não será uma regra fixa do sistema.

A implementação de recorrência não faz parte do escopo inicial.

Uma Disponibilidade que já possui um Agendamento não poderá receber outra reserva.

---

## Agendamento

Representa uma reserva realizada por um usuário a partir de uma Disponibilidade existente em uma Escala.

Relacionamentos:

- Está associado a uma Disponibilidade.
- Possui um usuário responsável.
- Pode possuir outros usuários participantes.
- Pode possuir convites associados.

O usuário responsável também conta como participante do Agendamento.

Cada Agendamento deve possuir:

- Mínimo de 2 participantes no total.
- Máximo de 3 participantes no total.

A entrada de outros usuários acontece por meio de convites enviados pelo responsável. O usuário convidado precisa aceitar o convite para integrar o Agendamento.

Uma Disponibilidade que já possui um Agendamento fica indisponível para novas reservas.

---

## Participantes

Representam os usuários associados a um Agendamento.

A relação deve permitir identificar quais usuários participam de cada Agendamento, incluindo o responsável.

A estrutura deve permitir controlar a quantidade total de participantes e respeitar os limites definidos para o Agendamento.

---

## Convites

Representam solicitações enviadas pelo responsável de um Agendamento para que outros usuários participem.

Os convites devem permitir identificar:

- O Agendamento relacionado.
- O usuário convidado.
- O estado do convite.

Um convite pendente não deve ser considerado uma participação confirmada.

A estrutura deve respeitar o limite máximo de participantes do Agendamento.

---

# Relacionamentos

Modelo conceitual simplificado dos principais relacionamentos:

```text
Usuario
  │
  ├── Role (enum)
  │
  ├── Responsável por Agendamentos
  │
  ├── Participante de Agendamentos
  │
  └── Recebe Convites
              │
              ▼
          Agendamento
              │
              ▼
        Disponibilidade
              │
              ▼
            Escala
              │
              ▼
             Ponto
              │
              ▼
             Local
```

O modelo representa os conceitos e suas associações principais. A implementação física dos relacionamentos, incluindo cardinalidades e tabelas auxiliares, deve ser definida no `schema.prisma`.

---

# Integridade Referencial

Todos os relacionamentos devem utilizar Foreign Keys.

O banco de dados é responsável por impedir referências inválidas.

O backend nunca deve depender apenas de validações em código para garantir consistência.

Também devem ser consideradas as seguintes regras:

- Uma Disponibilidade não deve possuir mais de um Agendamento ativo.
- Um Agendamento deve respeitar os limites de participantes.
- Um usuário não deve ser associado duplicadamente ao mesmo Agendamento.
- Convites e participações devem manter relacionamentos consistentes.

---

# Prisma ORM

O Prisma é a única tecnologia autorizada para acesso ao banco.

Não utilizar consultas SQL diretamente nos Services ou Controllers.

Toda comunicação deve ocorrer através dos Repositories.

---

# Migrações

Alterações estruturais devem seguir o fluxo:

1. Atualizar `schema.prisma`.
2. Validar a modelagem.
3. Criar e executar a migration.
4. Atualizar a documentação, quando necessário.

Nunca alterar o banco manualmente em produção.

---

# Índices

Índices devem ser criados apenas quando houver necessidade comprovada.

Priorizar:

- Chaves estrangeiras.
- Campos frequentemente utilizados em filtros.
- Campos utilizados em ordenações.
- Campos utilizados para consultar disponibilidades e agendamentos.

Evitar excesso de índices, pois impactam operações de escrita.

---

# Exclusão de Dados

Por padrão, exclusões devem preservar a integridade referencial.

Sempre avaliar se a exclusão física é realmente necessária.

O sistema prevê a utilização de Soft Delete para entidades que necessitem preservar histórico e integridade das informações.

A implementação e as regras específicas devem considerar as necessidades de cada entidade.

## Limpeza periódica

O sistema deverá possuir uma rotina automatizada de limpeza de dados, executada trimestralmente.

A rotina deverá identificar registros cuja referência temporal seja igual ou superior a três meses de antiguidade, considerando a data atual como referência.

Dados operacionais que já não possuem utilidade após esse período poderão ser removidos definitivamente através de Hard Delete.

Entre os registros sujeitos à limpeza estão:

- Agendamentos antigos, cuja data de realização ultrapassou o período de retenção.
- Disponibilidades antigas que já não possuem utilidade operacional.
- Convites e registros dependentes de Agendamentos removidos, quando não houver necessidade de preservação.
- Logs técnicos antigos.

Exemplo:

Se um Agendamento estava marcado para 14/07 e a data atual é 14/10, ele já atingiu o período de três meses e poderá ser removido definitivamente.

A exclusão deverá respeitar a integridade referencial, tratando os registros dependentes antes da remoção do Agendamento.

A rotina não deverá remover a Escala associada ao Ponto, pois ela representa uma estrutura permanente do sistema.

A limpeza periódica deve atuar sobre os dados operacionais antigos, e não sobre as estruturas permanentes do domínio.

A retenção de três meses representa a política inicial do projeto e poderá ser revisada conforme as necessidades do sistema evoluírem.

---

# Auditoria

Inicialmente, o sistema utiliza auditoria básica através dos campos:

- `createdAt`
- `updatedAt`

Funcionalidades futuras poderão ampliar os mecanismos de auditoria, incluindo:

- Usuário responsável pela alteração.
- Histórico de mudanças.
- Logs administrativos.

Os registros de auditoria e os logs técnicos devem ser considerados separadamente, pois podem possuir necessidades diferentes de retenção.

---

# Boas Práticas

Ao criar novas entidades:

- Utilizar UUID como identificador.
- Definir relacionamentos explicitamente.
- Utilizar nomes consistentes.
- Evitar redundância.
- Documentar alterações relevantes.
- Avaliar a necessidade de Soft Delete ou Hard Delete.
- Preservar a integridade referencial.

---

# Evolução

A modelagem do banco deverá evoluir conforme novas funcionalidades forem incorporadas ao sistema.

Novas entidades, enums, atributos e relacionamentos poderão ser adicionados conforme as necessidades do domínio.

Toda alteração estrutural deve ser refletida neste documento e no `DOMAIN.md`.

---

# Documentos Relacionados

- `DOMAIN.md`
- `ARCHITECTURE.md`
- `SECURITY.md`