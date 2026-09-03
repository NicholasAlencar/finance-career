# FINANCE CAREER

Plataforma de recrutamento exclusiva da **Finance Brazil**, construída sobre o projeto visual existente e integrada a Google Sheets/Drive por uma camada segura de Google Apps Script.

## Arquitetura

```text
Portal público / Painel RH / Área do candidato
                ↓
        API server-side do projeto
                ↓
          Google Apps Script
          ↙              ↘
Google Sheets            Google Drive
                         ↳ Currículos
                         ↳ Anexos
                         ↳ Exportações
                         ↳ Documentos de candidatos
```

O navegador não recebe IDs privados do Drive/Sheets, segredo do gateway nem chave da Finance AI. Não há Supabase, Base44 ou Lovable no runtime.

## Módulos implementados

- autenticação de candidatos e colaboradores, sessão HTTP-only e senha PBKDF2-SHA256;
- papéis: Administrador, RH/Recrutador, Gestor da Vaga, Entrevistador, Diretoria e Somente Leitura;
- permissões customizáveis por módulo e escopo por vaga/entrevistador;
- vagas com rascunho, publicação, pausa, encerramento, duplicação, busca e pipeline configurável;
- candidatura autenticada e Banco de Talentos com currículo privado de até 10 MB;
- portal do candidato deliberadamente simples, com somente **Perfil** e **Currículo**, sem acompanhamento do processo seletivo;
- perfil reutilizável com foto, contatos, LinkedIn, GitHub, portfólio e histórico de alterações;
- currículo estruturado com experiências, formações, cursos, certificações, competências e idiomas;
- importação de currículo/LinkedIn por prévia revisável, sem sobrescrita silenciosa;
- perguntas personalizadas por vaga, sem perguntas criadas automaticamente;
- pipeline Kanban persistido, histórico, auditoria, notificações e automações;
- entrevistas em lista/calendário, reagendamento, cancelamento, confirmação e links Meet/Teams/Zoom;
- avaliações e scorecards confidenciais com sete critérios de 1 a 5;
- Finance AI server-side para currículo, compatibilidade, busca, entrevistas, comparação, mensagens e insights;
- comunicação interna, tarefas e preparação para e-mail/WhatsApp com confirmação humana;
- relatórios reais, filtros por período, comparação de período, PDF/Excel e arquivamento no Drive;
- equipe, configurações, modelos, permissões e logs de auditoria imutáveis.

## Google Sheets

A planilha deve se chamar `Finance Career — Banco de Dados` e conter estas abas:

`Usuários`, `Papéis e Permissões`, `Vagas`, `Candidatos`, `Candidaturas`, `Banco de Talentos`, `Experiências`, `Formações`, `Competências`, `Entrevistas`, `Avaliações`, `Scorecards`, `Pipeline`, `Automações`, `Mensagens`, `Finance AI`, `Relatórios`, `Logs de Auditoria`, `Configurações`.

O arquivo `google-apps-script/Code.gs` contém `setupFinanceCareer()`, que cria/repara os cabeçalhos esperados **anexando colunas faltantes sem reordenar colunas existentes** e sem colocar IDs no repositório.


## Vagas iniciais

O repositório traz cinco vagas iniciais editáveis em `lib/domain/initial-jobs.mjs`:

- Desenvolvedor Júnior
- Desenvolvedor Pleno
- Desenvolvedor Sênior
- PMO
- Consultor Financeiro

As vagas de desenvolvimento tratam Inteligência Artificial como requisito obrigatório em níveis compatíveis com a senioridade. Nenhuma vaga inicial possui seção de benefícios. As perguntas de candidatura iniciam vazias.

Depois que o Apps Script estiver publicado, `npm run seed:jobs` cria somente as vagas que ainda não existem e **não sobrescreve vagas já editadas**.

## Configuração do Google Apps Script

Crie um projeto Apps Script, cole `google-apps-script/Code.gs` e configure estas **Script Properties** no ambiente do Google, nunca no Git:

```text
FINANCE_CAREER_SECRET
FINANCE_SPREADSHEET_ID
FINANCE_RESUMES_FOLDER_ID
FINANCE_ATTACHMENTS_FOLDER_ID
FINANCE_EXPORTS_FOLDER_ID
FINANCE_CANDIDATE_DOCS_FOLDER_ID
```

Execute `setupFinanceCareer()` uma vez e publique como Web App. A URL final `/exec` será usada somente pelo servidor do Finance Career.

## Variáveis do servidor

Copie `.env.example` para o gerenciador de secrets do ambiente de hospedagem e configure:

```text
GOOGLE_APPS_SCRIPT_URL
GOOGLE_APPS_SCRIPT_SECRET
FINANCE_SESSION_SECRET
FINANCE_AI_PROVIDER
FINANCE_AI_API_KEY
FINANCE_AI_MODEL
```

Nunca use prefixo `NEXT_PUBLIC_` para essas variáveis.

A Finance AI falha explicitamente com erro de configuração quando `FINANCE_AI_API_KEY` ou `FINANCE_AI_MODEL` não existem; não há resposta simulada.

## Primeiro Administrador

Depois que o Apps Script estiver publicado, configure temporariamente no terminal seguro:

```text
FINANCE_ADMIN_EMAIL
FINANCE_ADMIN_NAME
FINANCE_ADMIN_PASSWORD
```

E execute:

```bash
npm run setup:admin
```

A senha inicial deve ter pelo menos 10 caracteres e, no fluxo normal da plataforma, é validada com maiúscula, minúscula, número e caractere especial.

## Desenvolvimento e verificação

Requer Node.js `>=22.13.0`.

```bash
npm ci
npm run build
node --test tests/*.test.mjs
```

O GitHub Actions em `.github/workflows/ci.yml` repete instalação limpa, build e suíte completa em Node 24 a cada push/PR para `main`.

## Segurança

- secrets somente no servidor/Script Properties;
- sanitização e validação no gateway;
- limite de requisições por minuto;
- paginação e filtros;
- logs append-only;
- arquivos privados limitados às pastas gerenciadas pelo Finance Career;
- currículo e scorecards protegidos por escopo de acesso;
- revalidação do usuário interno em operações protegidas;
- mensagens externas e decisões de contratação/reprovação nunca são executadas automaticamente pela Finance AI.

## Publicação

O código do repositório fica pronto para CI/deploy. A publicação final requer apenas ações externas que não pertencem ao código: publicar o Web App do Google Apps Script, inserir os secrets reais no ambiente e publicar a build no hosting do projeto existente.
