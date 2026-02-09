# EVENT_PROTOCOL.md

## Visão Geral

Este documento define o **Protocolo de Eventos** que rege a comunicação entre o **LOCKDOWN** (core) e seus **Bots Satélites**. Ele estabelece princípios, responsabilidades, tipos de eventos e garantias de consistência, evitando sincronização direta de estado e promovendo escalabilidade, resiliência e evolução independente.

> Regra de ouro: **sincronizamos intenções, não estados**.

---

## 1. Filosofia do Protocolo

### 1.1 Fonte Única da Verdade

- O **LOCKDOWN** é a única fonte canônica de dados globais.
- Bots satélites **não compartilham banco**, nem sincronizam tabelas.
- Toda ação relevante é comunicada como **evento**.

### 1.2 Arquitetura Orientada a Eventos

- Satélites **emitem eventos**.
- LOCKDOWN **valida, decide e persiste**.
- Respostas podem gerar **eventos de retorno** (ack, policy update, deny).

### 1.3 Independência Operacional

- Satélites funcionam isoladamente.
- Podem usar SQLite, PostgreSQL simples ou outro armazenamento local.
- Falhas no LOCKDOWN não derrubam o satélite, apenas limitam impacto global.

---

## 2. Domínios Fundamentais do Ecossistema

O ecossistema é sustentado por **quatro pilares**. Todo evento pertence explicitamente a um deles.

### 2.1 AUTH (Portaria)

Responsável por identidade, verificação e permissões.

Exemplos:

- Verificação de usuário
- Aprovação manual ou automática
- Elevação ou revogação de permissões

### 2.2 MODERATION (Síndico / Zelador)

Responsável por ordem, regras e automoderação.

Exemplos:

- Warn, mute, kick, ban
- Automoderação preventiva
- Escalonamento de punições

### 2.3 SECURITY (Seguranças)

Responsável por antecipar riscos e reagir rapidamente.

Exemplos:

- Raid detection
- Spam massivo
- Comportamento anômalo

### 2.4 TICKETS (Comunicação Interna)

Responsável pela comunicação estruturada user → staff.

Exemplos:

- Abertura de ticket
- Escalonamento
- Auditoria de conversas

---

## 3. Tipos de Eventos

### 3.1 Evento Local

- Consumido apenas pelo próprio satélite.
- Não é enviado ao LOCKDOWN.

Exemplo:

- Cache de cooldown
- Métrica interna

### 3.2 Evento Global

- Enviado obrigatoriamente ao LOCKDOWN.
- Pode afetar múltiplos servidores.

Exemplo:

- Banimento
- Flag de risco

### 3.3 Evento Condicional

- Enviado ao LOCKDOWN, mas pode ser ignorado.
- LOCKDOWN decide se persiste ou não.

Exemplo:

- Warn leve
- Tentativa de ação bloqueada por política

---

## 4. Estrutura Base de um Evento

Todo evento deve conter:

```json
{
  "event_id": "uuid",
  "event_type": "MODERATION.BAN",
  "origin": "satellite-moderation",
  "timestamp": "ISO-8601",
  "guild_id": "string",
  "user_id": "string",
  "actor_id": "string",
  "payload": {},
  "signature": "HMAC ou JWT"
}
```

### Campos obrigatórios

- **event_id**: idempotência
- **event_type**: domínio + ação
- **origin**: identificador do satélite
- **signature**: validação de autenticidade

---

## 5. Fluxo de Comunicação

### 5.1 Satélite → LOCKDOWN

1. Ação ocorre localmente
2. Satélite valida regras mínimas
3. Evento é emitido
4. LOCKDOWN recebe e valida

### 5.2 LOCKDOWN → Satélite

- ACK (aceito)
- DENY (bloqueado por política)
- POLICY_UPDATE
- GLOBAL_FLAG

---

## 6. Cross-Server

### 6.1 Exclusividade

- **Apenas o LOCKDOWN** executa lógica cross-server.
- Satélites **não aplicam punições globais**.

### 6.2 Exemplo

- Usuário banido via satélite
- Evento enviado
- LOCKDOWN avalia histórico
- Decide impacto global
- Aplica (ou não) em outros servidores

---

## 7. Versionamento de Eventos

- Eventos são versionados: `event_type@v1`
- Mudanças incompatíveis criam nova versão
- Satélites antigos continuam funcionais

---

## 8. Segurança e Confiança

- Todos os eventos são assinados
- Satélites possuem escopo limitado
- LOCKDOWN pode revogar um satélite a qualquer momento

---

## 9. O que este protocolo NÃO é

- Não é sincronização de banco
- Não é replicação de estado
- Não é RPC direto

É **coordenação distribuída por intenção**.

---

## 10. Visão de Ecossistema

Este protocolo existe para:

- Evitar bots monolíticos
- Incentivar especialização
- Criar competição saudável
- Elevar o nível técnico do ecossistema Discord

O LOCKDOWN não é o fim.
É o **alicerce**.
