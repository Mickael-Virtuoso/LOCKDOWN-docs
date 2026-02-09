# 📄 POLÍTICA DE PRIVACIDADE

**Versão:** 1.0  
**Última Atualização:** 31 de janeiro de 2026  
**Efetivo em:** 17 de julho de 2025

---

## 1. INTRODUÇÃO

Esta Política de Privacidade ("Política") descreve como o LOCKDOWN Bot ("Bot", "Nós", "Nossa") coleta, utiliza, armazena e protege seus dados pessoais quando você usa o LOCKDOWN ou qualquer serviço relacionado (coletivamente, os "Serviços").

O LOCKDOWN é um bot de moderação e segurança para Discord, desenvolvido pela Mickael Virtuoso, com foco em proteção de servidores contra abuso, sincronização de bans entre servidores, auditoria completa de ações e conformidade com LGPD (Lei Geral de Proteção de Dados) e GDPR (General Data Protection Regulation).

---

## 2. DADOS QUE COLETAMOS

### 2.1 Dados Coletados Automaticamente

Quando você interage com o LOCKDOWN, coletamos automaticamente:

**Dados de Identificação:**

- User ID (Discord)
- Username e Tag
- Guild ID (servidor)
- Guild Name
- Channel ID
- Channel Name
- Role ID
- Role Name
- Email (opcional, durante verificação)
- Timestamp de todas as ações

**Dados de Ação:**

- Tipo de ação (ban, kick, warn, mute, timeout)
- Motivo da ação
- Moderador que executou (ID e Name)
- Data e hora exata da ação

**Dados de Segurança:**

- Hash privado de evidências
- Foto frente e verso do documento (durante verificação de idade)
- Foto do proprietário da conta (durante verificação de idade)
- Hash do documento de identidade
- IP durante autenticação (somente para detectar VPNs/proxies/redirecionamentos)
- Tokens encriptados para Truth Confiability (não tokens pessoais do Discord)

**Dados de Verificação:**

- Data de criação da conta Discord
- Resposta de verificação de idade (auto-reportada)
- Documentos pessoais (fotos)
- Plataformas de contato configuradas (WhatsApp, Telegram, Email)

**Métricas e Analytics:**

- Frequência de uso de comandos
- Tipos de comandos utilizados
- Dados de uso geral (anônimos)
- Logs de autenticação (com hash privado)
- Logs de todas as ações do bot

### 2.2 Dados NÃO Coletados

O LOCKDOWN **NÃO coleta:**

- ❌ Endereço IP pessoal (exceto para detectar VPN/proxy na autenticação)
- ❌ Tokens pessoais do Discord ou outras plataformas
- ❌ Conteúdo de mensagens diretas
- ❌ Senhas de outras plataformas
- ❌ Dados de localização GPS
- ❌ Dados financeiros ou de pagamento (tratados separadamente)

---

## 3. COMO USAMOS SEUS DADOS

Usamos os dados coletados para:

**Funcionalidade Essencial:**

- Executar moderação (bans, kicks, warns)
- Manter registro de ações
- Sincronizar dados entre servidores (com consentimento)
- Verificação de idade e identidade
- Autenticação e autorização
- Manter snapshots de configuração de servidor

**Segurança:**

- Detectar e prevenir fraude
- Detectar abuso de plataforma
- Investigar atividades suspeitas
- Proteger integridade do bot e dos servidores
- Detectar VPNs/proxies para segurança aprimorada

**Conformidade Legal:**

- Responder a requisições judiciais
- Investigar violações de Termos de Serviço
- Atender obrigações legais

**Melhorias de Serviço:**

- Analytics interno (anônimo)
- Identificar e corrigir bugs
- Melhorar performance
- Desenvolver novas features

---

## 4. RETENÇÃO DE DADOS

### 4.1 Dados Gerais

**Padrão: 90 dias**

- Todos os dados de ações, logs e registros são automaticamente deletados após 90 dias
- Isso inclui bans, kicks, warns, motivos, timestamps
- Dados não deletáveis (veja seção 4.2) seguem política específica

### 4.2 Dados NÃO Deletáveis (Funcionamento Essencial)

Os seguintes dados **NUNCA são deletados automaticamente**, pois são essenciais para funcionamento do bot:

- User ID
- Username
- Action Type (tipo de ação)
- Timestamp da ação
- Moderator ID
- Motivo da ação
- Guild ID
- Hash da evidência

Esses dados podem ser deletados apenas mediante solicitação formal do usuário (veja seção 7).

### 4.3 Snapshots de Servidor

- **Retenção configurável:** entre 30 e 90 dias (definido pelo owner do servidor)
- Inclui: canais, roles, permissões, histórico de mensagens (com acesso restrito)
- Histórico de mensagens: somente visível com autorização do owner, modo read-only
- Auto-deletados conforme configuração

### 4.4 Documentos de Identidade

- **Normal:** Deletados após 48 horas da verificação bem-sucedida
- **Em caso de fraude:** Mantidos por até 90 dias para investigação
- Hash do documento: mantido permanentemente como prova de verificação
- Fotos originais: deletadas conforme prazos acima

### 4.5 Dados de Whitelist/Truth Confiability

- Mantidos enquanto o admin permanece no servidor
- Ao ser removido do servidor: **deletados automaticamente em até 24 horas**
- Tokens/senhas: **sempre encriptados e nunca compartilhados**
- Owner pode forçar reset total ou parcial a qualquer momento

### 4.6 Email

- **Email coletado:** Mantido enquanto o usuário usa o bot
- **Após saída:** Deletado após 90 dias
- **Se solicitado:** Deletado imediatamente (com confirmação)

### 4.7 Dados Menores (16-17 anos)

- **Minimizado:** Apenas IDs, names, ações, motivos, timestamps
- **Retenção:** 30 dias (vs 90 dias para maiores de idade)
- **Snapshots:** Não participam de cross-server
- **Email:** Não coletado exceto em casos críticos

---

## 5. COMPARTILHAMENTO DE DADOS

### 5.1 Cross-Server Sync (Sincronização Entre Servidores)

**Dados compartilhados entre servidores:**

- User ID
- Username
- Evidência (transcript ou arquivo anexado)
- Histórico de banimentos anteriores
- Guild Name onde foi banido (SEM Guild ID)
- Rank de motivos mais comuns

**Dados NÃO compartilhados:**

- Guild ID (nunca revelado)
- Moderator ID ou Name
- Email (privado)
- IP ou VPN data
- Documentos pessoais

**Condições:**

- **Ambos os servidores** devem autorizar explicitamente
- Server A: ✅ "Permitir compartilhamento de dados"
- Server B: ✅ "Receber dados compartilhados"
- Sem autorização de ambos: **zero sincronização**
- Sincronização pode ser manual (comando) ou automática (se configurada)

### 5.2 Equipe de Segurança do LOCKDOWN

- **Acesso:** Apenas Data Security Admins
- **Quantidade:** Mínimo viável (atualmente: só o desenvolvedor)
- **Permissões:** Ler e deletar dados (ambos auditados)
- **Quando:** Apenas com autorização explícita e motivo documentado
- **Auditoria:** Todos os acessos registrados com log permanente
- **Violação:** Se alguém deletar sem autorização, backups pessoais recuperam dados

### 5.3 Compartilhamento com Autoridades

**Requisições Judiciais:**

- Compartilharemos dados **APENAS com ordem judicial válida**
- **Antes** de compartilhar: notificaremos você (máximo 30 dias antes)
- **Exceção:** Se a ordem judicial proibir expressamente notificação (raro)
- **Log:** Registro permanente de toda requisição judicial
- **Seu direito:** Você pode contestar judicialmente

### 5.4 Terceiros

O LOCKDOWN **NÃO compartilha dados com terceiros**, exceto:

- ✅ Discord API (integração necessária)
- ✅ Autoridades (com ordem judicial)
- ✅ Equipe de segurança do LOCKDOWN (com autorização)
- ✅ Bots WhatsApp/Telegram (para notificações, se você autorizar)

---

## 6. SEGURANÇA DOS DADOS

### 6.1 Criptografia

- **Em trânsito:** HTTPS obrigatório + JWT/OAuth seguros
- **Em repouso:** Dados sensíveis encriptados no PostgreSQL
- **Senhas/Tokens:** Hash privado (não reversível)
- **Documentos:** Armazenados criptografados

### 6.2 Acesso Restrito

- Senhas/tokens: Acessados apenas por Data Security Admins
- Dashboard: Acesso via OAuth (Discord) + JWT (restrições granulares)
- Truth Confiability: Requer whitelist + senha para ações críticas
- Snapshots: Read-only, acesso controlado

### 6.3 Backups

- **Frequência:** Diária
- **Acesso:** Apenas desenvolvedor principal
- **Motivo:** Recuperação em caso de incident ou violação
- **Sucessão:** Será definida em documento separado (testamento digital)

### 6.4 Notificação de Breach

Se seus dados forem expostos:

- **Notificação:** Até 12 horas após identificação
- **Métodos:** Email → WhatsApp → Telegram (automático)
- **Conteúdo da notificação:**
  - Data e hora exata do vazamento
  - Dados expostos (específico)
  - Dados NÃO expostos (não foi tudo)
  - Recomendações (o que você deve fazer)
  - Opção de agendamento para diagnóstico
  - Data prévia para correção (ou confirmação se já corrigido)

- **Sua ação:** Você pode:
  - Resetar password via `/my-data`
  - Solicitar diagnóstico
  - Revogar autorização de qualquer plataforma

---

## 7. SEUS DIREITOS (LGPD/GDPR)

Você tem direito a:

### 7.1 Direito de Acesso

- **Comando:** `/my-data > Data Collected > Request my Data`
- **Prazo:** Recebe dados dentro de 30 dias via email
- **Conteúdo:** Todos os seus dados coletados desde uso inicial
- **Formato:** Arquivo estruturado (JSON ou CSV)

### 7.2 Direito à Portabilidade

- Solicite seus dados em formato estruturado
- Disponibilizamos para migrar para outro serviço
- Sem bloqueio ou punição por exercer esse direito

### 7.3 Direito à Exclusão ("Direito ao Esquecimento")

- **Comando:** `/my-data > Delete my Data`
- **O que é deletado:**
  - Email
  - Snapshots de histórico
  - Analytics pessoais
  - Logs de acesso dashboard

- **O que NÃO é deletado** (necessário para funcionamento):
  - User ID, Username
  - Action Type, Timestamp
  - Motivo da ação
  - Guild ID
  - Moderator ID
  - Hash de evidência

- **Prazo:** Exclusão em até 30 dias

### 7.4 Direito à Retificação

- Solicite correção de dados incorretos
- Contato: privacy@lockdown.bot
- Resposta em até 10 dias úteis

### 7.5 Direito de Revogação de Consentimento

- Qualquer momento, revogar consentimento:
  - Compartilhamento cross-server
  - Notificações via WhatsApp/Telegram
  - Coleta de email
  - Acesso a snapshots

### 7.6 Direito de Contestação

- Se banido injustamente, pode apelar
- Processo transparente dentro do servidor
- Equipe LOCKDOWN pode revisar caso
- Reversão possível se provado erro

---

## 8. VERIFICAÇÃO DE IDADE

### 8.1 Requisitos

- **Idade mínima:** 18 anos (recomendado)
- **Menores permitidos:** 16 anos (com restrições e consentimento)
- **Verificação dupla:**
  1. Data de criação da conta Discord (age >= 18)
  2. Pergunta ao usuário: "Qual sua idade?"
  3. Opcional: foto do documento pessoal

### 8.2 Documentos

- **Aceitos:** RG, CPF, Passaporte, CNH
- **Foto:** Frente e verso + selfie do proprietário
- **Armazenamento:** 48 horas (normal) / 90 dias (se fraude)
- **Hash:** Mantido permanentemente como prova

### 8.3 Menores (16-17 anos)

**Se autorizado (com restrições):**

- ✅ Usar o bot (funcionalidade básica)
- ✅ Receber avisos de atualizações/features
- ✅ Acessar `/my-data`
- ❌ Participar de cross-server sync
- ❌ Visualizar dados de outros usuários
- ❌ Configuração de Truth Confiability

**Dados coletados:**

- IDs (user, channel, guild)
- Names (user, channel, guild)
- Ações executadas
- Motivos
- Timestamps
- **Apenas esses dados**

### 8.4 Consequência de Mentira sobre Idade

Se você mentir sobre sua idade:

- **Banimento global** a todos produtos LOCKDOWN
- Inclui: LOCKDOWN Bot, bots WhatsApp, bots Telegram, outros projetos
- **Permanente** (pode apelar após 1 ano)

### 8.5 Emancipação

- Menores emancipados podem usar como maiores de idade
- Obrigatório apresentar documento de emancipação
- Acesso a todas as features

---

## 9. EMAIL (OPCIONAL)

### 9.1 Se você NÃO informar email:

- ❌ Sem notificação de breach via email
- ❌ Sem direito a defesa se DM do Discord estiver bloqueada
- ✅ Ainda pode usar o bot normalmente
- ✅ Pode solicitar dados depois (precisará informar email quando solicitar)

### 9.2 Se você informar email:

- ✅ Notificações de breach via email
- ✅ Direito a defesa em caso de ban injusto
- ✅ Password reset via email
- ✅ Relatório de dados via email

### 9.3 Alternativas a Email

Se não tem email ou prefere outros métodos:

- WhatsApp: Configure durante verificação
- Telegram: Configure durante verificação
- Ambos requerem autorização explícita

---

## 10. TRANSCRIPTS E EVIDÊNCIAS

### 10.1 O que é uma Evidência

- Registro completo de mensagens relevantes
- Filtrado: somente mensagens do usuário em questão
- Inclui: datas, horários, autor, conteúdo
- Leitura: qualquer pessoa pode ver (transparência)
- Edição: **ninguém pode editar** (integridade garantida)

### 10.2 Mensagens Deletadas/Editadas

- **Sistema de log:** Registra todas as deletações e edições
- **Visualização:** Tags "Deleted" ou "Modified" próximo à mensagem original (se possível)
- **Se não possível:** Arquivo separado em anexo com logs completos
- **Conteúdo:** Integralmente preservado (prova irrefutável)

### 10.3 Terceiros em Transcripts

- Se a conversa envolvia outros usuários, seus dados aparecem apenas se relevantes
- Nomes e IDs de terceiros podem aparecer (contexto necessário)
- Terceiros **não são notificados** da evidência
- Evidência acessível apenas a: owner do servidor, LOCKDOWN admins, autoridades

---

## 11. COOKIES E TECNOLOGIAS SIMILARES

- Dashboard LOCKDOWN: utiliza cookies de sessão (obrigatório)
- Rastreamento: nenhum (sem Google Analytics, Hotjar, etc)
- Localização: nunca coletada
- Cookies de terceiros: não utilizados

---

## 12. CONTATO - PRIVACIDADE

**Para dúvidas sobre privacidade:**

- Email: privacy@lockdown.bot
- Resposta: Dentro de 10 dias úteis
- Assunto: Mention "LGPD", "GDPR" ou "Privacy"

**Para requisições LGPD (Acesso, Exclusão, Portabilidade):**

- Use comando `/my-data` (automático)
- Ou email: privacy@lockdown.bot
- Prazo legal: 30 dias (cumprido em 10-20 dias)

**Para requisições judiciais:**

- Enviar ordem judicial para: legal@lockdown.bot
- Atendimento: Dentro de 24 horas

---

## 13. ATUALIZAÇÕES A ESTA POLÍTICA

- Política pode ser atualizada a qualquer momento
- Você será notificado via Discord caso mudem seus direitos
- Continuidade do uso = consentimento com novas termos
- Acesso completo a versão anterior: solicite privacy@lockdown.bot

**Data da última atualização:** 31 de janeiro de 2026

---
