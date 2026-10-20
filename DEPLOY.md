# Sonar — Deploy na Vercel + Neon (link permanente)

Guia completo para hospedar o aplicativo com **link fixo**, conectado ao **PostgreSQL do Neon**.
Funciona também na Render/Railway (basta seguir os mesmos passos e escolher a plataforma).

---

## Visão geral

| Peça | Onde | Gratuito? |
|---|---|---|
| Código do app | Vercel | Sim |
| Banco de dados | Neon (Postgres) | Sim (plano inicial) |
| E-mail de recuperação | Resend | Sim (3.000/mês) |

O **link da Vercel não muda** a cada atualização: é sempre `https://seu-app.vercel.app`.
As **sessões ficam no banco**, então atualizar **não desloga** os usuários (já testado).

---

## Passo 1 — Banco de dados no Neon (5 min)

1. Crie a conta em **https://neon.tech** (pode entrar com GitHub/Google).
2. Crie um projeto. O Neon mostra a **connection string** — copie.
3. Guarde essa string. Ela começa com `postgresql://...` e tem `sslmode=require`.

> As tabelas são criadas automaticamente no `npm run build` (`drizzle-kit push`), então o `DATABASE_URL` precisa estar definido já no build.

---

## Passo 2 — Subir o código para o GitHub (5 min)

O ZIP que você baixou contém todo o código. Envie para um repositório GitHub:

1. Crie um repositório vazio em **https://github.com** (pode ser **privado**).
2. Dentro da pasta extraída do ZIP, rode no terminal:

```bash
git init
git add .
git commit -m "Sonar - primeira versão"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/SEU_REPO.git
git push -u origin main
```

> O arquivo `.env` (com segredos) **não é versionado** — o `.gitignore` já o exclui.
> Só o `.env.example` vai para o Git.

---

## Passo 3 — Conectar a Vercel ao Neon (10 min)

1. Entre em **https://vercel.com** com a mesma conta do GitHub.
2. **Add New → Project** e escolha o seu repositório do Sonar.
3. A Vercel detecta o Next.js automaticamente. **Não mude nada** na build.
4. Antes de criar, clique em **Environment Variables** e adicione:

| Nome | Valor |
|---|---|
| `DATABASE_URL` | a connection string do Neon |
| `APP_URL` | `https://SEU-APP.vercel.app` |
| `ADMIN_EMAIL` | seu Gmail (para virar admin no 1º cadastro) |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | login com Google (veja Passo 6A) |
| `SMTP_HOST` / `SMTP_PORT` / `SMTP_USER` / `SMTP_PASS` | código de verificação por e-mail (veja Passo 6) |
| `RESEND_API_KEY` | (opcional) chave do Resend, alternativa ao SMTP |
| `EMAIL_FROM` | (opcional) `Suporte <suporte@seudominio.com>` |
| `JAMENDO_CLIENT_ID` | (opcional) seu client_id do Jamendo |

5. Clique em **Deploy**. Em ~1 minuto o app está no ar em `https://seu-app.vercel.app`.

> Essas variáveis também podem ser adicionadas depois em
> **Project → Settings → Environment Variables**, e aí basta fazer um novo Deploy.

---

## Passo 4 — Criar o administrador

1. Abra `https://seu-app.vercel.app`.
2. Clique em **Criar conta** e preencha.
3. Se cadastrou com o `ADMIN_EMAIL` definido acima, essa conta **já é admin**.
4. Entre e acesse **Painel administrativo** pelo menu do avatar.

> Se já tiver outra conta e quiser promover: **Admin → Usuários → Promover a admin**.

---

## Passo 5 — Atualizar o app (sem deslogar ninguém)

Qualquer mudança no código:

```bash
git add .
git commit -m "descreva a atualização"
git push
```

A Vercel detecta o push e publica sozinha. **O link continua o mesmo** e
**ninguém é deslogado** (as sessões ficam no banco de dados).

---

## Passo 6A — Login com Google

1. Em **https://console.cloud.google.com** crie um projeto → **APIs e serviços → Tela de consentimento OAuth** (tipo Externo; preencha nome e e-mail).
2. **Credenciais → Criar credenciais → ID do cliente OAuth → Aplicativo da Web**.
3. Em **URIs de redirecionamento autorizados** coloque `https://SEU-APP.vercel.app/api/auth/google/callback` (e `http://localhost:3000/api/auth/google/callback` para testar local).
4. Copie o ID e o segredo para `GOOGLE_CLIENT_ID` e `GOOGLE_CLIENT_SECRET` e faça um novo Deploy.
5. Enquanto a tela de consentimento estiver em "Teste", só e-mails adicionados como "usuários de teste" conseguem entrar. Clique em **Publicar app** para liberar para todos.

## Passo 6 — E-mail: código de verificação e recuperação de senha

**Mais simples (Gmail):** ative a verificação em duas etapas na sua conta Google, crie uma **senha de app** em myaccount.google.com/apppasswords e use `SMTP_HOST=smtp.gmail.com`, `SMTP_PORT=465`, `SMTP_USER=seu@gmail.com`, `SMTP_PASS=<senha de app>`. Envia para qualquer pessoa (limite do Gmail: ~500/dia).

## Passo 6B (opcional) — Resend em vez de SMTP

Sem isso, quem esquecer a senha vê um aviso e você gera o link manualmente
em **Admin → Usuários → Link de senha**. Para automatizar:

1. Crie conta em **https://resend.com** → **API Keys → Create API Key** (copie, começa com `re_`).
2. Adicione `RESEND_API_KEY` e `EMAIL_FROM` nas variáveis da Vercel.
3. Faça um novo Deploy.

Agora "Esqueci minha senha" envia e-mail automático (link válido por 1 hora).

> Alternativa gratuita: **Brevo** (300 e-mails/dia).

---

## Passo 7 (opcional) — Domínio próprio

Na Vercel: **Project → Settings → Domains** → adicione `seumusica.com`.
Depois aponte o `APP_URL` para o domínio novo e atualize o DNS conforme a Vercel indicar.

---

## Checklist antes de publicar para o público

- [ ] Trocar a senha provisória (Configurações → Senha).
- [ ] Trocar logo, nome e textos (Admin → Configurações).
- [ ] Revisar Termos e Política de Privacidade (Admin → Configurações).
- [ ] Testar cadastro, login e "Esqueci minha senha".
- [ ] Publicar ao menos uma música de teste e conferir se toca.

---

## Custos esperados

| Serviço | Inicial | Se crescer (~1.000 usuários) |
|---|---|---|
| Vercel | Grátis | Pro ~US$20/mês |
| Neon | Grátis | ~US$19/mês |
| Resend | Grátis | pago conforme volume |

> Não testei com 1.000 usuários simultâneos, então não afirmo que o plano
> gratuito aguenta sem teste real. O inicial costuma bastar para começar.

---

## Solução de problemas

- **"Error: DATABASE_URL is required"** → a variável não foi salva na Vercel, ou faltou um novo Deploy depois de adicioná-la.
- **Erro de conexão com o banco** → confirme que a string do Neon tem `?sslmode=require` e que o Neon não está em pausa (plano gratuito pausa após inatividade; acorde no painel).
- **E-mail não chega** → confirme `RESEND_API_KEY` e `EMAIL_FROM`; veja os logs em Vercel → Deploy → Functions.
