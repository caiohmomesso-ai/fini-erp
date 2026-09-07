# Estoque Fini

Controle de estoque da loja Fini: cadastro, contagem, movimentos, relatórios de saída e inteligência de compra.

O app é **um arquivo só** (`index.html`). Não tem build, não tem npm, não tem servidor. Você abre no navegador e funciona.

---

## O que ele faz

| Aba | Para quê |
|---|---|
| **Estoque** | Saldo por SKU no depósito e no balcão, valor a custo, situação de cada produto. Clicar num produto abre a ficha dele. |
| **Movimentos** | Entrada (compra, bonificação), saída (venda, degustação, desperdício, quebra) e transferência do depósito para o balcão. Importa a venda do Degust e lê nota de compra em PDF. |
| **Relatórios** | O que mais e o que menos sai, e a saída dia a dia separada por motivo. |
| **Inteligência** | Consumo médio, quantos dias de estoque restam e quanto comprar considerando o prazo do fornecedor. |
| **Contagem** | Contagem do depósito e do balcão, na mão ou colando planilha, com apuração de diferença antes de virar saldo. |
| **Produtos** | Catálogo, foto por produto, importação em massa, conciliação de códigos do Degust. |

### Duas regras que definem o modelo

1. **Granel perde o sabor no balcão.** No depósito cada sabor tem saldo próprio. Ao ser transferido para a área de venda, todo granel passa a compor o **GULOSEIMAS FINI** — que é como a venda sai do Degust, num código único em KG. Por isso o giro de cada sabor é medido pela **transferência**, o último ponto onde o sabor ainda é conhecido.
2. **Saldo é calculado, não guardado.** Saldo = última contagem aplicada + movimentos posteriores. Uma contagem nova zera qualquer erro acumulado. É o que faz o inventário continuar confiável depois de meses.

---

## Implantação — GitHub → Supabase → Vercel

Leva uns 20 minutos na primeira vez. Faça na ordem.

### Parte 1 — Supabase (o banco de dados)

1. Entre em **supabase.com** e crie a conta (pode entrar com o GitHub).
2. **New project**. Dê o nome `fini-estoque`, escolha a região **South America (São Paulo)** e defina uma senha do banco (guarde, mas você não vai precisar dela no dia a dia). Aguarde uns 2 minutos até o projeto ficar pronto.
3. No menu lateral, abra **SQL Editor** → **New query**.
4. Abra o arquivo `schema.sql` deste projeto, copie **todo** o conteúdo, cole no editor e clique em **Run**. Deve aparecer *Success*.
5. Ainda no menu lateral, vá em **Project Settings** → **API**. Anote dois valores:
   - **Project URL** — algo como `https://abcdefgh.supabase.co`
   - **anon public** — uma chave longa começando com `eyJ...`

> **Sobre o acesso.** O `schema.sql` já vem com a opção segura ligada: só quem tem usuário no Supabase lê e grava. Se você quiser começar sem login — mais simples, porém qualquer pessoa com o link e a chave consegue gravar — o próprio arquivo explica como trocar, no bloco "OPÇÃO B". Para criar os usuários da equipe: **Authentication → Users → Add user**.

### Parte 2 — GitHub (onde o código mora)

1. Entre em **github.com** e crie a conta, se ainda não tiver.
2. Clique em **New repository**. Nome: `fini-estoque`. Deixe **Private**. Crie.
3. Na tela seguinte, clique em **uploading an existing file**.
4. Arraste os três arquivos deste pacote: `index.html`, `schema.sql`, `README.md`.
5. Antes de subir, abra o `index.html` no seu computador com o Bloco de Notas (ou VS Code) e preencha a configuração no topo do arquivo:

```js
window.FINI_CONFIG = {
  supabaseUrl: "https://abcdefgh.supabase.co",
  supabaseKey: "eyJhbGciOi...sua chave anon...",
  tabela: "fini_docs"
};
```

6. Salve, arraste o arquivo já preenchido e clique em **Commit changes**.

### Parte 3 — Vercel (colocar no ar)

1. Entre em **vercel.com** e faça login **com a conta do GitHub**.
2. **Add New** → **Project**.
3. Encontre `fini-estoque` na lista e clique em **Import**.
4. Não mexa em nada: Framework Preset fica em **Other**, sem build command, sem output directory.
5. **Deploy**. Em menos de um minuto sai um endereço tipo `fini-estoque.vercel.app`.
6. Abra o endereço no celular e adicione à tela de início — no iPhone, botão de compartilhar → *Adicionar à Tela de Início*; no Android, menu → *Adicionar à tela inicial*. Fica com cara de aplicativo.

### Parte 4 — primeira carga

1. Abra o app e vá em **Produtos** → **Importar lista**.
2. Cole o catálogo do Degust. Uma linha por produto, colunas separadas por tabulação (colar do Excel funciona):
   `código · nome · categoria · custo · tipo · un/caixa · peso un`
   Só as duas primeiras são obrigatórias. Em **tipo**, escreva `granel` ou `item`.
3. Vá em **Contagem** → **Contar estoque fechado**. Essa primeira contagem é a abertura do estoque: sem ela não existe saldo.
4. Aplique a contagem. A partir daí é só lançar movimento e importar venda.

---

## Carga inicial por código

O arquivo `carga-inicial-fini.js` grava os 214 produtos do portal Sults já calibrados — custo, tipo (granel ou item), unidades por caixa e peso da embalagem. Não carrega quantidade nenhuma: só o cadastro.

1. Abra o app no computador.
2. `F12` → aba **Console**.
3. Cole o arquivo inteiro e aperte Enter.
4. Acompanhe o andamento nas mensagens do console.

Ele apaga o que existir antes e baixa um backup automático do estado anterior. Depois disso, o caminho é **Contagem → Contar estoque fechado**, que é o que abre o saldo.

## Zerar o estoque

No fim da aba **Produtos** existe um único botão vermelho, **Zerar estoque**. Ele apaga produtos, contagens e movimentos de uma vez, pede senha e baixa um backup antes.

A senha fica no topo do `index.html`, em `senhaEmergencia` — o padrão é `FINI2026` e vale trocar. É uma trava contra o clique errado, não uma proteção de segurança: quem abrir o código-fonte da página consegue lê-la. Se o risco importar, deixe a política do Supabase exigindo login (a OPÇÃO A do `schema.sql`).

## Como atualizar o site depois

Abra o repositório no GitHub, clique no `index.html`, no ícone de lápis, cole a versão nova e **Commit**. A Vercel publica sozinha em segundos. Não precisa mexer no Supabase de novo — os dados ficam lá, independentes do código.

---

## Rotina sugerida

| Quando | O quê |
|---|---|
| Todo dia, na abertura | Transferir do depósito para o balcão o que for repor |
| Todo dia, no fechamento | Importar a venda do Degust em **Movimentos → Importar vendas** |
| Na hora que acontecer | Lançar desperdício, degustação e quebra — é o que separa perda de venda |
| Toda semana | Olhar **Inteligência**: quem está em "Repor já" e o desperdício da semana em **Relatórios → Por data** |
| Todo mês | Contagem completa do depósito, aplicada. É ela que corrige o acumulado |

---

## Detalhes técnicos

**Armazenamento.** O app procura, nesta ordem: o banco do Claude (quando aberto como artifact), depois o Supabase (se `FINI_CONFIG` estiver preenchido), depois o `localStorage` do navegador. Sem configuração ele funciona, mas os dados ficam só naquele aparelho — bom para testar, ruim para operar.

**Estrutura dos dados.** Uma tabela, quatro coleções:

| Coleção | Documento |
|---|---|
| `produtos` | `{codigo, nome, categoria, tipo, custo, unPorCaixa, pesoUn, codigosAlt, estoqueMin, leadTime, foto, ativo}` |
| `contagens` | `{tipo, status, itens:{sku: qtd}, data, criadoEm, aplicadaEm, responsavel}` |
| `movimentos` | `{direcao, motivo, local, itens:{sku: qtd}, granelKg, data, criadoEm, responsavel, origem}` |
| `config` | `{leadTime, coberturaAlvo, janela}` |

Quantidades sempre na unidade base: **kg** para granel, **unidade** para item. As conversões de caixa e peso acontecem na digitação.

**Imagens.** A foto do produto é guardada dentro do próprio registro, já reduzida para no máximo 480 px e cerca de 80 KB. Não existe serviço de arquivos separado nem link externo que possa quebrar — a imagem vem junto com o produto no backup. No celular, o botão de escolher imagem abre a câmera. Reimportar o catálogo **não** apaga foto, estoque mínimo, prazo próprio nem códigos conciliados.

**Contas da inteligência.**

```
consumo diário   = consumo da janela ÷ dias da janela
cobertura        = saldo ÷ consumo diário
ponto de pedido  = consumo diário × prazo de entrega
compra sugerida  = consumo diário × (prazo + cobertura desejada) − saldo
```

Item consome pela venda; granel consome pela transferência ao balcão.

**Leitura de PDF.** As telas de importação e a **Entrada por PDF** leem o arquivo direto, sem copiar nada. O interpretador (pdf.js) é baixado do cdnjs na primeira vez que você abre um PDF, então essa função precisa de internet. Só funciona com PDF de texto — documento escaneado como imagem não é lido. Na entrada, o app procura no documento os códigos que já existem no cadastro e pega a quantidade da mesma linha; tudo aparece numa tabela editável antes de virar movimento.

**Backup.** No Supabase, **Table Editor → fini_docs → Export as CSV**. Ou use o **Exportar CSV** dentro do próprio app, que sai já legível no Excel.

**Limites.** Foi feito para uma loja. Dois ou três aparelhos usando ao mesmo tempo funcionam bem. Duas pessoas contando o mesmo produto na mesma contagem: vale o último que salvar — por isso a contagem por planilha existe, para dividir o trabalho por trecho.
