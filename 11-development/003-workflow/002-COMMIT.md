# 🧾 Guia de Commits no Git

Este documento define o **padrão oficial de commits** utilizado neste repositório.

Seu objetivo é garantir **clareza, consistência e rastreabilidade** no histórico do Git, facilitando manutenção, revisão de código e colaboração em equipe.

---

## 🎯 Por que seguir um padrão de commits?

Commits bem escritos funcionam como documentação viva do projeto. Eles ajudam a:

- 📖 Entender rapidamente o que mudou e por quê
- 🧠 Evitar retrabalho e confusão no futuro
- 🤝 Facilitar code reviews e trabalho em equipe
- 🔍 Tornar mais simples o debug e o rollback

Um bom commit conta uma história clara. Um commit ruim vira ruído.

---

## 🧱 Padrão adotado

Este projeto utiliza o padrão **Conventional Commits**, amplamente adotado na indústria.

### Formato básico

```
<tipo>(<escopo>): <descrição curta>
```

### Exemplo

```
feat(auth): adicionar login com JWT
```

---

## 🏷️ Tipos de commit

Os tipos indicam **a intenção da mudança**.

| Tipo     | Descrição                                |
| -------- | ---------------------------------------- |
| feat     | Nova funcionalidade                      |
| fix      | Correção de bug                          |
| refactor | Refatoração sem alterar comportamento    |
| perf     | Melhoria de performance                  |
| docs     | Alterações em documentação               |
| test     | Criação ou ajuste de testes              |
| style    | Formatação, lint ou espaços (sem lógica) |
| chore    | Tarefas internas (build, configs, deps)  |

---

## 🧩 Escopo

O escopo identifica **qual parte do sistema foi afetada**.

### Exemplos

```
fix(bot): corrigir loop infinito no handler
refactor(client): reorganizar inicialização
chore(deps): atualizar dependências
```

Recomenda-se usar nomes coerentes com a estrutura do projeto.

---

## ✍️ Descrição do commit

Boas descrições seguem estas regras:

- Use **verbo no infinitivo** (adicionar, corrigir, remover)
- Seja claro e objetivo
- Não use ponto final
- Limite recomendado: **até 72 caracteres**

### Bons exemplos

```
feat(commands): adicionar comando de ajuda
fix(api): validar token expirado
```

### Exemplos ruins

```
update stuff
ajustes diversos
mudanças
```

---

## 📜 Commit com corpo (mensagem longa)

Para mudanças mais complexas, utilize um corpo explicativo.

### Estrutura

```
<tipo>(<escopo>): <descrição curta>

Contexto da mudança
Motivação
Impactos relevantes
```

### Exemplo

```
feat(bot): implementar sistema de comandos dinâmicos

Permite carregar comandos via filesystem
Reduz acoplamento entre handlers
Facilita manutenção futura
```

---

## 🧠 Boas práticas

- ✅ Commits pequenos e frequentes
- ❌ Evite commits grandes com múltiplas responsabilidades
- 🔍 Revise com `git diff` antes de commitar
- 🧪 Garanta que o código funcione antes do commit
- 🧹 Não misture refatoração com feature no mesmo commit

---

## ⚙️ Comandos úteis

Ver estado do repositório:

```
git status
```

Ver diferenças antes do commit:

```
git diff
```

Commit simples:

```
git commit -m "feat(core): adicionar sistema de logs"
```

Commit com mensagem longa (editor):

```
git commit
```

---

## 🏁 Considerações finais

Este padrão deve ser seguido em **todos os commits do projeto**.

Commits bem escritos tornam o projeto mais profissional, previsível e fácil de manter.

> Código muda. Histórico permanece.
