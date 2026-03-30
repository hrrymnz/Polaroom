# Polaroom

O Polaroom é um site pessoal criado para organizar coisas do dia a dia: links favoritos, playlists, anotações, resumos e muito mais. Ele também serve como um espaço de prática e experimentação de ideias de desenvolvimento.

O visual é inspirado nas eras da Taylor Swift — isso faz parte da identidade do projeto.

Acesse em: [polaroom.page](https://polaroom.page)

---

## O que ele faz

É como um caderno digital personalizado, dividido em seções temáticas chamadas de "eras":

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

---

## Como ele funciona

- Você faz login e a aplicação carrega tudo que foi salvo anteriormente.
- As informações ficam guardadas localmente no navegador e também podem ser sincronizadas com uma nuvem.
- O site é publicado na internet de forma estática, o que significa que carrega rápido e não depende de um servidor sempre ligado.

---

## Personalização e extras

O Polaroom também tem espaço para descontrair e deixar tudo com a sua cara:

- **Perfil de usuário** - você pode criar e personalizar o seu próprio perfil dentro do site *(funcionalidade ainda sendo aprimorada)*
- **Calendário do GitHub** - nas configurações, basta adicionar o link do seu perfil do GitHub para exibir o seu calendário de contribuições personalizado
- **Itens fixados** - você escolhe quais itens quer deixar fixados para acessar mais rápido
- **Tema noturno** - o site possui modo escuro para quem prefere navegar com menos luz
- E outras opções de personalização que continuam sendo adicionadas ao longo do tempo

> O painel de commits também está passando por melhorias.

---

## Para quem quiser rodar localmente

Você vai precisar ter o [Node.js](https://nodejs.org) instalado. Depois, basta rodar no terminal:

```bash
cd Polaroom
npm ci
npm run dev
```

Para publicar:

```bash
npm run build
npx wrangler deploy
```

---

## Estrutura resumida

| Pasta / Arquivo | Para que serve |
|---|---|
| `src/App.jsx` | Tela principal e login |
| `src/legacy/app.js` | Renderiza as seções do site |
| `src/legacy/storage.js` | Salva e sincroniza os dados |
| `src/styles/style.css` | Aparência visual |
| `server/` | Parte opcional de servidor |
| `wrangler.jsonc` | Configuração de publicação |

---

*O projeto está em constante evolução e recebe novidades com frequência.*
