# Portal Ecguaíba — www.ecguaiba.com.br

Cópia de trabalho do código do portal que roda na hospedagem (HostGator, cPanel,
PHP 7.4). **O servidor é o que vale**: esta pasta serve para consulta, histórico e
para refazer uma instalação. Ao mudar algo no servidor, atualize a cópia daqui.

Atualizado em 27/09/2026.

---

## 1. O que existe no site

| Endereço | O que é | Quem entra |
|---|---|---|
| `/` | Site institucional (WordPress) | Todos |
| `/acesso/` | Tela de login do portal | — |
| `/ecguaiba/` | Página inicial da Ecguaíba: cartões dos painéis (desde 30/09/2026) | Equipe (Microsoft 365) |
| `/contabsul/` | Página inicial da Contabsul: cartões dos painéis (desde 30/09/2026) | Grupo Contabsul (`grupo_contabsul_id`) |
| `/acesso/escolher.php` | "Escolha a área" — quem entra sem destino e tem os dois escritórios | — |
| `/intranet/` | Base de Conhecimento Fiscal (HTML estático) — **abandonada por enquanto**: no ar, mas sem links e fora do destino padrão | Equipe (Microsoft 365) |
| `/clientes/equipe/` | Painel da carteira (Planejamento RTC) — **adormecido** desde 30/09/2026 (no ar, sem cartão) | Equipe (Microsoft 365) |
| `/clientes/<cliente>/` | Página de cada cliente (ex.: `/clientes/madevan/`) | Equipe e o usuário do cliente |
| `/clientes/` | Página neutra | Qualquer usuário logado |
| `/RTC_Ecguaiba/` | Painel RTC — Comunicação aos Clientes | Equipe (Microsoft 365) |
| `/nfse_ecguaiba/` | Resumo da importação de NFS-e (só consulta) — desde 30/09/2026 | Equipe (Microsoft 365) |
| `/nfse_dominio_ecguaiba/` | Serviços prestados — NFS-e x Domínio x Prefeitura (só consulta) — desde 30/09/2026 | Equipe (Microsoft 365) |
| `/cadastro_ecguaiba/` | Painel Matriz Cadastral (time de cada empresa editável) | Equipe (Microsoft 365) |
| `/Cadastro_Contabsul/` | Painel Matriz Cadastral da Contabsul | Grupo do RTC Contabsul (`grupo_contabsul_id`) |
| `/nfse_contabsul/` | Resumo da importação de NFS-e da Contabsul (só consulta) — desde 30/09/2026 | Grupo Contabsul (`grupo_contabsul_id`) |
| `/nfse_dominio_contabsul/` | Serviços prestados — NFS-e x Domínio x Prefeitura da Contabsul (só consulta) — desde 30/09/2026 | Grupo Contabsul (`grupo_contabsul_id`) |

**Logins**

- **Equipe:** botão "Entrar com a Microsoft", com as contas @ecguaiba.com.br do grupo
  **`Ecguaíba - Equipe`** no Entra ID (inclui os usuários da Contabsul). Para dar ou tirar
  acesso, mexa só no grupo.
- **Clientes:** usuário e senha do arquivo de senhas do servidor
  (`/home1/ecgua152/.htpasswds/public_html/clientes/passwd`). Cada cliente só abre a própria
  pasta. As senhas em texto claro ficam em
  `ecguaiba_com_br - Documentos\Intranet\credenciais.json`.
- O antigo usuário **`equipe`** (acesso de emergência) foi **desativado**: a tela de login
  recusa esse usuário.

---

## 2. Onde fica cada coisa no servidor

```
/home1/ecgua152/
├── public_html/
│   ├── .htaccess              trecho "Ecguaiba - areas estaticas fora do WordPress"
│   │                          (ver servidor/public_html/htaccess-raiz-trecho.txt)
│   ├── acesso/                login, porteiro e Microsoft 365          (servidor/public_html/acesso)
│   ├── intranet/  .htaccess   encaminha tudo ao porteiro              (gerado pelo publicar_intranet.py)
│   ├── clientes/  .htaccess   idem, em cada subpasta                  (gerado pelo publicar_intranet.py)
│   ├── RTC_Ecguaiba/          painel RTC                               (servidor/public_html/RTC_Ecguaiba)
│   └── cadastro_ecguaiba/     painel matriz cadastral                  (XML - ecguaiba_com_br\01 Cadastro de Empresas)
└── ecg_portal/                FORA do site (não é acessível pela web)
    ├── config.php             configuração + segredo da Microsoft (permissão 0600)
    │                          modelo sem segredo: servidor/ecg_portal/config.exemplo.php
    ├── sessoes/               sessões de login
    ├── logs/acesso-AAAA-MM.log  cada login, recusa, prévia e atualização aplicada
    ├── tentativas.json        controle de senha errada (5 erros = 15 min de bloqueio)
    └── rtc/                   dados do painel RTC
        ├── base.json          tributação calculada (+ versão de cada arquivo usado)
        ├── ajustes.json       grupos escolhidos e observações   <- dado da equipe
        ├── ajustes-log.jsonl  histórico de cada alteração (quem, quando, o quê)
        ├── painel_modelo.html visual do painel
        ├── mensagens_padrao.json
        └── historico/         cópias anteriores de base e ajustes
    └── cadastro/              dados do painel matriz cadastral
        ├── painel_cache.html  cópia do painel_matriz.html do SharePoint (renovada quando muda)
        └── classificacao-log.jsonl  histórico de cada troca de time (quem, quando, de -> para)
```

**Como funciona o porteiro (`acesso/porteiro.php`):** o `.htaccess` de `/intranet` e
`/clientes` manda todo pedido para o porteiro, que confere quem está logado e se pode abrir
aquele caminho; só então entrega o arquivo. Arquivos ocultos (`.htaccess`, backups) e
`.php` nunca são entregues. As páginas HTML recebem um selo "nome · Sair" no canto.

---

## 3. Microsoft 365 (Entra ID)

- Aplicativo: **Portal Ecguaíba**, ID `ffe9ba60-0c8b-487d-a1ed-392036a0ef9b`,
  locatário `8e3e5c0f-d523-4528-a088-a74b79aa66cb`, só contas da organização.
- URI de redirecionamento: `https://www.ecguaiba.com.br/acesso/m365-retorno.php`.
- Permissões delegadas (com consentimento do administrador): `User.Read`,
  `Files.ReadWrite.All` (desde 28/09/2026; antes `Files.Read.All`) e `offline_access`. Os painéis
  leem o SharePoint **em nome de quem está logado**; só o `/cadastro_ecguaiba/` grava (no
  `classificacao_empresas.json`), e só consegue se a pessoa puder editar a biblioteca Base de Dados.
- Grupo de acesso: **Ecguaíba - Equipe** (`ea38a5f4-8cea-4062-ad19-a0b9d0aeba0c`),
  atribuído ao aplicativo, com "Atribuição necessária = Sim".
- **`eh_equipe()` confere o grupo de verdade** (a partir de 27/09/2026): antes só checava "logou
  com alguma conta Microsoft 365"; agora confere também se a conta está no grupo "Ecguaíba -
  Equipe" (`m365.grupo_id`), via `pertence_grupo()`. Isso vale para tudo que usa `eh_equipe()`:
  `/intranet/`, `/clientes/equipe/` e `/RTC_Ecguaiba/`. A lista de grupos vem da claim `groups`
  do id_token e é guardada na sessão no login (`m365-retorno.php` → `abrir_sessao()`).
- **Grupo extra do `/RTC_Contabsul/`** (a partir de 27/09/2026): esse painel específico tem seu
  próprio grupo do Entra, dedicado só a ele, diferente do "Ecguaíba - Equipe" — conferido em
  código (`grupo_contabsul_id` no `config.php`), de forma independente de `eh_equipe()` (usa
  `eh_m365()` + `pertence_grupo()` — ver `_rtc.php` do projeto RTC - Contabsul). Quem estiver
  nesse grupo mas fora de "Ecguaíba - Equipe" não consegue nem logar, a não ser que esse grupo
  também seja atribuído à Enterprise Application "Portal Ecguaíba" — ver o comentário completo em
  `servidor_config/config-contabsul.trecho.txt` do projeto RTC - Contabsul.
- **Segredo do aplicativo:** vence em 24 meses a partir da criação (26/09/2026). Antes de
  vencer: Entra → Registros de aplicativo → Portal Ecguaíba → Certificados e segredos →
  Novo segredo; cole o **Valor** em `ecg_portal/config.php`, linha `client_secret`
  (cPanel → Gerenciador de arquivos → Editar). Sem isso o botão da Microsoft some.
- Não guarde o segredo em arquivo nesta pasta (ela é sincronizada com o SharePoint).

---

## 4. Painel RTC (`/RTC_Ecguaiba/`)

Roda inteiro no servidor. As bases são lidas da biblioteca **Base de Dados** do site
**XML** no SharePoint (`drive` configurado em `rtc_drive_id`):

| Arquivo (na biblioteca Base de Dados) | Obrigatório |
|---|---|
| `01 Cadastro Empresas/dados_dominio/dominio_empresas.json` | sim |
| `01 Cadastro Empresas/dados_rfb/estabelecimentos_rfb.json` | sim |
| `01 Cadastro Empresas/dados_rfb/simples_rfb.json` | sim |
| `01 Cadastro Empresas/dados_rfb/empresas_rfb.json` | não |
| `01 Cadastro Empresas/dados_rfb/socios_rfb.json` | não |
| `RTC - Ecguaiba - Analise Opcao Simples/anexos_simples_cnaes.json` | sim |
| `RTC - Ecguaiba - Analise Opcao Simples/faturamento_dominio_12m.json` | não |
| `RTC - Ecguaiba - Analise Opcao Simples/mensagens_grupos.json` | não |
| `RTC - Empresas Regime Regular/matriz_impacto_cbs.json` | não |

A lista fica em `RTC_Ecguaiba/_rtc.php` (`RTC_FONTES`). **Se algum arquivo mudar de pasta,
ajuste ali** e envie o `_rtc.php` ao servidor. Desde 27/09/2026 a cópia local desse e dos
demais arquivos de `RTC_Ecguaiba/` (`_calculo.php`, `_painel.php`, `api.php`, `chave.php`,
`index.php`, `.htaccess`) NÃO fica mais aqui: mora em
`XML - Projetos\RTC - Ecguaiba - Analise Opcao Simples\servidor_php`, por ser código
específico do painel RTC, não do portal em geral — ver LEIA-ME de lá.

**Fluxo de atualização:** ao abrir o painel, o servidor compara esses arquivos com os usados
no último cálculo → se algo mudou, aparece o aviso → "Ver o que muda" recalcula e mostra a
prévia (não grava) → "Aplicar atualização" grava a base nova. Grupos e observações não são
tocados. Qualquer pessoa da equipe pode aplicar; fica registrado quem aplicou.

**Cálculo:** `RTC_Ecguaiba/_calculo.php` é o porte do `calcular()`/`classificar()` do
`rtc_comunicacao_opcao_simples.py` (projeto `XML - Projetos\RTC - Ecguaiba - Analise Opcao Simples`,
que é irmão — não mais subpasta — do `01 Cadastro de Empresas`).
Conferido em 26/09/2026: mesmo resultado do Python, 130 matrizes, zero diferenças.
**Se mudar uma regra no Python, mude igual no `_calculo.php`.** Isso não é automático nem
periódico: é um passo manual só quando uma regra de cálculo muda (raro). A cópia local desse
e dos demais arquivos PHP do painel (`_painel.php`, `_rtc.php`, `api.php`, `chave.php`,
`index.php`, `.htaccess`) fica em
`XML - Projetos\RTC - Ecguaiba - Analise Opcao Simples\servidor_php` (não mais aqui no
Portal Ecguaiba, desde 27/09/2026) — depois de editar, envie o arquivo ao servidor pelo
cPanel, como sempre.

**Visual:** o `painel_modelo.html` vem do `build_html()` do mesmo script, gerado por
`gerar_modelo_web.py`. Ao mudar o visual no Python: rode `gerar_modelo_web.py` (sem
nenhuma flag) para conferir uma prévia com dados reais em
`XML - web_validacao\RTC - Ecguaiba - Analise Opcao Simples` e, só depois de validada essa
prévia, rode de novo com `--publicar` — ele grava `painel_modelo.html` e
`mensagens_padrao.json` em
`XML - ecguaiba_com_br\RTC - Ecguaiba - Analise Opcao Simples` (pasta de staging do FTP,
já FORA desta pasta do Portal Ecguaiba — ver LEIA-ME do projeto RTC) e,
com `--enviar-ftp`, também envia os dois arquivos direto para `ecg_portal/rtc/` no servidor
por FTPS (ver `ftp_rtc.py`; credenciais em `C:\Ecguaiba\ftp_rtc_config.json`, fora do
OneDrive). Sem `--enviar-ftp`, os arquivos só ficam na pasta de staging para o envio manual
de sempre (Core FTP, AUTH TLS, binário). Assim como o `_calculo.php`, isso só é rodado
quando o visual muda — não é um passo periódico. O `build_html()` precisa manter o trecho
do modo web (procure por `RTC_SERVIDOR`).

**Domínio:** o banco só é acessível no escritório. A tarefa agendada
**"Ecguaiba\RTC - exportar Dominio"** (criada pelo `agendar_exportacao_dominio.bat`) roda
todo dia às 06:30 o `rtc_exportar_dominio.bat`, que grava o cadastro e o faturamento de 12
meses no SharePoint; o painel percebe e avisa.

**Versão local de conferência:** `rtc_comunicacao_opcao_simples.py` também gera, a cada
recálculo, um `rtc_comunicacao_opcao_simples.html` em
`XML - web_validacao\RTC - Ecguaiba - Analise Opcao Simples` — pasta só para painéis sem
uso operacional real, que a equipe usa apenas para conferir visualmente (ver LEIA-ME do
projeto RTC). Antes de montar esse HTML, o script baixa por FTP o `ajustes.json`
publicado aqui no servidor (mesma conta/config do envio) para os grupos e observações
mostrados baterem com o que já está no ar; a base local (`rtc_comunicacao_base.json`)
nunca é alterada por esse download. Além do aviso fixo no topo dizendo que é uma versão
apenas para conferência, esse HTML é somente leitura de forma estrutural (não só visual):
qualquer clique nos controles de edição é interceptado e cancelado por um script travado
antes do painel carregar, então nenhuma alteração é possível ali — só no painel web.

---

## 4b. Painel Matriz Cadastral (`/cadastro_ecguaiba/`)

Desde 28/09/2026 (publicado). Código em `XML - ecguaiba_com_br\01 Cadastro de Empresas`
(`_cadastro.php`, `index.php`, `api.php`, `.htaccess`) — envie para `public_html/cadastro_ecguaiba/`.
Lê da mesma biblioteca **Base de Dados** do RTC (`cadastro_drive_id` no `config.php`; se não
existir, usa `rtc_drive_id`):

| Arquivo (na biblioteca Base de Dados) | Papel |
|---|---|
| `01 Cadastro Empresas/dados_painel/painel_matriz.html` | a página, gerada pelo `painel_matriz.bat` (só leitura) |
| `01 Cadastro Empresas/dados_classificacao/classificacao_empresas.json` | **banco de dados do time** — o painel grava direto aqui |
| `01 Cadastro Empresas/dados_nfse/nfse_situacao.json` | situação das NFSe, lida a cada acesso (desde 28/09/2026) |
| `01 Cadastro Empresas/dados_sefaz/sefaz_situacao.json` | situação dos DF-e da Sefaz, lida a cada acesso (desde 28/09/2026) |

As colunas **NFSe - Situação** e **DFEs - Sefaz** são montadas no navegador a partir dos dois
JSON acima, que os agentes de download atualizam a cada rodada: não dependem de o painel ser
gerado de novo. O servidor guarda uma cópia de cada um em `ecg_portal/cadastro/` e só baixa de
novo quando o arquivo muda no SharePoint.

Trocar o time: "Alterar" na coluna Time → escolher → confirmar. A gravação usa `If-Match` (se
duas pessoas gravarem juntas, o servidor relê e aplica de novo, sem perder nenhuma). Os
projetos RTC - Empresas Simples Nacional e Regime Regular leem o mesmo JSON (o
`times_empresas.csv` foi descontinuado e está em `01 Cadastro Empresas\_legado`). Histórico:
`ecg_portal/cadastro/classificacao-log.jsonl` e o histórico de versões do arquivo no SharePoint.

---

## 4c. Painel Matriz Cadastral da Contabsul (`/Cadastro_Contabsul/`)

Desde 28/09/2026. Cópia do §4b para a Contabsul. Código em
`XML - ecguaiba_com_br\01 Cadastro de Empresas - Contabsul` (`_cadastro.php`, `index.php`,
`api.php`, `.htaccess`) — envie para `public_html/Cadastro_Contabsul/`. Dados do servidor em
`ecg_portal/cadastro_contabsul/`.

- **Acesso:** o mesmo login Microsoft 365 e o **mesmo grupo dedicado do `/RTC_Contabsul/`**
  (`grupo_contabsul_id` no `config.php`), conferido sem `eh_equipe()`.
- **SharePoint:** biblioteca **Base de Dados do site XMLContabsul** (`cadastro_contabsul_drive_id`
  no `config.php`; se não existir, usa `rtc_contabsul_drive_id`):
  `01 Cadastro Empresas/dados_painel/painel_matriz.html`,
  `01 Cadastro Empresas/dados_classificacao/classificacao_empresas.json` (time — o painel grava aqui) e
  `01 Cadastro Empresas/dados_nfse/nfse_situacao.json` (lido a cada acesso).
- **Sem coluna DFEs - Sefaz** (a Contabsul não tem XMLs do Webservice SefazRS). A NFSe casa pelo
  código do Domínio e, na falta, pelo CNPJ (layout de pastas por nome).
- Quem vai trocar times precisa poder **editar** a biblioteca Base de Dados do site XMLContabsul.

---

## 4d. Páginas iniciais `/ecguaiba/` e `/contabsul/` (desde 30/09/2026)

- Código: `servidor/public_html/ecguaiba/index.php`, `contabsul/index.php` (3 linhas cada) e, em
  `acesso/`, `_inicio.php` (monta a página), `_areas.php` (**lista única das áreas**) e `escolher.php`.
- **Área nova = uma linha em `ECG_AREAS`** (`_areas.php`): endereço, pastas aceitas no `volta`,
  escritório, grupo de acesso, nome e descrição. O cartão aparece sozinho para quem tem permissão,
  e o login passa a aceitar o endereço. Falta só a regra no `.htaccess` da raiz.
- Sem `volta`, o login leva: equipe Ecguaíba → `/ecguaiba/`; só grupo Contabsul → `/contabsul/`;
  os dois → `/acesso/escolher.php`; cliente → `/clientes/<cliente>/`. Nunca mais `/intranet/`.
- A `/intranet/` segue no ar só para quem tem o endereço. Não rodar mais o `publicar_intranet.bat`
  para ela (o `/clientes/` continua sendo publicado por ele). Mais adiante: 301 de `/intranet/`
  para `/ecguaiba/` e remoção da pasta.
- Backups desta mudança em `ecg_portal/`: `_nucleo.php.bak-20260930b`, `m365-retorno.php.bak-20260930`,
  `htaccess-raiz.bak-20260930b`.
- **Atenção ao enviar pelo cPanel:** a leitura de arquivo do Gerenciador (e a API
  `get_file_content`) às vezes "corrige" HTML dentro de PHP (inseriu `<meta charset>` depois de
  um `<header>`). O arquivo gravado fica certo; para conferir, baixe o arquivo em vez de abrir no editor.

---

## 4e. Resumo da importação de NFS-e (`/nfse_ecguaiba/`)

Desde 30/09/2026. Primeiro painel migrado com o guia `migrar-painel-local-para-servidor.md`
(padrão A; desde 01/10/2026 com `api.php` para a conferência mensal).

- Código: `XML - ecguaiba_com_br\Painel XML Portal NFSe` (`_nfse.php`, `index.php`, `api.php`, `.htaccess`)
  → `public_html/nfse_ecguaiba/`. Dados do servidor: `ecg_portal/nfse_ecguaiba/painel_cache.*`.
- Lê da biblioteca Base de Dados (`nfse_drive_id` ou, na falta, `rtc_drive_id`):
  `Painel XML Portal NFSe/dados_painel/resumo_importacao_nfse.html`, gravado pelo
  `gerar_painel.py` a cada rodada do agente de NFS-e (`XML - Agentes\NFSe`).
- Cartão na página `/ecguaiba/` (linha `nfse_ecguaiba` em `acesso/_areas.php`).
- Cópia de validação (desde 01/10/2026): `XML - web_validacao\Painel XML Portal NFSe - Ecguaiba\`
  (`PASTA_VALIDACAO` em `caminhos.py`) — regra em `web-validacao-paineis.md`.
- **Conferência mensal** (desde 01/10/2026): grupos 01 Sem dados / 02 Analisar / 03 Ajustar /
  04 Conferido + observação por empresa e mês, editáveis no portal. `api.php` grava direto em
  `Painel XML Portal NFSe/conferencia/AAAA-MM.json` na biblioteca Base de Dados (Graph, If-Match,
  como o Cadastro); cópia em `ecg_portal/nfse_ecguaiba/conf_AAAA-MM.json` e histórico em
  `conferencia-log.jsonl`. O arquivo do mês só nasce quando alguém gera a conferência no painel
  (`api.php` `acao=iniciar`; o painel pergunta na primeira vez que o mês é aberto). Detalhes no
  README do projeto Python.

---

## 4e2. Resumo da importação de NFS-e da Contabsul (`/nfse_contabsul/`)

Desde 30/09/2026. Cópia do `/nfse_ecguaiba/` (padrão A, só consulta, sem `api.php`).

- Código: `XML - ecguaiba_com_br\Painel XML Portal NFSe - Contabsul` (`_nfse.php`, `index.php`, `.htaccess`)
  → `public_html/nfse_contabsul/`. Dados do servidor: `ecg_portal/nfse_contabsul/painel_cache.*`.
- Lê da biblioteca Base de Dados do site XMLContabsul (`nfse_contabsul_drive_id` ou, na falta,
  `rtc_contabsul_drive_id`): `Painel XML Portal NFSe/dados_painel/resumo_importacao_nfse.html`,
  gravado pelo `gerar_painel.py` de `XML Contabsul - Projetos\Painel XML Portal NFSe` a cada rodada
  do agente `XML - Agentes\NFSe Contabsul` (tarefa "NFSe Contabsul - Rodar Tudo", de hora em hora, minuto :25).
- Acesso: grupo `grupo_contabsul_id` (não usa `eh_equipe()`). O PHP recusa painel sem
  `"escritorio": "contabsul"` — nunca entrega o da Ecguaíba.
- A cópia do portal sai sem caminhos locais e sem nome de arquivo de certificado
  (`painel_html._erro_para_web`, desde 30/09/2026).
- Cartão na página `/contabsul/` (linha `nfse_contabsul` em `acesso/_areas.php`).

---

## 4f. Serviços prestados — NFS-e x Domínio x Prefeitura (`/nfse_dominio_ecguaiba/`)

Desde 30/09/2026. Mesmo desenho do §4e (padrão A; desde 01/10/2026 com `api.php` para a
conferência mensal, igual à do §4e: grupos 01–04 + observação por mês de competência, em
`Painel XML Portal NFSe x Domínio/conferencia/AAAA-MM.json`).

- Código: `XML - ecguaiba_com_br\Painel XML Portal NFSe x Domínio` (`_nfse.php`, `index.php`, `api.php`, `.htaccess`) → `public_html/nfse_dominio_ecguaiba/`.
  Cache: `ecg_portal/nfse_dominio_ecguaiba/`.
- Lê `Painel XML Portal NFSe x Domínio/dados_painel/servicos_prestados_nfse_x_dominio.html`
  da biblioteca Base de Dados (~1–2 MB).
- **Não é automático:** o arquivo só muda quando alguém roda o projeto no escritório (o passo 1
  consulta o banco do Domínio). A data de geração aparece no cabeçalho do painel.
- A cópia do portal sai sem caminhos locais e sem nome de arquivo de certificado
  (`painel_html._erro_para_web`, desde 30/09/2026).

---

## 4f2. Serviços prestados da Contabsul (`/nfse_dominio_contabsul/`)

Desde 30/09/2026. Cópia do §4f para a Contabsul (padrão A, só consulta).

- Código: `XML - ecguaiba_com_br\Painel XML Portal NFSe x Domínio - Contabsul` (`_nfse.php`, `index.php`, `.htaccess`)
  → `public_html/nfse_dominio_contabsul/`. Cache: `ecg_portal/nfse_dominio_contabsul/painel_cache.*`.
- Lê da biblioteca Base de Dados do site XMLContabsul (`nfse_contabsul_drive_id` ou, na falta,
  `rtc_contabsul_drive_id`): `Painel XML Portal NFSe x Domínio/dados_painel/servicos_prestados_nfse_x_dominio.html`
  (~2,3 MB), gravado pelo `gerar_painel.py` de `XML Contabsul - Projetos\Painel XML Portal NFSe x Domínio`
  (banco do Domínio da Contabsul, DSN **Contabil2**; o `dominio.ini` fica em `XML Contabsul - Base de Dados\Banco de Dados Dominio`).
- **Automático desde 30/09/2026:** passo 4/4 do agente `XML - Agentes\NFSe Contabsul` (de hora em hora, minuto :25).
  Fora da rede do escritório o passo falha (só no log) e o portal continua mostrando a última cópia.
- Prefeitura: ainda sem CSV (`XML Contabsul - Base de Dados\Painel XML Portal NFSe x Domínio\Prefeitura`,
  padrão `AAAA MM - Prefeitura <Município>.csv`); a linha Prefeitura fica de fora até lá.
- Acesso: grupo `grupo_contabsul_id`; o PHP recusa painel sem `"escritorio": "contabsul"`.
- Cartão na página `/contabsul/` (linha `nfse_dominio_contabsul` em `acesso/_areas.php`).

---

## 5. Tarefas comuns

- **Publicar a intranet / páginas de cliente:** `publicar_intranet.bat`
  (`ecguaiba_com_br - Documentos\Intranet`) e enviar por FTP (Core FTP, AUTH TLS, binário):
  `C:\Intranet\intranet\` com a conta `intranet@ecguaiba.com.br` e
  `C:\Intranet\clientes\` com `clientes@ecguaiba.com.br`. Rode o `.bat` antes de cada envio.
- **Cliente novo:** criar a pasta com `dados.json` na base, rodar o `publicar_intranet.bat`,
  colar o `htpasswd-para-o-servidor.txt` no arquivo de senhas do servidor, enviar por FTP.
- **Colaborador novo (ou da Contabsul):** incluir no grupo `Ecguaíba - Equipe` no Entra. Para
  aplicar atualizações do RTC ele também precisa de leitura na biblioteca Base de Dados.
- **Dar acesso ao `/RTC_Contabsul/` especificamente:** incluir a pessoa no grupo extra descrito
  em §3 (`grupo_contabsul_id`) — e conferir se ela também está em `Ecguaíba - Equipe` (sem isso,
  o login é recusado antes mesmo de chegar no painel).
- **Ver quem acessou:** `ecg_portal/logs/acesso-AAAA-MM.log` (cPanel → Gerenciador de arquivos).
- **Desfazer uma atualização do RTC:** as versões anteriores ficam em `ecg_portal/rtc/historico/`.

---

## 6. Esta pasta

```
LEIA-ME.md
servidor/                      espelho do que está no servidor (sem segredos)
  public_html/acesso/          login, porteiro, Microsoft 365
  public_html/intranet/.htaccess, clientes/.htaccess   modelos (o script gera os reais)
  public_html/htaccess-raiz-trecho.txt
  ecg_portal/config.exemplo.php
```

**Conferida com o servidor em 30/09/2026** (arquivo a arquivo): `acesso/`, `.htaccess` de
`intranet/`, `clientes/` e `clientes/equipe/`, trecho do `.htaccess` da raiz e a estrutura do
`config.php` (em `config.exemplo.php`, sem o segredo). Guia para levar painéis locais ao site e
padrão de endereços em minúsculas: `migrar-painel-local-para-servidor.md`. Cópia local de
validação de cada painel publicado (`XML - web_validacao`): `web-validacao-paineis.md`.

Esta pasta guarda só o que é geral do portal (login, porteiro, Microsoft 365, intranet,
clientes) — não código nem dado de um painel específico. Dois conjuntos de arquivos que
já estiveram aqui saíram, ambos em 27/09/2026:

- `public_html/RTC_Ecguaiba/*.php` e `.htaccess` (o código PHP do painel RTC) agora ficam em
  `XML - Projetos\RTC - Ecguaiba - Analise Opcao Simples\servidor_php`, por serem
  específicos desse painel, não do portal em geral.
- Os arquivos gerados pelo `gerar_modelo_web.py` do projeto RTC (`painel_modelo.html`,
  `mensagens_padrao.json`) vivem só na pasta de staging do próprio projeto RTC
  (`XML - ecguaiba_com_br\RTC - Ecguaiba - Analise Opcao Simples` — ver LEIA-ME de lá).

Para enviar um arquivo ao servidor: cPanel → Gerenciador de arquivos → pasta
correspondente → Carregar (ou Editar e colar). Os arquivos que começam com `_` e os
`.htaccess` são internos e não abrem pela web.
