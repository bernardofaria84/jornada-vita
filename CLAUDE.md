# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md — Contexto do Projeto ArcaAlliance

## Quem somos

**Bernardo Faria** — fundador e builder técnico do ecossistema **Arca Hub**.
**Dr. Domingos Mantelli** — sócio estratégico, médico.
**Dra. Erica Mantelli** — ginecologista e obstetra, cliente piloto.

---

## O que é o ArcaAlliance

ArcaAlliance é um **SaaS para mentores** — especificamente médicos e profissionais de saúde no Brasil. Compete diretamente com Hotmart, Kiwify, Memberkit e Kajabi, mas com foco vertical em saúde.

**Funcionalidades planejadas para o produto completo:**
- Upload de vídeos com bibliotecas organizadas por curso/mentoria
- Área de membros com controle de acesso por compra
- Sistema de pagamentos integrado
- Dashboard do mentor: membros ativos, inadimplentes, progresso
- Perfil do membro: assinatura, pagamentos, conteúdo

ArcaAlliance faz parte de um ecossistema maior chamado **Arca Hub**, que inclui:
- **ArcaMentoring** — IA que clona o conhecimento do especialista (RAG + embeddings + Whisper + LLMs)
- **ArcaTalk** — comunicação clínica segura
- **ArcaHealth** — ERP médico com transcrição por voz
- **Giul.ia** — IA para comunicações automatizadas

---

## O MVP atual — Piloto com a Dra. Erica Mantelli

### Filosofia do MVP
> Validar o conceito do ArcaAlliance com custo zero ou próximo de zero, usando ferramentas existentes "costuradas" entre si, antes de construir o produto real.

### O produto piloto
**Jornada Vita** — mentoria de saúde feminina da Dra. Erica Mantelli.
- 8 semanas de conteúdo
- Até 100 participantes (turma inaugural)
- Preço: R$ 1.497 à vista
- Bônus: encontro presencial para as 50 primeiras

---

## Stack Técnica do MVP

### Domínio e hospedagem
- **`ericamantelli.com.br`** — domínio principal da Dra. Erica
- **Hostinger** — hospedagem (PHP 8.3.30), com backup automático diário
- **WordPress + Hello Elementor** — CMS e page builder
- **WooCommerce** — e-commerce e gestão de membros

### Landing page
- Construída em WordPress com Elementor no domínio `ericamantelli.com.br`
- Todo o conteúdo de venda, CTAs, depoimentos, etc.

### Checkout e pagamentos
- **WooCommerce** como carrinho e checkout
- **e-Rede Itaú** como gateway de pagamento (cartão + PIX)
- Produto: ID `108` no WooCommerce, R$ 1.497

### Vídeos
- **YouTube Não-Listado** (não Privado — Privado não embeda e tem limite de 50 pessoas)
- Parâmetros de proteção no embed: `rel=0`, `modestbranding=1`, `iv_load_policy=3`, `cc_load_policy=0`, `playsinline=1`
- Overlays transparentes sobre o player para bloquear cliques em links externos
- Cada semana tem um ID de vídeo do YouTube substituível no HTML

### Área de membros
- **Página WordPress** (`ericamantelli.com.br/jornada-vita-area-de-membros/`) protegida por PHP server-side
- Proteção em duas camadas:
  1. Não logado → redirect para `/minha-conta/`
  2. Logado sem compra → redirect para `/produto/jornada-vita/`
- O conteúdo real é um **`membros.html` hospedado no GitHub Pages** (`bernardofaria84.github.io/jornada-vita/membros.html`) embedado via `<iframe>` na página WordPress
- Comunicação iframe ↔ WordPress via `postMessage`

### Sistema de progresso
- **WordPress AJAX** + **`wp_usermeta`** — 100% dinâmico e real
- Fluxo: aluna clica "Concluí esta aula" → iframe envia `postMessage` → Elementor captura → AJAX salva em `wp_usermeta` com `meta_key = jornada_vita_progresso`
- JSON salvo: `{"s1": {"concluida": true, "data": "2026-04-22 22:36:00"}}`
- Persiste entre sessões e dispositivos
- Segurança: nonce WordPress em cada requisição AJAX

### Conta da Mentora (Dra. Erica)
- Usuário WordPress com role customizada **"Mentora"** — user ID `5`, login `dra.erica`
- Ao logar: vai direto para o dashboard de progresso (não para "Minha Conta")
- Vê apenas: **Jornada Vita** (dashboard) + **Análises** (WooCommerce)
- Não vê: Posts, Páginas, Plugins, Temas, WooCommerce operacional, Usuários, etc.

### Dashboard da Dra. Erica
- Página dentro do `wp-admin` (`admin.php?page=jornada-vita-dashboard`)
- Dados 100% reais do banco de dados:
  - Cards: total de alunas, média de progresso, concluíram tudo, sem atividade
  - Tabela: nome clicável, barra de progresso %, badges S1-S8 verdes/cinza, última atividade, data de inscrição
  - Ficha individual: clica no nome → abre página com dados cadastrais + progresso detalhado por semana
- Busca dados de: `wp_usermeta` (progresso) + `wc_get_orders` (pedidos) + `get_userdata` (cadastro)

---

## Arquitetura de comunicação iframe ↔ WordPress

```
membros.html (GitHub Pages)
    │
    │── postMessage({ type: 'iframeReady' }) ──────────────▶ Elementor
    │◀── postMessage({ type: 'carregarProgresso', ... }) ──── Elementor ◀── AJAX ◀── wp_usermeta
    │
    │── postMessage({ type: 'concluirSemana', semana: 's1' }) ▶ Elementor
    │◀── postMessage({ type: 'progressoSalvo', ... }) ──────── Elementor ◀── AJAX ◀── wp_usermeta
    │
    │── postMessage({ type: 'iframeHeight', height: N }) ────▶ Elementor (ajusta altura do iframe)
```

---

## Fluxo completo da aluna

```
1. Acessa ericamantelli.com.br → landing page
2. Clica CTA → checkout WooCommerce → paga via e-Rede
3. Conta criada automaticamente pelo WooCommerce
4. Email de confirmação enviado automaticamente
5. Acessa /minha-conta/ → menu "► Acessar Jornada Vita" (só aparece se comprou)
6. Entra em /jornada-vita-area-de-membros/ → carrega membros.html via iframe
7. WordPress injeta progresso salvo → semanas concluídas aparecem em verde
8. Assiste vídeo → clica "✓ Concluí esta aula" → salva no banco → badge vira verde
9. Na próxima visita: progresso já aparece correto automaticamente
```

---

## Fluxo da Dra. Erica (Mentora)

```
1. Login em ericamantelli.com.br/ffwd/ (URL customizada pelo WPS Hide Login)
2. Redireciona automaticamente para o dashboard Jornada Vita
3. Vê todas as alunas com progresso em tempo real
4. Clica no nome de uma aluna → ficha com dados cadastrais e progresso detalhado
5. Acessa Análises → relatórios de vendas, pedidos, receita do WooCommerce
```

---

## Acesso ao servidor

```
ssh -p 65002 u290500739@185.245.180.57
```

- Autenticação por chave SSH (sem senha)
- WordPress em: `~/domains/ericamantelli.com.br/public_html/`
- WP-CLI disponível: `wp --path=~/domains/ericamantelli.com.br/public_html <comando>`
- Também existe: `~/domains/clinicamantelli.com.br/` (outro site, não mexer)

### Operações comuns via SSH

```bash
# Ler functions.php
cat ~/domains/ericamantelli.com.br/public_html/wp-content/themes/hello-elementor/functions.php

# Editar capabilities da role Mentora
wp --path=~/domains/ericamantelli.com.br/public_html eval '...'

# Verificar capabilities de um usuário
wp --path=... user list --role=mentora
wp --path=... eval 'print_r(get_role("mentora")->capabilities);'

# Testar endpoint REST como Mentora (user ID 5)
wp --path=... eval '
wp_set_current_user(5);
$req = new WP_REST_Request("GET", "/wp/v2/settings");
$res = rest_do_request($req);
echo $res->get_status();'
```

---

## Arquivos críticos do projeto

### `functions.php` (tema Hello Elementor)
Localização: `wp-content/themes/hello-elementor/functions.php`

Contém todas as funções customizadas da Jornada Vita:
1. `jornada_vita_usuario_comprou()` — verifica pedido completed/processing com produto ID 108
2. `jornada_vita_proteger_area_membros()` — PHP redirect server-side
3. `jornada_vita_adicionar_menu_minha_conta()` — menu "Acessar Jornada Vita" só para quem comprou
4. `jornada_vita_remover_menus_minha_conta()` — remove Downloads
5. `jornada_vita_url_menu_minha_conta()` — URL da área de membros
6. `jornada_vita_salvar_progresso()` — wp_ajax, salva em user_meta com nonce
7. `jornada_vita_carregar_progresso()` — wp_ajax, retorna progresso + total + nome
8. `jornada_vita_injetar_dados()` — wp_localize_script JornadaVita {ajaxurl, nonce, nome}
9. `jv_registrar_role_mentora()` — role 'mentora' com capabilities específicas
10. `woocommerce_prevent_admin_access` filter — libera mentora no wp-admin
11. `jv_redirect_mentora_apos_login()` — login_redirect para o dashboard
12. `jv_mentora_nao_vai_para_minha_conta()` — woocommerce_login_redirect
13. `jv_mentora_woocommerce_rest_check()` — libera API REST do WooCommerce para mentora
14. `jv_remover_menus_mentora()` — remove menus desnecessários no wp-admin
15. `jv_mentora_admin_css()` — esconde Pagamentos, Marketing e avisos técnicos via CSS
16. `jv_dashboard_menu()` — registra menu "Jornada Vita" no wp-admin
17. `jv_dashboard_styles()` — CSS do dashboard (cores da identidade visual)
18. `jv_ficha_aluna()` — renderiza ficha individual de cada aluna
19. `jv_dashboard_render()` — renderiza o dashboard completo

### `membros.html` (GitHub Pages)
Localização: `bernardofaria84.github.io/jornada-vita/membros.html`

Contém:
- Layout visual completo da área de membros (design Jornada Vita)
- 8 cards de semanas com estados: Em breve / Ao vivo / Assistir / Concluída
- Player YouTube embedado com overlays de proteção
- Sistema de progresso visual (badges, barra, contadores)
- Comunicação via postMessage com o WordPress pai
- Função `concluirAula('sX')` chamada pelos botões
- Função `reportHeight()` para eliminar scroll duplo

### Elementor HTML Widget (página área de membros)
- iframe com auto-resize via postMessage
- Bridge de comunicação WordPress ↔ iframe
- Recebe `iframeReady` → carrega progresso via AJAX
- Recebe `concluirSemana` → salva via AJAX → retorna confirmação

---

## Role Mentora — capabilities obrigatórias

```php
array(
    'read'                     => true,
    'manage_woocommerce'       => true,
    'view_woocommerce_reports' => true,
    'view_admin_dashboard'     => true,
    'manage_options'           => true,  // OBRIGATÓRIA para WC Analytics (ver seção abaixo)
)
```

**Por que `manage_options` é necessária**: O React app do WooCommerce Analytics chama `/wp/v2/settings` durante a inicialização. Esse endpoint WordPress requer `manage_options`. Sem ela, o app crasha silenciosamente e a página Análises fica em branco. Os menus de Configurações do WordPress são removidos via `jv_remover_menus_mentora()`, então a Mentora não acessa as páginas de settings pela UI.

Se um futuro Claude remover `manage_options` da role Mentora, o menu Análises voltará a ficar em branco.

---

## WooCommerce Analytics — como funciona internamente

O menu "Análises" (`wc-admin&path=/analytics/overview`) é um **React Single-Page App** que monta em `<div id="root">`. Quando a página carrega:

1. PHP renderiza apenas `<div class="wrap"><div id="root"></div></div>`
2. JavaScript carrega `woocommerce/assets/client/admin/app/index.js` (247KB minificado)
3. O app inicializa fazendo chamadas REST API em sequência:
   - `GET /wp/v2/settings` — **requer `manage_options`** — busca configurações do site
   - `GET /wp/v2/users/me` — dados do usuário atual
   - `GET /wc-analytics/reports/*` — dados de analytics
4. Se qualquer chamada inicial falhar com 403, o app crasha sem mensagem de erro visível

O `wcSettings` (objeto global injetado no HTML via PHP) inclui `currentUserIsAdmin: true` para qualquer usuário com `manage_woocommerce`, mas isso não é suficiente — o endpoint `/wp/v2/settings` tem sua própria verificação independente.

---

## Identidade visual da Jornada Vita

| Token | Valor |
|---|---|
| Marrom escuro (primário) | `#2C2416` |
| Dourado (accent) | `#C19C6E` |
| Creme (fundo) | `#FAF7F2` |
| Marrom suave | `#9C8B76` |
| Bege | `#E4D9CC` |
| Verde conclusão | `#4A7C59` |
| Fonte títulos | Cormorant Garamond |
| Fonte corpo | Jost |

---

## Dados técnicos importantes

| Item | Valor |
|---|---|
| ID produto WooCommerce | `108` |
| Meta key progresso | `jornada_vita_progresso` |
| Slug página membros | `jornada-vita-area-de-membros` |
| URL login customizada | `ericamantelli.com.br/ffwd/` |
| GitHub Pages URL | `bernardofaria84.github.io/jornada-vita/membros.html` |
| PHP version | 8.3.30 |
| WordPress version | 6.9.4 |
| WordPress theme | Hello Elementor |
| AJAX actions | `jv_salvar_progresso`, `jv_carregar_progresso` |
| Nonce key | `jornada_vita_nonce` |
| Mentora user ID | `5` |
| Mentora login | `dra.erica` |
| SSH host | `185.245.180.57:65002` |
| SSH user | `u290500739` |
| WP root no servidor | `~/domains/ericamantelli.com.br/public_html/` |

---

## Decisões de arquitetura e seus motivos

| Decisão | Motivo |
|---|---|
| YouTube Não-Listado vs Privado | Privado não embeda e tem limite de 50 pessoas |
| GitHub Pages para membros.html | Hospedagem gratuita, deploy instantâneo via git push |
| iframe + postMessage vs página WP direta | Preserva design customizado sem depender do Elementor |
| WordPress AJAX vs Google Forms | Progresso multi-dispositivo, zero fricção, salva no perfil do usuário |
| Proteção PHP server-side vs JavaScript | JS pode ser burlado; PHP roda antes da página carregar |
| Hello Elementor theme | Neutro, não interfere no design Elementor |
| e-Rede Itaú | Gateway já existente do cliente, integração WooCommerce nativa |
| `manage_options` na role Mentora | WC Analytics React app exige essa cap para inicializar via REST |

---

## Armadilhas conhecidas (não repetir)

### 1. CSS agressivo no wp-admin da Mentora
O seletor `[class*="woocommerce-"][class*="-notice"]` corresponde a elementos do React app como `woocommerce-layout__notice-list`, não apenas a avisos PHP. Usar seletores específicos no CSS de `jv_mentora_admin_css()`. Classes seguras para esconder avisos:
```css
.woocommerce-store-alerts, .woocommerce-message, .wc-admin-notice,
.notice.woocommerce-notice, .components-notice-list,
.woocommerce-layout__notice-list-hide
```

### 2. `remove_role()` + `add_role()` para atualizar capabilities
Simplesmente editar o código de `jv_registrar_role_mentora()` não atualiza as capabilities já salvas no banco de dados. Para forçar a atualização imediata:
```bash
wp --path=~/domains/ericamantelli.com.br/public_html eval '
remove_role("mentora");
add_role("mentora", "Mentora", array(
    "read" => true,
    "manage_woocommerce" => true,
    "view_woocommerce_reports" => true,
    "view_admin_dashboard" => true,
    "manage_options" => true,
));'
```

### 3. WooCommerce Analytics página em branco
Se o menu "Análises" abrir em branco para a Mentora, verificar primeiro:
```bash
wp --path=... eval '
wp_set_current_user(5);
$r = rest_do_request(new WP_REST_Request("GET", "/wp/v2/settings"));
echo $r->get_status(); // deve ser 200, não 403'
```
Se retornar 403, a role perdeu `manage_options`.

### 4. `woocommerce_rest_check_permissions` não cobre todos os endpoints
O filtro `jv_mentora_woocommerce_rest_check` cobre endpoints da API WooCommerce clássica, mas endpoints do namespace `wc-analytics/` têm seus próprios permission callbacks e podem precisar de tratamento separado.

### 5. `remove_menu_page('woocommerce')` não apaga `$submenu['woocommerce']`
WordPress separa `$menu` de `$submenu`. Remover o item principal do `$menu` não limpa `$submenu['woocommerce']`, que o WooCommerce usa internamente para registrar páginas do Analytics. Isso é o comportamento correto — não tentar limpar `$submenu` manualmente.

---

## Status do MVP (abril 2026)

### Funcionando
- Landing page completa
- Checkout WooCommerce + e-Rede (testado com pagamento + estorno)
- Criação automática de conta no checkout
- Proteção server-side da área de membros por compra
- Menu "Acessar Jornada Vita" condicional por compra
- Área de membros com iframe + design completo
- Sistema de progresso AJAX — salva e carrega do banco de dados real
- Badges de conclusão por semana (verde/cinza)
- Barra de progresso e contadores dinâmicos
- Player YouTube com overlays de proteção
- Botão "← Minha Conta" com target="_parent"
- Dashboard da Dra. Erica com dados reais
- Ficha individual clicável por aluna
- Conta Mentora com acesso limitado ao wp-admin
- Redirect automático ao logar para o dashboard
- Menu Análises (WooCommerce Analytics) funcionando para a Mentora

### Pendente antes do lançamento
- Trocar preço de R$ 4 (teste) para R$ 1.497
- Ativar e-Rede em modo produção (hoje: sandbox)
- Substituir vídeo de teste pelo real da Semana 1
- Teste completo ponta a ponta com compra real no valor correto

---

## Plano de Negócios — ArcaAlliance (Alto Nível)

### Posicionamento estratégico

**Tese central:** ArcaAlliance não é uma plataforma de cursos — é uma **fintech disfarçada de plataforma de cursos**. O produto (área de membros, dashboard, gestão de alunos) é o cavalo de Tróia para capturar o float financeiro dos criadores.

**Diferença fundamental vs. concorrentes:**

| | Hotmart / Kiwify | ArcaAlliance |
|---|---|---|
| Modelo de receita | Comissão por venda (9,9%+) | Float financeiro sobre saldo custodiado |
| Incentivo | Cobrar mais por venda | Manter dinheiro na plataforma mais tempo |
| Alinhamento com criador | Conflitante (quanto mais ele vende, mais paga) | Alinhado (quanto mais ele vende e cresce, mais você ganha) |
| Nicho | Genérico | Vertical: profissionais de saúde |

---

### Modelo de receita

**Fonte primária — Float Investment:**
- Criadores hospedam cursos e recebem pagamentos via ArcaAlliance
- O saldo custodiado é aplicado automaticamente em CDI overnight via parceiro BaaS
- ArcaAlliance fica com o spread entre o rendimento bruto e o que repassa ao criador
- Taxa de comissão por venda: **zero**

**Estrutura de repasse ao criador (taxas decrescentes por tempo):**

| Tempo com saldo na plataforma | Criador recebe | ArcaAlliance retém |
|---|---|---|
| Saque imediato (D+1) | 0% do CDI | 100% |
| 7 dias | 30% do CDI | 70% |
| 30 dias | 50% do CDI | 50% |
| 90 dias | 70% do CDI | 30% |
| 180+ dias | 85% do CDI | 15% |

**Fontes secundárias (futuro):**
- Antecipação de recebíveis — criador antecipa vendas parceladas com deságio
- Crédito com base no histórico de vendas na plataforma
- Planos premium com features avançadas (IA, automações)

---

### Projeção financeira simplificada

Premissa: CDI ~10,5% a.a., criador médio fatura R$ 8k/mês na plataforma, taxa média de retenção de saldo de 45 dias.

| Criadores ativos | Float médio custodiado | Receita bruta anual | Margem estimada (50%) |
|---|---|---|---|
| 10 | R$ 120k | R$ 12.6k | R$ 6.3k |
| 50 | R$ 600k | R$ 63k | R$ 31.5k |
| 200 | R$ 2.4M | R$ 252k | R$ 126k |
| 500 | R$ 6M | R$ 630k | R$ 315k |
| 2.000 | R$ 24M | R$ 2.52M | R$ 1.26M |

O modelo só se torna relevante a partir de ~200 criadores ativos. Abaixo disso, o produto SaaS e a prova de conceito são o foco — não a lucratividade.

---

### Estratégia de entrada no mercado

#### Fase 1 — Nicho vertical (0–12 meses)
**Foco exclusivo: profissionais de saúde.**

Hotmart e Kiwify são genéricos. Um médico prefere uma plataforma feita para médicos. O Dr. Domingos é o principal canal de aquisição — acesso direto a redes médicas, conselhos regionais, grupos de especialistas.

Meta: **50 criadores de saúde ativos** ao final do ano 1.

Ações:
- Usar a Jornada Vita (Dra. Erica) como caso de sucesso público
- Palestras e presença em congressos médicos via Dr. Domingos
- Programa de indicação: criador que indica outro criador ganha melhores taxas de repasse CDI
- Onboarding white glove: ArcaAlliance migra tudo do Hotmart/Kiwify pelo criador

#### Fase 2 — Cliente âncora (meses 3–9)
Identificar e fechar **1 criador de saúde que fature R$ 200k+/mês**. Um único criador nesse patamar gera R$ 2.4M/ano de float — resolve o problema de volume sozinho e valida o modelo financeiro.

Como chegar nele:
- Mapeamento dos top 20 médicos criadores de conteúdo no Brasil (YouTube, Instagram)
- Proposta personalizada: 0% de comissão + 90% do CDI por 12 meses + suporte dedicado
- O custo desse subsídio é calculável e limitado; o valor de tê-lo como referência é multiplicador

#### Fase 3 — Infraestrutura financeira (meses 6–18)
Só após validar o modelo com criadores reais, investir na estrutura BaaS:

- Contratar parceiro BaaS: **Celcoin** (maior BaaS BR), **Swap** (mais acessível para startups) ou **QI Tech**
- Obter licença de Instituição de Pagamento junto ao BACEN — ou operar sob licença do parceiro BaaS
- Lançar conta digital ArcaAlliance: saldo rende automaticamente, paga fornecedores, antecipa recebíveis

---

### Requisito regulatório (não ignorar)

Custodiar dinheiro de terceiros e rentabilizá-lo requer licença do **Banco Central** como Instituição de Pagamento (IP), ou operação sob licença de uma IP parceira (BaaS).

**Caminho recomendado para o MVP financeiro:**
1. Parceria com BaaS licenciado (Celcoin, Swap, Dock)
2. ArcaAlliance opera como **subconta** da IP parceira
3. O BaaS cuida da custódia, liquidação e compliance; você cuida do produto

Sem isso, não é possível legalmente segurar e rentabilizar o dinheiro dos criadores.

---

### Plano de 90 dias (ações concretas)

| Semana | Ação | Responsável |
|---|---|---|
| 1–2 | Lançar Jornada Vita com a Dra. Erica (produção + vendas) | Bernardo + Dra. Erica |
| 1–4 | Mapear top 20 criadores de saúde do Brasil | Bernardo |
| 2–4 | Estruturar proposta comercial ArcaAlliance para médicos | Bernardo + Dr. Domingos |
| 3–6 | Primeira reunião com parceiro BaaS (Celcoin ou Swap) | Bernardo |
| 4–8 | Fechar primeiros 5 criadores de saúde via rede Dr. Domingos | Dr. Domingos |
| 6–12 | Identificar e abordar cliente âncora (criador R$ 200k+/mês) | Bernardo + Dr. Domingos |
| 8–12 | Versão 1 do produto ArcaAlliance (não WordPress — plataforma real) | Bernardo |

---

### Riscos principais e mitigações

| Risco | Probabilidade | Mitigação |
|---|---|---|
| Volume insuficiente para margem financeira relevante | Alta (ano 1) | Aceitar: ano 1 é validação, não lucratividade |
| Barreira regulatória BaaS | Média | Iniciar conversa com BaaS cedo; operação sem float no MVP |
| Criador não deixa dinheiro na plataforma | Média | Gamificar incentivos; integrar pagamentos de fornecedores dentro da plataforma |
| Hotmart/Kiwify copia o modelo (zero taxa) | Baixa no curto prazo | Eles dependem de comissão para sobreviver; não podem zerar facilmente |
| Concentração em 1 nicho (saúde) | Baixa | É uma vantagem no início; expansão horizontal vem depois da prova de conceito |

---

### Visão de longo prazo (3–5 anos)

```
Ano 1: Piloto nicho saúde, 50 criadores, prova de conceito do float
Ano 2: Licença IP + conta digital ArcaAlliance, 200 criadores, R$ 2.4M float
Ano 3: Expansão para outros nichos (direito, finanças, educação), 1.000 criadores
Ano 4: Crédito para criadores baseado em histórico de vendas, antecipação de recebíveis
Ano 5: ArcaAlliance como banco dos criadores de conteúdo no Brasil
```

O produto de cursos vira infraestrutura. O negócio real é financeiro.

---

## Como liberar uma nova semana (operação recorrente)

1. Dra. Erica grava a aula → sobe como **Não-Listado** no YouTube
2. Pega o ID do vídeo (parte após `v=` na URL)
3. No `membros.html`, localiza o card da semana e substitui o ID placeholder
4. Muda as classes CSS do card:
   - `Em breve` → `Ao Vivo`: classes `aovivo`, badge pulsante vermelho, live-banner
   - `Ao Vivo` → `Assistir`: classes `ativa`, badge dourado, botão "Concluí"
   - `Assistir` → `Concluída`: acontece automaticamente quando a aluna clica o botão
5. Git commit + push → GitHub Pages atualiza em segundos
6. Zero downtime, zero custos adicionais
