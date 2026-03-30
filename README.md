# Polaroom

Documentação de arquitetura, funcionamento e manutenção do projeto.

Este README descreve apenas o que está versionado no Git. Quando um arquivo ou pasta é citado aqui, ele existe no repositório ou é derivado diretamente de um arquivo versionado, como 'Polaroom/.env.example'.

O código principal da aplicação está em 'Polaroom/'.

O projeto nasceu da ideia de ter um lugar próprio para salvar links, playlists, anotações e outras coisas que eu uso no dia a dia. A proposta sempre foi misturar organização pessoal com prática real de desenvolvimento, usando o projeto como espaço para testar ideias, exercitar conhecimentos e ir refinando a experiência aos poucos.

O Polaroom continua em andamento e recebe adições constantes. Ele também já tem seu próprio link público:

- [polaroom.page](https://polaroom.page)

Visualmente, o site é temático e usa cores inspiradas nas eras da Taylor Swift, o que faz parte da identidade do projeto e da motivação pessoal por trás dele.

## O que é o projeto

O Polaroom é uma aplicação pessoal inspirada em "eras", usada para organizar links, playlists, notas, perfis, preferências e outros dados do dia a dia.

A aplicação combina:

- uma shell em React
- módulos legados em JavaScript puro
- autenticação com Supabase
- persistência local-first
- deploy estático via Cloudflare Workers

## Visão geral da arquitetura

Hoje o projeto funciona em camadas:

1. A shell em React monta o layout base, controla autenticação e tema.
2. Depois do login, a app carrega módulos legados que ainda cuidam de boa parte da experiência interna.
3. O estado da aplicação é salvo localmente e pode ser sincronizado com a camada remota.
4. Existe um backend opcional em 'Polaroom/server/' para cenários de apoio, compatibilidade e persistência autenticada.
5. O deploy do frontend é feito a partir da raiz do repositório com a configuração em 'wrangler.jsonc'.

## Stack principal

- React 19
- Vite
- JavaScript ES Modules
- Supabase
- localStorage como base local-first
- Express + PostgreSQL no backend opcional
- Cloudflare Workers para publicação do frontend

## Estrutura versionada do repositório

```text
Polaroom/
|-- README.md
|-- wrangler.jsonc
`-- Polaroom/
    |-- .editorconfig
    |-- .env.example
    |-- eslint.config.js
    |-- index.html
    |-- package.json
    |-- package-lock.json
    |-- postcss.config.js
    |-- vite.config.js
    |-- public/
    |-- scripts/
    |-- server/
    `-- src/
```

### Pastas principais em 'Polaroom/'

- `'public/'` -> assets públicos do frontend
- `'scripts/'` -> utilitários de manutenção, como checagem de encoding
- `'server/'` -> backend opcional em Express
- `'src/'` -> código do frontend e dos módulos legados

## Como o frontend sobe

### Entrada da aplicação

O ponto de entrada é `'Polaroom/src/main.jsx'`.

Esse arquivo monta a app no `'#root'` e carrega `'Polaroom/src/App.jsx'`.

`'Polaroom/src/App.jsx'` é a shell principal. Ela:

- controla o loading inicial
- usa o hook de autenticação
- exibe a tela de login quando não há sessão
- carrega os módulos legados depois da autenticação
- controla o tema
- habilita workspaces quando 'VITE_ENABLE_WORKSPACES=true'

### Autenticação

Arquivos centrais:

- `'Polaroom/src/hooks/useAuth.js'`
- `'Polaroom/src/lib/supabase.js'`

Fluxo resumido:

1. 'useAuth()' tenta recuperar a sessão atual.
2. O hook escuta mudanças de autenticação com 'onAuthStateChange'.
3. O hook expõe login, cadastro, logout, reset de senha e atualização de senha.
4. Se `'VITE_SUPABASE_URL'` e `'VITE_SUPABASE_ANON_KEY'` não estiverem definidos, `'Polaroom/src/lib/supabase.js'` mantém o client como 'null' para a UI operar em fallback seguro.

### Boot da aplicação interna

Depois do login, `'Polaroom/src/App.jsx'` faz import dinâmico de:

- `'Polaroom/src/legacy/storage.js'`
- `'Polaroom/src/legacy/app.js'`
- `'Polaroom/src/legacy/repositorios.js'`
- `'Polaroom/src/legacy/bootstrap.js'`

Ordem importante:

1. `'Storage.setUser(...)'`
2. `'Storage.bootstrapPersistence()'`
3. `'initLegacyApp()'`
4. `'initShellInteractions()'`
5. `'initRepositorios()'`

Isso garante que a aplicação tente hidratar o estado salvo antes de renderizar a experiência interna.

## Modelo mental da aplicação

A interface é organizada por "eras", que funcionam como áreas temáticas:

- Início
- Repositórios
- Música
- Notas
- Ferramentas
- Links
- Vídeos
- Resumos e Anotações
- Perfil
- Configurações

Grande parte dessas telas ainda é renderizada pela camada legada em `'Polaroom/src/legacy/app.js'`.

## Persistência e sincronização

Arquivo central:

- `'Polaroom/src/legacy/storage.js'`

Esse módulo cuida de:

- itens por categoria
- ordenação manual
- pins
- recentes
- preferências de UI
- perfil
- avatar e header
- notificações
- histórico local de versões
- sincronização remota quando disponível

### Estratégia de persistência

O projeto segue uma abordagem local-first.

Na prática:

1. o estado é salvo no 'localStorage'
2. depois ele pode ser sincronizado com a camada remota
3. a UI recebe status de sincronização
4. o app mantém versões históricas do estado

No backend, os snapshots e históricos trabalham com as tabelas 'app_state' e 'app_state_versions'.

## Shell e interações globais

Arquivo principal:

- `'Polaroom/src/legacy/bootstrap.js'`

Esse módulo concentra:

- navegação por hash e sidebar
- topbar
- busca
- modais
- edição de perfil
- avatar e header
- toasts
- integração visual com o status da aplicação
- bindings do seletor de workspace

## Dashboard GitHub e home

Arquivo:

- `'Polaroom/src/legacy/repositorios.js'`

Esse módulo cuida de:

- cards da área de Repositórios
- páginas recentes
- itens mais acessados
- painel de commits públicos
- heatmap de contribuições

O dashboard também lê preferências persistidas para alterar o perfil GitHub exibido na home.

## Backend opcional

Pasta:

- `'Polaroom/server/'`

Arquivo principal:

- `'Polaroom/server/src/index.js'`

Esse backend oferece:

- `'GET /health'`
- API legacy de 'items' protegida por flag
- endpoints autenticados para leitura e gravação de snapshots
- endpoints autenticados para histórico de versões

Observações importantes:

- o fluxo principal do projeto é frontend + Supabase
- o backend é opcional
- a API legacy de 'items' fica bloqueada por padrão se `'ENABLE_UNSCOPED_ITEMS_API'` não estiver habilitada

## Variáveis de ambiente

Arquivo-base:

- `'Polaroom/.env.example'`

Esse arquivo documenta as variáveis esperadas pelo frontend e pelo backend opcional.

### Frontend

Obrigatórias:

- `'VITE_SUPABASE_URL'`
- `'VITE_SUPABASE_ANON_KEY'`

Opcional:

- `'VITE_ENABLE_WORKSPACES=false'`

### Backend

Usadas pelo servidor opcional:

- `'DATABASE_URL'`
- `'SUPABASE_URL'`
- `'SUPABASE_ANON_KEY'`
- `'ALLOWED_ORIGINS'`
- `'ENABLE_UNSCOPED_ITEMS_API=false'`
- `'JWT_SECRET'`
- `'NODE_ENV'`

## Como rodar a partir do clone

### Frontend

```bash
cd Polaroom
npm ci
npm run dev
```

### Build

```bash
cd Polaroom
npm run build
```

### Lint

```bash
cd Polaroom
npm run lint
```

### Backend opcional

```bash
cd Polaroom/server
npm ci
npm run dev
```

## Scripts importantes

Arquivo:

- `'Polaroom/package.json'`

Scripts principais:

- `'npm run dev'`
- `'npm run build'`
- `'npm run preview'`
- `'npm run lint'`
- `'npm run check:encoding'`
- `'npm run fix:ptbr'`

Os utilitários ficam em:

- `'Polaroom/scripts/check-encoding.mjs'`
- `'Polaroom/scripts/fix-ptbr-mojibake.mjs'`

## Deploy

Configuração:

- `'wrangler.jsonc'`

O deploy usa a raiz do repositório, mas publica os assets gerados em `'Polaroom/dist'`.

Fluxo atual:

1. instalar dependências do frontend em 'Polaroom/'
2. buildar o frontend em 'Polaroom/'
3. publicar com 'wrangler deploy' usando a configuração da raiz

Exemplo:

```bash
npm ci --prefix "./Polaroom"
npm run build --prefix "./Polaroom"
npx wrangler deploy
```

## Arquivos que valem atenção ao editar

- `'Polaroom/src/App.jsx'` -> shell, login, layout base e boot inicial
- `'Polaroom/src/hooks/useAuth.js'` -> autenticação, sessão e recovery
- `'Polaroom/src/lib/supabase.js'` -> criação do client Supabase
- `'Polaroom/src/legacy/storage.js'` -> persistência, sync e histórico local
- `'Polaroom/src/legacy/bootstrap.js'` -> binds de UI, modais, perfil e shell
- `'Polaroom/src/legacy/app.js'` -> renderização das eras e telas internas
- `'Polaroom/src/legacy/repositorios.js'` -> dashboard GitHub e destaques da home
- `'Polaroom/src/styles/style.css'` -> estilos globais e responsividade
- `'Polaroom/server/src/index.js'` -> backend opcional
- `'wrangler.jsonc'` -> publicação do frontend

## Resumo prático

Se você for mexer no projeto no dia a dia, a ordem mental útil é:

1. `'App.jsx'` monta a shell e autentica
2. `'storage.js'` hidrata o estado e controla sync
3. `'app.js'` renderiza as eras
4. `'bootstrap.js'` conecta as interações
5. `'repositorios.js'` cuida da home e do GitHub
6. `'style.css'` amarra a camada visual

Se você estiver fazendo deploy:

1. builda o frontend em `'Polaroom/'`
2. garante que `'Polaroom/dist'` foi gerado
3. publica com 'wrangler deploy'

Se você estiver resolvendo bug de dados:

1. olha `'storage.js'`
2. confirma o estado local da aplicação
3. confirma a camada remota
4. só depois mexe na UI
