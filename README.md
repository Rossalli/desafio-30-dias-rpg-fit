# Desafio dos 30 Dias — RPG FIT

Site estático que transforma o PDF do **Desafio dos 30 Dias** em uma jornada acompanhável: marca onde você está, gera card de compartilhamento com sua foto, exporta os desafios para a agenda e responde os quizzes do Mestre.

Um arquivo HTML, sem build, sem backend, sem chave de API. Roda no GitHub Pages.

---

## Como publicar

1. Suba o conteúdo deste repositório para o GitHub.
2. **Settings → Pages → Source:** branch `main`, pasta `/ (root)`.
3. Aguarde a build. O site fica em `https://SEU-USUARIO.github.io/NOME-DO-REPO/`.

Também funciona abrindo o `index.html` direto no navegador, ou em Netlify, Vercel e Cloudflare Pages.

---

## Estrutura

```
.
├── index.html      # o site inteiro: markup, estilo, lógica e dados
├── img/            # artes dos desafios, extraídas do PDF
│   ├── dia-01.jpg
│   ├── dia-02.jpg
│   └── ...           (até dia-30.jpg)
└── README.md
```

Não há dependência instalada. As fontes (Cinzel, Alegreya, IBM Plex Mono) vêm do Google Fonts por CDN; sem internet o site cai para as fontes do sistema e continua funcionando.

---

## O que o site faz

### Progresso

Você define a data do **Dia 1** no painel de agenda. A partir dela o site calcula sozinho qual é a missão de hoje — o botão "Ir para hoje" salta direto para ela.

Cada missão concluída vira um selo carimbado na trilha, agrupado pelas cinco etapas do desafio original.

### XP

| Fonte | Valor |
|---|---|
| Missão concluída | 1 XP × 30 |
| Quiz respondido corretamente | 1 XP × 2 |
| **Total possível** | **32 XP** |

O XP é sempre recalculado a partir do estado (`feitos.length + acertos()`), nunca acumulado numa variável. Desmarcar uma missão devolve o ponto.

### Quizzes

Os dois quizzes do PDF (dias 4 e 22) são respondíveis dentro do card. Ao escolher uma alternativa, a correta fica verde, a escolhida errada fica vermelha, e aparece a explicação. Dá para responder de novo.

Na trilha, dias com quiz mostram um ponto dourado — vazio enquanto não acertou, preenchido depois.

### Compartilhamento

O site monta o card num `<canvas>`: sua foto ao fundo, gradiente escuro por cima, e dia, título, missão e hashtags gravados em texto. Dois formatos:

- **Post** — 1080 × 1350
- **Story** — 1080 × 1920

Três saídas:

| Botão | Comportamento |
|---|---|
| Compartilhar card | Usa a Web Share API (`navigator.share` com arquivo). No celular abre a bandeja do sistema, com Instagram e WhatsApp na lista. |
| Enviar no WhatsApp | Tenta a bandeja primeiro; sem suporte, baixa a imagem e abre `wa.me` com a legenda pronta. |
| Baixar imagem | JPEG direto para o rolo da câmera ou pasta de downloads. |

O Instagram não aceita publicação direta pelo navegador — nenhum site consegue isso. O caminho é a bandeja de compartilhamento no celular, ou baixar e postar manualmente.

### Agenda

Gera arquivos `.ics` padrão iCalendar, com alarme 10 minutos antes:

- **Os 30 dias** — um evento por dia, em sequência a partir da data de início
- **Só este desafio** — evento único do dia aberto
- **Google Agenda** — abre o formulário pré-preenchido numa aba nova

O `.ics` importa em Google Agenda, Apple Calendário, Outlook e qualquer cliente compatível.

---

## Onde o progresso fica salvo

No `localStorage` do navegador, por aparelho. Não há conta, login nem servidor.

Para continuar em outro celular ou computador, use **"Levar progresso para outro aparelho"** no rodapé. Ele gera um código assim:

```
RPGFIT30.eyJpIjoiMjAyNi0wOC0xOCIsImYiOlsxLDIsM10sInIiOnsiNCI6Mn19
```

O código **é** o dado, não um ponteiro para ele. Carrega três campos, em base64:

| Campo | Conteúdo |
|---|---|
| `i` | data do Dia 1 |
| `f` | array de missões concluídas |
| `r` | respostas dos quizzes (`dia: índice escolhido`) |

Dia atual e XP não vão junto — são derivados na hora. O texto dos desafios também não, já que vive no `index.html` dos dois lados. Por isso o código fica curto mesmo com a jornada completa.

> **Nota:** base64 é codificação, não criptografia. Quem tiver o código consegue ler o conteúdo. Para uma lista de dias isso não importa, mas não reaproveite o padrão para dados sensíveis.

---

## Personalizar

### Trocar as artes

Substitua os arquivos em `img/` mantendo os nomes (`dia-01.jpg` … `dia-30.jpg`). Se um arquivo faltar, o card renderiza sem imagem — o `onerror` remove o elemento e nada quebra.

### Editar um desafio

Todos os 30 vivem no array `DIAS`, no início do `<script>`:

```js
{
  t: "O Elixir de Vigor",              // título
  a: "Beba água",                      // a missão, em uma linha
  d: "Sua primeira missão é simples…", // descrição
  e: "texto do desafio extra"          // opcional
}
```

### Adicionar um quiz

Acrescente a chave `quiz` ao dia desejado:

```js
quiz: {
  intro: "texto de ambientação",
  p:     "a pergunta",
  o:     ["opção A", "opção B", "opção C", "opção D"],
  c:     1,                    // índice da resposta certa (0 = A)
  why:   "explicação mostrada depois de responder"
}
```

O XP máximo se ajusta sozinho — `XP_MAX` é derivado de quantos dias têm `quiz`.

### Mudar as etapas

O array `ETAPAS` define nome e faixa de dias de cada uma. Alterar ali reorganiza a trilha inteira.

---

## Se um dia precisar de backend

O código de jornada resolve continuidade entre aparelhos, mas não resolve ranking, guilda ou histórico compartilhado. Se isso entrar no escopo, o ponto de troca são só duas funções:

```js
async function salvar()   { /* grava o objeto `estado` */ }
async function carregar() { /* lê o objeto `estado` */ }
```

Ambas já são `async`. Trocar `localStorage` por Firestore, Supabase ou qualquer outro não toca em mais nada do arquivo.

O plano gratuito do Firestore (20 mil escritas/dia) cobre esse uso com folga — a decisão de não usá-lo foi de simplicidade, não de custo.

---

## Créditos

Desafio original: **Martus Jonathan** — [@rpgfitbrasil](https://instagram.com/rpgfitbrasil) · [martusrpgfit.com.br](https://www.martusrpgfit.com.br)

Textos, artes e a estrutura das cinco etapas vêm do PDF *Desafio dos 30 Dias — O Retorno dos Mentores Perdidos*. Este repositório é uma implementação web daquele material; o conteúdo permanece de seu autor.

`#desafio30dias`
