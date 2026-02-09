# 🌱 Guia de Branches no Git

Este documento descreve o **padrão de branches** adotado neste repositório.

---

## 🎯 Objetivo das branches

Branches permitem:

- 🧩 Desenvolvimento paralelo
- 🔒 Isolamento de mudanças
- 🚀 Entregas organizadas

---

## 🧱 Branches principais

| Branch  | Finalidade                 |
| ------- | -------------------------- |
| main    | Código estável em produção |
| develop | Integração de features     |

---

## 🌿 Branches de trabalho

### Feature

```
feature/<nome>
```

Exemplo:

```
feature/auth-login
```

### Fix

```
fix/<nome>
```

Exemplo:

```
fix/bot-crash
```

---

## ➕ Criar branch

```
git checkout -b feature/nova-feature
```

---

## 🔄 Atualizar branch

```
git pull --rebase origin develop
```

---

## 🔀 Merge

Merge deve ser feito via **Pull Request**, nunca direto na `main`.

---

## 🧠 Boas práticas

- Branch curta vive feliz
- Uma responsabilidade por branch
- Nome claro e descritivo

---

## 🏁 Conclusão

Branches bem organizadas tornam o projeto escalável e colaborativo.

> Uma branch limpa vale mais que mil commits confusos.
