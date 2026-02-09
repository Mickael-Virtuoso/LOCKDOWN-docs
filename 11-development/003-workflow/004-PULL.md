# 📥 Guia de Pull no Git

Este documento descreve o **processo padrão para realizar pulls** neste repositório, garantindo sincronização segura com o remoto e evitando conflitos desnecessários.

---

## 🎯 Objetivo do pull

O comando `git pull` é utilizado para:

- 🔄 Atualizar o repositório local com mudanças remotas
- 🤝 Manter o código sincronizado com o time
- 🧠 Evitar divergências entre branches

---

## 🧱 Fluxo recomendado

Antes de iniciar qualquer trabalho:

```
git status
```

Garanta que:

- Não existam alterações não commitadas
- Você esteja na branch correta

---

## 🔽 Pull padrão

```
git pull origin <branch>
```

### Exemplo

```
git pull origin main
```

---

## 🔀 Pull com rebase (recomendado)

Para manter um histórico mais limpo:

```
git pull --rebase origin <branch>
```

### Exemplo

```
git pull --rebase origin develop
```

---

## ⚠️ Conflitos

Caso ocorram conflitos:

1. Resolva manualmente os arquivos indicados
2. Marque como resolvidos:

```
git add .
```

3. Continue o rebase:

```
git rebase --continue
```

---

## 🧠 Boas práticas

- Sempre faça pull **antes de começar a codar**
- Prefira `--rebase` em branches de feature
- Nunca faça pull com código quebrado localmente

---

## 🏁 Conclusão

Pulls frequentes evitam conflitos grandes e mantêm o fluxo saudável.

> Código sincronizado é código tranquilo.
