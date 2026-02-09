# 🚀 Bem-vindo ao LOCKDOWN!

## Guia de Início Rápido

Esta pasta contém tudo que você precisa para começar a trabalhar no projeto LOCKDOWN.

---

## 📖 Ordem de Leitura Recomendada

Siga esta sequência para configurar seu ambiente de desenvolvimento:

1. **[001-SETUP.md](./001-SETUP.md)** ⭐ **COMECE AQUI!**
   - Instalação completa do projeto
   - Pré-requisitos (Node.js, pnpm, Git)
   - Setup inicial do monorepo
   - Comandos de desenvolvimento

2. **[002-ENVIRONMENT.md](./002-ENVIRONMENT.md)**
   - Configuração de variáveis de ambiente
   - `.env` para backend, frontend e bot
   - Variáveis obrigatórias e opcionais
   - Segurança e boas práticas

3. **[003-FAQ.md](./003-FAQ.md)**
   - Perguntas frequentes
   - Dúvidas comuns de novos desenvolvedores
   - Respostas rápidas

4. **[004-TROUBLESHOOTING.md](./004-TROUBLESHOOTING.md)**
   - Problemas comuns e soluções
   - Debugging
   - Resolução de erros

---

## ⚡ Quick Start (Resumo Rápido)

Se você já tem experiência com monorepos e quer começar rapidamente:

```bash
# 1. Clonar e instalar
git clone <repo-url>
cd LOCKDOWN
pnpm install

# 2. Configurar .env (veja 002-ENVIRONMENT.md)
cp packages/backend/.env.example packages/backend/.env
cp packages/lockdown-bot/.env.example packages/lockdown-bot/.env
cp packages/frontend/.env.example packages/frontend/.env

# 3. Rodar tudo
pnpm dev

# 4. Acessar
# Backend: http://localhost:3000
# Frontend: http://localhost:5173
```

---

## 🎯 Para Quem é Esta Pasta?

- ✅ **Novos desenvolvedores** no projeto
- ✅ **Onboarding** de equipe
- ✅ **Setup de ambiente** local
- ✅ **Troubleshooting** de instalação

---

## 📚 Próximos Passos

Após configurar seu ambiente, continue para:

- **[../01-architecture/](../01-architecture/)** - Entender a arquitetura do sistema
- **[../03-core/](../03-core/)** - Conhecer os pilares fundamentais
- **[../04-backend/](../04-backend/)** - Desenvolvimento backend
- **[../05-frontend/](../05-frontend/)** - Desenvolvimento frontend
- **[../06-bots/](../06-bots/)** - Desenvolvimento de bots Discord

---

## 💡 Dica

**Não pule o SETUP!** Mesmo que você seja experiente, leia o [001-SETUP.md](./001-SETUP.md) para entender as particularidades deste monorepo.

---

**Vamos começar!** 🚀
