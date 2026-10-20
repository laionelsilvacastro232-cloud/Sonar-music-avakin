# Plataforma musical (nome provisório: "Sonar")

Next.js (App Router) + PostgreSQL (Drizzle ORM). Interface preta e branca, player persistente, biblioteca, playlists colaborativas, perfis sociais, painel administrativo e PWA instalável.

## Primeiro acesso / painel administrativo
- A **primeira conta cadastrada** (ou a conta cujo e-mail seja igual a `ADMIN_EMAIL`) vira administradora. Cadastre-se em `/cadastro` e acesse `/admin`.
- Todas as ações administrativas são validadas no servidor (`actionAdmin`) e registradas em **Admin → Auditoria**.

## Login com Google e código de verificação
- **Cadastro com e-mail e senha:** a conta é criada, um código de 6 dígitos (válido por 15 min, máx. 5 tentativas) chega por e-mail e a sessão só abre depois de digitá-lo em `/verificar`. Quem tenta entrar sem ter confirmado recebe um código novo.
- **Google:** `/api/auth/google` (OAuth 2.0 com PKCE). O Google já confirma o e-mail, então não pede código. Se já existir conta com o mesmo e-mail, ela é vinculada ao Google.
- **Banco:** `npm run build` roda `drizzle-kit push`, que cria/atualiza as tabelas no `DATABASE_URL` antes de compilar. Para criar só o banco: `npm run db:push`.

## Identidade visual
**Admin → Configurações**: nome, slogan, logotipo, favicon, textos institucionais (Termos, Privacidade, Sobre), links do rodapé, aviso global e recursos habilitados (cadastros, recursos sociais).

## Variáveis de ambiente
| Variável | Para quê | Obrigatória |
|---|---|---|
| `DATABASE_URL` | PostgreSQL | sim |
| `ADMIN_EMAIL` | e-mail que recebe papel admin ao se cadastrar | não |
| `APP_URL` | URL pública (links de e-mail) | recomendada em produção |
| `GOOGLE_CLIENT_ID` + `GOOGLE_CLIENT_SECRET` | botão "Continuar com o Google" (login e cadastro) | não — sem isso o botão não aparece |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS` **ou** `RESEND_API_KEY` + `EMAIL_FROM` | envio do **código de verificação** e da recuperação de senha | não — sem isso o cadastro entra direto, sem código |
| `JAMENDO_CLIENT_ID` | importar músicas Creative Commons com reprodução completa | não |

## Fontes de áudio
1. **Upload próprio / de artistas autorizados** (Admin → Conteúdo, ou Estúdio do artista com revisão).
2. **URL licenciada / de uso livre** (seed: exemplos SoundHelix).
3. **iTunes Search API** (sem chave): metadados, capas e **prévias oficiais de 30 s** + link para a faixa completa.
4. **Jamendo** (com `JAMENDO_CLIENT_ID`): faixas CC com reprodução integral. Uso comercial exige licença da Jamendo.
5. Faixas sem áudio liberado ficam como "somente fonte original" (botão "Abrir na fonte original").

Spotify, TUNE e NePlay **não** são usados como fonte de áudio (não há reprodução integral permitida/extração).

## Uploads
Gravados em `storage/uploads` (disco local) e servidos por `/media/<arquivo>` com suporte a Range. Em hospedagem com disco efêmero, use volume persistente ou migre para S3/R2.

## Testes
`node scripts/e2e.mjs` (com o servidor em `BASE=http://127.0.0.1:3100`) executa 44 verificações: cadastro, login, permissões, curtidas, playlists, histórico, admin e fontes de áudio.

## Limites
Rate limit em memória (por instância). Para múltiplas instâncias use Redis. Cache de páginas é dinâmico; para alto volume adicione CDN/cache e réplica de leitura.
