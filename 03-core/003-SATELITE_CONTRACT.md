# SATELLITE_CONTRACT.md

## Visão Geral

Este documento define o **Contrato de Satélite** do ecossistema **LOCKDOWN**. Ele estabelece os requisitos técnicos, limites operacionais e responsabilidades que um bot deve cumprir para ser considerado um **Satélite Oficial**.

Um satélite não é um bot comum.
É um **agente especializado** operando sob regras claras.

---

## 1. Princípios Fundamentais

### 1.1 Especialização

- Cada satélite atua **em apenas um pilar principal**.
- Não existem satélites generalistas.

### 1.2 Subordinação Lógica

- O satélite **reporta eventos**.
- O LOCKDOWN **decide políticas globais**.

### 1.3 Independência Operacional

- O satélite deve continuar funcional mesmo sem o LOCKDOWN.
- A ausência do LOCKDOWN limita apenas efeitos globais.

---

## 2. Elegibilidade de um Satélite

Para ser elegível, um bot deve:

- Escolher **um único pilar primário**
- Implementar o `EVENT_PROTOCOL`
- Assinar eventos com credencial válida
- Respeitar escopos de permissão

Pilares disponíveis:

- AUTH
- MODERATION
- SECURITY
- TICKETS

---

## 3. Identidade do Satélite

Todo satélite possui:

- `satellite_id`
- `satellite_type`
- `pillar`
- `event_version`

Exemplo:

```json
{
  "satellite_id": "mod-sat-01",
  "satellite_type": "moderation",
  "pillar": "MODERATION",
  "event_version": "v1"
}
```

---

## 4. Escopo de Permissões

### 4.1 Princípio do Menor Privilégio

- Satélites solicitam apenas permissões essenciais.
- Permissões excessivas invalidam o contrato.

### 4.2 Permissões Globais

- Nenhum satélite pode aplicar punições cross-server.
- Apenas o LOCKDOWN possui esse escopo.

---

## 5. Eventos Permitidos

Cada satélite possui um conjunto **explicitamente permitido** de eventos.

Exemplo para MODERATION:

- MODERATION.WARN
- MODERATION.MUTE
- MODERATION.BAN_REQUEST

Eventos não listados são automaticamente rejeitados.

---

## 6. Persistência de Dados

- O armazenamento local é de responsabilidade do satélite.
- Dados locais não são considerados canônicos.
- Eventos enviados ao LOCKDOWN não devem depender de estado externo.

---

## 7. Respostas do LOCKDOWN

O LOCKDOWN pode responder com:

- ACK
- DENY
- POLICY_UPDATE
- SATELLITE_REVOKED

O satélite deve tratar todas de forma graciosa.

---

## 8. Revogação de Satélite

O LOCKDOWN pode revogar um satélite quando:

- Eventos inválidos são enviados repetidamente
- Há violação de escopo
- Assinatura inválida
- Comportamento malicioso

Revogação implica:

- Eventos ignorados
- Credenciais invalidadas

---

## 9. Versionamento e Compatibilidade

- Mudanças quebráveis geram nova versão do contrato
- Satélites antigos continuam operacionais
- Eventos incompatíveis são rejeitados

---

## 10. Filosofia

Este contrato existe para:

- Proteger o ecossistema
- Incentivar boas práticas
- Evitar concentração de poder

Um satélite não compete com o LOCKDOWN.
Ele **orbita**.
