# 📤 Guia de Push no Git

Este documento define o **processo padrão para envio de commits** ao repositório remoto.

---

## 🎯 Objetivo do push

O comando `git push` é responsável por:

- 🚀 Enviar commits locais para o remoto
- 🤝 Compartilhar progresso com o time
- 🔒 Garantir versionamento centralizado

---

## 🧱 Pré-requisitos

Antes de realizar um push:

- ✅ Commits bem escritos (ver `COMMIT.md`)
- 🧪 Código testado
- 🔄 Branch atualizada com pull

---

## 📦 Push padrão

```
git push origin <branch>
```

### Exemplo

```
git push origin feature/login
```

---

## 🆕 Primeiro push da branch

Ao criar uma nova branch:

```
git push -u origin <branch>
```

Isso define o upstream automaticamente.

---

## ⚠️ Push forçado

🚨 **Evite ao máximo**.

```
git push --force-with-lease
```

Use apenas quando:

- Você sabe exatamente o que está fazendo
- Está em branch própria

---

## 🧠 Boas práticas

- Nunca faça push direto na `main`
- Prefira branches de feature
- Push pequeno e frequente

---

## 🏁 Conclusão

Push é publicação. Trate com responsabilidade.

> Se vai mandar pro mundo, mande bem.
