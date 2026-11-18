# Aion - Domain Model

> Beyond the Ordinary

## Objetivo

Este documento descreve o domínio de negócio do Aion, apresentando os principais conceitos, entidades, relacionamentos, regras de negócio e fluxos que compõem o sistema.

Seu objetivo é servir como referência para desenvolvedores, colaboradores e agentes de IA, permitindo que todos compreendam não apenas como o sistema foi implementado, mas principalmente **o problema que ele resolve**.

Enquanto o documento **ARCHITECTURE.md** descreve a arquitetura técnica do backend, este documento descreve a arquitetura do negócio.

---

# Visão Geral

O Aion é um sistema destinado ao gerenciamento de escalas de Testemunho Público.

Seu propósito é permitir que os responsáveis pela organização das atividades administrem usuários, locais, pontos, disponibilidades e agendamentos de maneira simples, segura e organizada.

O sistema permite organizar os horários disponíveis em cada ponto e possibilita que os usuários realizem agendamentos, incluindo outros participantes por meio de convites.

O sistema foi projetado para reduzir processos manuais, minimizar conflitos de agendamento e facilitar o gerenciamento das atividades da congregação.

---

# Objetivos do Domínio

O sistema possui os seguintes objetivos principais:

- Centralizar todas as escalas em um único sistema.
- Organizar os pontos físicos onde ocorre o Testemunho Público.
- Gerenciar disponibilidades de datas e horários.
- Permitir que usuários realizem agendamentos.
- Facilitar a participação de outros usuários por meio de convites.
- Facilitar o gerenciamento de usuários e locais.
- Permitir controle de permissões administrativas.
- Reduzir conflitos e erros durante a realização de agendamentos.

---

# Glossário

## Usuário

Pessoa cadastrada no sistema.

Todo acesso aos recursos protegidos do sistema ocorre através de um usuário autenticado.

Cada usuário possui um papel (Role), responsável por determinar suas permissões dentro da aplicação.

Um usuário pode realizar agendamentos e participar de atividades por meio de convites.

---

## Publicador

Usuário responsável por participar das atividades de Testemunho Público.

Pode consultar as disponibilidades e realizar agendamentos, respeitando as regras estabelecidas pelo sistema.

Também pode participar de agendamentos para os quais tenha recebido e aceitado um convite.

Não possui permissões administrativas.

---

## Servo

Usuário com permissões administrativas intermediárias.

Seu papel é auxiliar na administração operacional do sistema, podendo gerenciar usuários, locais, pontos e escalas, incluindo o cadastro e a alteração das disponibilidades de datas e horários.

O SERVO pode realizar essas atividades conforme a organização e o alinhamento estabelecidos com os anciãos.

Não pode remover ou desativar usuários.

---

## Ancião

Administrador funcional do sistema.

Possui as permissões administrativas operacionais do SERVO.

Também pode remover ou desativar usuários e possui responsabilidade sobre o gerenciamento geral da operação.

---

## Root

Administrador técnico da plataforma.

Possui as permissões do ANCIÃO, além de funcionalidades exclusivas de administração técnica.

Seu papel está relacionado à administração e manutenção da plataforma, não à operação diária do Testemunho Público.

---

## Local

Representa uma localização geográfica onde ocorre o Testemunho Público.

Exemplos:

- Praça
- Parque
- Avenida
- Feira
- Orla
- Centro Comercial

Um Local pode possuir diversos Pontos.

---

## Ponto

Representa um local físico específico dentro de um Local.

Identifica o lugar exato onde a atividade será realizada.

Exemplos:

- Entrada principal de uma praça.
- Próximo a determinado acesso.
- Área específica de um parque.

Cada Ponto pertence a um único Local e possui exatamente uma Escala.

---

## Escala

Representa a estrutura de disponibilidades de um Ponto.

É responsável por organizar as datas e os intervalos de horários nos quais o Testemunho Público poderá ocorrer naquele Ponto.

Cada Escala pertence a um único Ponto e permanece associada a ele.

---

## Disponibilidade

Representa uma data e um intervalo de horário disponíveis dentro de uma Escala.

É a unidade utilizada para organizar os horários que podem receber agendamentos.

Uma Disponibilidade pode receber um Agendamento, respeitando as regras de disponibilidade do sistema.

---

## Agendamento

Representa a reserva de uma Disponibilidade por um usuário responsável.

O responsável pelo agendamento pode incluir outros participantes por meio de convites.

Cada Agendamento deve possuir no mínimo dois e no máximo três participantes, considerando o próprio responsável.

Uma Disponibilidade que já possui um Agendamento não pode receber outra reserva.

---

## Participante

Representa um usuário que fará parte de um Agendamento.

O responsável pelo Agendamento também é considerado um participante.

Outros usuários podem participar após receberem e aceitarem um convite.

---

## Convite

Representa uma solicitação enviada pelo responsável de um Agendamento a outro usuário para participar da atividade.

O convite pode ser aceito ou recusado pelo usuário convidado.

Um convite pendente não representa uma participação confirmada.

---

# Papéis do Sistema

O sistema utiliza **RBAC (Role-Based Access Control)**.

Atualmente existem quatro papéis de acesso.

```text
PUBLICADOR
    │
    ▼
SERVO
    │
    ▼
ANCIAO
    │
    ▼
ROOT
```

---

## PUBLICADOR

Representa o usuário comum do sistema.

Permissões:

- Consultar escalas e disponibilidades.
- Realizar agendamentos.
- Participar de agendamentos mediante convite aceito.
- Atualizar informações do próprio perfil.

Restrições:

- Não pode acessar funcionalidades administrativas.
- Não pode gerenciar usuários.
- Não pode gerenciar locais, pontos ou escalas.
- Não pode alterar configurações do sistema.

---

## SERVO

Representa um administrador operacional.

Possui todas as permissões do PUBLICADOR.

Além disso pode:

- Criar e editar usuários.
- Criar e gerenciar Locais.
- Criar e gerenciar Pontos.
- Criar, editar e remover Escalas.
- Cadastrar, editar e remover Disponibilidades.
- Executar tarefas administrativas relacionadas à operação do sistema.

Restrições:

- Não pode remover ou desativar usuários.
- Não pode gerenciar configurações globais da plataforma.

---

## ANCIAO

Representa o administrador funcional do sistema.

Possui todas as permissões do SERVO.

Além disso pode:

- Remover ou desativar usuários.

---

## ROOT

Representa o administrador técnico da plataforma.

Possui todas as permissões do ANCIÃO.

Além disso pode:

- Gerenciar configurações globais.
- Alterar parâmetros do sistema.
- Configurar integrações.
- Executar tarefas de manutenção.
- Acessar funcionalidades exclusivas de administração.

O papel ROOT deve ser utilizado apenas para administração técnica da plataforma.

---

# Entidades do Sistema

## Usuário

Representa qualquer pessoa cadastrada na aplicação.

É responsável pela autenticação e autorização dentro do sistema.

Relacionamentos:

- Possui exatamente um Role.
- Pode ser responsável por diversos Agendamentos.
- Pode participar de diversos Agendamentos.
- Pode receber diversos Convites.
- Pode criar e gerenciar entidades conforme suas permissões.

---

## Local

Representa o espaço geográfico onde ocorre o Testemunho Público.

Relacionamentos:

- Possui diversos Pontos.

---

## Ponto

Representa uma localização física específica dentro de um Local.

Relacionamentos:

- Pertence a um único Local.
- Possui exatamente uma Escala.

---

## Escala

Representa a estrutura de disponibilidades associada a um Ponto.

Relacionamentos:

- Pertence a um único Ponto.
- Possui diversas Disponibilidades.

---

## Disponibilidade

Representa uma data e um intervalo de horário dentro de uma Escala.

Relacionamentos:

- Pertence a uma única Escala.
- Pode possuir um Agendamento.

---

## Agendamento

Representa a reserva de uma Disponibilidade.

Relacionamentos:

- Está associado a uma única Disponibilidade.
- Possui um Usuário responsável.
- Possui de dois a três participantes no total, incluindo o responsável.
- Pode possuir Convites destinados a outros usuários.

---

## Participante

Representa a participação de um usuário em um Agendamento.

Relacionamentos:

- Está associado a um Agendamento.
- Representa um Usuário participante.

---

## Convite

Representa o convite enviado para que outro usuário participe de um Agendamento.

Relacionamentos:

- Está associado a um Agendamento.
- Possui um Usuário convidado.
- Possui um estado que representa sua situação, como pendente, aceito ou recusado.

---

# Relacionamentos do Domínio

```text
Local
  │
  │ possui
  ▼
Ponto
  │
  │ possui exatamente uma
  ▼
Escala
  │
  │ organiza
  ▼
Disponibilidade
  │
  │ recebe
  ▼
Agendamento
  │
  ├── Responsável (Usuário)
  │
  ├── Participantes (Usuários)
  │
  └── Convites
        │
        └── Usuários convidados
```

---

# Fluxos de Negócio

## Cadastro de Usuário

```text
SERVO ou ANCIÃO

↓

Cria Usuário

↓

Sistema valida os dados

↓

Usuário é persistido

↓

Conta segue o fluxo de ativação definido pelo sistema
```

---

## Criação de Local e Ponto

```text
SERVO ou ANCIÃO

↓

Cria Local

↓

Sistema valida as informações

↓

Local é criado

↓

Administrador cria um Ponto associado ao Local

↓

Sistema valida as informações

↓

Ponto é criado
```

---

## Criação de Escala

```text
SERVO ou ANCIÃO

↓

Seleciona um Ponto

↓

Cria a Escala associada ao Ponto

↓

Sistema valida as informações

↓

Escala é criada
```

---

## Cadastro de Disponibilidades

```text
SERVO ou ANCIÃO

↓

Seleciona uma Escala

↓

Define datas e intervalos de horários

↓

Sistema valida as informações

↓

Disponibilidades são cadastradas
```

---

## Criação de Agendamento

```text
Usuário autenticado

↓

Seleciona uma Disponibilidade

↓

Realiza o Agendamento como responsável

↓

Sistema valida a disponibilidade e as regras de participantes

↓

Agendamento é criado
```

---

## Convite de Participantes

```text
Responsável pelo Agendamento

↓

Seleciona outro usuário

↓

Envia um Convite

↓

Usuário convidado recebe o Convite

↓

Aceita ou recusa

↓

Sistema atualiza a situação do Convite

↓

Participação é confirmada em caso de aceite
```

---

## Consulta de Escalas

```text
Usuário autenticado

↓

Sistema identifica suas permissões

↓

Escalas e disponibilidades autorizadas são retornadas
```

---

# Regras de Negócio

As regras abaixo representam princípios fundamentais do sistema.

## Usuários

- Todo usuário deve estar autenticado para acessar recursos protegidos.
- Todo usuário possui exatamente um papel (Role).
- Usuários não podem acessar recursos sem autorização.
- Usuários só podem executar operações permitidas pelo seu papel.
- O gerenciamento de usuários deve respeitar as permissões de SERVO, ANCIÃO e ROOT.

---

## Locais e Pontos

- Todo Ponto deve estar vinculado a um Local.
- Um Local pode possuir diversos Pontos.
- Cada Ponto deve possuir exatamente uma Escala.
- Um Ponto não pode possuir múltiplas Escalas.

---

## Escalas e Disponibilidades

- Toda Escala pertence a um único Ponto.
- Uma Escala não existe de forma independente de um Ponto.
- Cada Escala organiza as disponibilidades do seu Ponto.
- Uma Disponibilidade pertence a uma única Escala.
- As Disponibilidades representam datas e intervalos de horários individuais.
- SERVO, ANCIÃO e ROOT podem cadastrar e gerenciar as disponibilidades.

---

## Agendamentos e Participantes

- Todo Agendamento deve estar associado a uma Disponibilidade.
- Uma Disponibilidade que já possui um Agendamento não pode receber outra reserva.
- O responsável pelo Agendamento também é considerado participante.
- Todo Agendamento deve possuir no mínimo dois e no máximo três participantes no total.
- Outros usuários não podem solicitar participação diretamente em Agendamentos existentes.
- A participação de outros usuários ocorre por meio de Convites enviados pelo responsável.
- Um Convite pendente não representa uma participação confirmada.
- A participação somente é confirmada após o aceite do Convite.

---

## Permissões

- Recursos administrativos devem respeitar os papéis definidos pelo RBAC.
- Funcionalidades técnicas e configurações globais são exclusivas do papel ROOT.
- Recursos protegidos devem validar autenticação e autorização.
- O sistema deve impedir acessos indevidos por meio de validações de propriedade e autorização contextual, incluindo proteção contra IDOR.

---

# Casos de Uso

O sistema deverá suportar, entre outros, os seguintes casos de uso.

### Autenticação

- Login.
- Logout.
- Renovação de sessão.
- Recuperação de senha.

---

### Usuários

- Cadastro.
- Atualização de perfil.
- Consulta de informações.
- Gerenciamento administrativo de usuários.

---

### Locais e Pontos

- Cadastro de Locais.
- Alteração de Locais.
- Remoção de Locais.
- Consulta de Locais.
- Cadastro de Pontos.
- Alteração de Pontos.
- Remoção de Pontos.
- Consulta de Pontos.

---

### Escalas e Disponibilidades

- Criação de Escalas.
- Atualização de Escalas.
- Remoção de Escalas.
- Consulta de Escalas.
- Cadastro de Disponibilidades.
- Atualização de Disponibilidades.
- Remoção de Disponibilidades.

---

### Agendamentos

- Criação de Agendamentos.
- Consulta de Agendamentos.
- Gerenciamento de participantes.
- Envio de Convites.
- Aceite de Convites.
- Recusa de Convites.

---

### Administração

- Configuração de parâmetros globais.
- Gerenciamento de configurações da plataforma.
- Administração técnica do sistema (ROOT).

---

# Evolução do Domínio

Este documento descreve o domínio atual do sistema.

Novas funcionalidades poderão introduzir novas entidades, papéis e regras de negócio.

Funcionalidades como recorrências automáticas e outros modelos de organização de escalas poderão ser avaliadas em versões futuras.

Sempre que houver alteração significativa no domínio, este documento deverá ser atualizado antes da implementação ou em conjunto com ela.

---

# Documentos Relacionados

- ARCHITECTURE.md
- DATABASE.md
- SECURITY.md
- AGENTS.md
- README.md