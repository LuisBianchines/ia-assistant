# AI Chat Button (Lapidatto AI Assistant)

Widget de chat em JavaScript (frontend puro) que injeta um botão/flutuante e uma janela de conversa no seu site. O chatbot usa as APIs Nativas de IA do Chrome (Prompt API para Gemini Nano) para responder em tempo real, renderizando Markdown via `marked`.

> Importante: atualmente funciona apenas no Google Chrome/Canary com flags habilitadas (detalhes em Requisitos).

## Sumário
- [Visão Geral](#visão-geral)
- [Requisitos](#requisitos)
- [Instalação e Execução](#instalação-e-execução)
- [Como Usar/Embedar](#como-usarembedar)
- [Configuração do Bot](#configuração-do-bot)
- [Arquitetura e Estrutura](#arquitetura-e-estrutura)
- [Desenvolvimento](#desenvolvimento)
- [Limitações e Notas](#limitações-e-notas)
- [Roadmap (Ideias)](#roadmap-ideias)
- [Licença](#licença)

## Visão Geral
- Interface de chat leve, injetada em runtime com HTML/CSS próprios.
- Respostas via APIs nativas de IA do Chrome (`window.LanguageModel`), com streaming e botão de parar.
- Configuração por JSON e arquivos de texto (`botData/`).
- Conteúdo de respostas em Markdown (renderizado com `marked` via CDN).

## Requisitos
- Navegador: Google Chrome (ou Chrome Canary) recente.
- Flags do Chrome (digite `chrome://flags/` na barra de endereço e habilite):
  - Prompt API for Gemini Nano: `chrome://flags/#prompt-api-for-gemini-nano`
  - Reinicie o navegador após habilitar.
- Node.js 18+ (para rodar o servidor de desenvolvimento com BrowserSync).

## Instalação e Execução
1. Instalar dependências:
   ```bash
   npm install
   ```
2. Rodar em desenvolvimento (servidor estático com live-reload):
   ```bash
   npm start
   ```
3. Abra o navegador em `http://localhost:3000` (Chrome com as flags habilitadas).

## Como Usar/Embedar
- Uso local (neste projeto): o arquivo `index.html` já importa o widget:
  ```html
  <script type="module" src="./sdk/src/index.js"></script>
  ```
- Para embutir em outro site/projeto estático:
  - Copie as pastas `sdk/` e `botData/`, além de `llms.txt` e o ícone/`avatar.webp`.
  - No HTML do seu site, adicione:
    ```html
    <script type="module" src="/sdk/src/index.js"></script>
    ```
  - Garanta que os caminhos permaneçam válidos, pois o widget faz `fetch` relativo a `sdk/src/index.js` para:
    - `sdk/ew-chatbot.css`
    - `sdk/ew-chatbot.html`
    - `botData/systemPrompt.txt`
    - `botData/chatbot-config.json`
    - `llms.txt`

## Configuração do Bot
Edite `botData/chatbot-config.json` para personalizar aparência e texto:
- `primaryColor`, `buttonColor`, `backgroundColor`, `headerColor`: cores principais do tema.
- `userBubble`, `botBubble`, `userText`, `botText`: cores dos balões e textos.
- `iconUrl`, `botAvatar`: URLs do ícone/flutuante e avatar do bot (ex.: `./botData/avatar.webp`).
- `chatbotName`: nome exibido no cabeçalho.
- `welcomeBubble`: texto do balão de boas‑vindas (fora da janela).
- `firstBotMessage`: primeira mensagem do bot dentro do chat (texto simples, sem Markdown).
- `typingDelay`: controla a velocidade da animação de “digitando” (ms ou número compatível; o código converte para duração da animação).

Conteúdo e contexto do assistente:
- `botData/systemPrompt.txt`: instruções do sistema (regras de conduta do assistente).
- `llms.txt`: conteúdo contextual (links, serviços, portfólio, contatos). É concatenado ao `systemPrompt` no carregamento.

Dicas:
- Mantenha caminhos relativos válidos (o código carrega via `fetch`).
- Troque `avatar.webp` em `botData/` para customizar a identidade visual.

## Arquitetura e Estrutura
- `index.html`: página de exemplo que injeta o widget.
- `sdk/src/index.js`: boot do widget; carrega CSS/HTML, lê `botData/` e inicializa Controller/View/Service.
- `sdk/src/views/chatBotView.js`:
  - Renderiza UI, aplica tema via CSS custom properties, exibe mensagens e estado “digitando”.
  - Usa `marked` (CDN) para renderizar Markdown das respostas do bot.
- `sdk/src/controllers/chatBotController.js`:
  - Orquestra eventos (abrir, enviar, parar), valida requisitos e faz streaming das respostas.
- `sdk/src/services/promptService.js`:
  - Abstrai `window.LanguageModel`: cria sessão com `initialPrompts` e expõe `promptStreaming()`.
- `sdk/ew-chatbot.html` e `sdk/ew-chatbot.css`: layout e estilos do widget.
- `botData/`: configurações, prompts e ativos do bot (ex.: `avatar.webp`).

Árvore resumida:
```
├─ index.html
├─ llms.txt
├─ botData/
│  ├─ chatbot-config.json
│  ├─ systemPrompt.txt
│  └─ avatar.webp
└─ sdk/
   ├─ ew-chatbot.css
   ├─ ew-chatbot.html
   └─ src/
      ├─ index.js
      ├─ controllers/chatBotController.js
      ├─ services/promptService.js
      └─ views/chatBotView.js
```

## Desenvolvimento
- Servidor de desenvolvimento:
  - `npm start` roda `browser-sync` servindo o diretório atual com watch em `sdk/**` na porta 3000.
- Padrões de código:
  - Projeto em JS moderno (ES Modules). Sem bundler por padrão.
  - Evite dependências desnecessárias; priorize simplicidade e compatibilidade estática.
- Dicas de depuração:
  - Abra o console e verifique mensagens do Controller (streaming e erros).
  - Se a UI carregar mas a IA não responder, revise as flags do Chrome e veja se `window.LanguageModel` existe.

## Limitações e Notas
- Compatibilidade: depende das APIs nativas de IA (`window.LanguageModel`), hoje disponíveis apenas no Chrome/Canary com flags.
- Hospedagem: como há `fetch` de arquivos estáticos, sirva tudo no mesmo domínio/origem do seu site para evitar CORS.
- Segurança: conteúdo do bot é renderizado com `marked`. Avalie políticas de conteúdo e sanitização se ingerir entradas não‑confiáveis.
- Acessibilidade: botões possuem `aria-label`, mas recomenda‑se validar com ferramentas de A11y do seu projeto.

## Roadmap (Ideias)
- Empacotar como módulo NPM (build ESM/UMD) para facilitar embed.
- Fallback para provedores de IA via API quando `LanguageModel` não estiver disponível.
- Persistência de histórico no `localStorage` ou backend opcional.
- Temas predefinidos e suporte a i18n da UI.

## Licença
ISC — veja `package.json`.
