# Portal Ecguaíba — www.ecguaiba.com.br

Site de acesso restrito da equipe (HostGator, cPanel, PHP 7.4) que recebe os painéis antes gerados
só como HTML local. Esta pasta (`XML - Projetos\Portal Ecguaiba\`) guarda o código dos projetos que
publicam no portal, a cópia `servidor\` do que é geral (login, porteiro, M365) e estes documentos.

**O servidor é o que vale**: a cópia daqui serve para consulta, histórico e para refazer uma
instalação. Ao mudar algo no servidor, atualize a cópia.

Reorganizado em 03/10/2026 (versão anterior em `_legado\antes-contexto-20261003\`).

---

## 0. Documentos — leia este primeiro

Cada assunto tem um dono só; os outros apontam para ele.

| Arquivo | Assunto | Quando ler |
|---|---|---|
| `LEIA-ME.md` | Visão geral, servidor, Microsoft 365, tarefas comuns, pendências, pastas locais (§7) | Sempre |
| `01-enderecos-e-login.md` | Endereços oficiais, `volta`, `_areas.php`, teste obrigatório | Antes de criar, mover ou publicar uma área |
| `02-migrar-painel-para-o-portal.md` | Padrões A/B, receita de migração, regras de PHP/Graph/servidor | Ao levar um painel local para o site |
| `03-web-validacao-paineis.md` | Cópia local de conferência (`XML - web_validacao`) | Ao mexer no Python que gera painel |
| `04-reorganizacao-portal-2026-10.md` | Mudança para `/portal/` e `/portal_contabsul/`; projetos movidos para esta pasta | Enquanto a reorganização não terminar |
| `05-envio-ftp.md` | Contas FTP, `ftp-ecguaiba.sh`, cuidados de envio | Ao enviar qualquer arquivo ao servidor |
| `06-paineis.md` | Ficha de cada painel: fontes, staging, cache, automação, particularidades | Ao mexer num painel específico |

O README/LEIA-ME de cada projeto (subpastas) cobre como rodar o Python daquele painel.

---

## 1. O site (03/10/2026)

| Raiz | Público | Login | Situação |
|---|---|---|---|
| `/` | Todos | — | Site institucional (WordPress) |
| `/portal/` | Equipe Ecguaíba | Microsoft 365, grupo "Ecguaíba - Equipe" | Página inicial no ar desde 03/10 (`/ecguaiba/` → 301). Painéis ainda nos endereços antigos |
| `/portal_contabsul/` | Equipe Contabsul | Microsoft 365, `grupo_contabsul_id` | Idem (`/contabsul/` → 301) |
| `/clientes/<cliente>/` | Clientes | usuário e senha **ou** convidado M365 do grupo do cliente | `grupokalata` (M365 desde 01/10), `madevan` |
| `/acesso/` | — | Login, Microsoft, porteiro, `escolher.php` | Comum a todas as raízes |
| `/intranet/`, `/clientes/equipe/` | — | — | Adormecidos: no ar, sem cartão e fora do destino padrão |

Lista de painéis e endereços: `01-enderecos-e-login.md` §1. Detalhe de cada painel: `06-paineis.md`.

**Logins**

- **Equipe:** "Entrar com a Microsoft", contas @ecguaiba.com.br do grupo **`Ecguaíba - Equipe`** (inclui a Contabsul). Para dar ou tirar acesso, mexa só no grupo.
- **Contabsul:** além disso, grupo dedicado `grupo_contabsul_id` (conferido sem `eh_equipe()`).
- **Clientes:** usuário e senha do arquivo `/home1/ecgua152/.htpasswds/public_html/clientes/passwd` (cada cliente só abre a própria pasta; senhas em claro em `ecguaiba_com_br - Documentos\Intranet\credenciais.json`) **ou** convidado Microsoft 365 (B2B) do grupo do cliente — lista `ECG_CLIENTES` em `acesso/_areas.php` + `'clientes_m365'` no `config.php`.
- O antigo usuário **`equipe`** (emergência) está **desativado**.

---

## 2. Onde fica cada coisa no servidor

```
/home1/ecgua152/
├── public_html/
│   ├── .htaccess          trecho "Ecguaiba - areas estaticas fora do WordPress" (servidor\public_html\htaccess-raiz-trecho.txt)
│   ├── acesso/            login, porteiro, M365, _areas.php            (servidor\public_html\acesso)
│   ├── portal/            página inicial Ecguaíba (index.php + .htaccess)  (conta FTP portal)
│   ├── portal_contabsul/  página inicial Contabsul                     (conta FTP portal_contabsul)
│   ├── ecguaiba/, contabsul/   antigas páginas iniciais (301 para as novas)
│   ├── <painéis>/         cadastro_ecguaiba/, RTC_Ecguaiba/, nfse_*/, gestao/... (ver 06)
│   ├── intranet/          .htaccess → porteiro (gerado pelo publicar_intranet.py)
│   └── clientes/          .htaccess → porteiro, em cada subpasta
└── ecg_portal/            FORA do site
    ├── config.php         configuração + segredo da Microsoft (0600); modelo: servidor\ecg_portal\config.exemplo.php
    ├── sessoes/, tentativas.json (5 erros = 15 min de bloqueio)
    ├── logs/acesso-AAAA-MM.log   logins, recusas, prévias, atualizações aplicadas
    ├── bak-AAAAMMDD-<motivo>/    backups de arquivos gerais
    └── <area>/            dados e cache de cada painel (nomes NÃO mudam na reorganização; ver 06)
```

**Porteiro (`acesso/porteiro.php`):** o `.htaccess` de `/intranet` e `/clientes` manda todo pedido
para o porteiro, que confere quem está logado e se pode abrir aquele caminho. Ocultos e `.php`
nunca são entregues. Páginas HTML recebem o selo "nome · Sair".

---

## 3. Microsoft 365 (Entra ID)

- Aplicativo **Portal Ecguaíba**, ID `ffe9ba60-0c8b-487d-a1ed-392036a0ef9b`, locatário `8e3e5c0f-d523-4528-a088-a74b79aa66cb`.
- Redirecionamento: `https://www.ecguaiba.com.br/acesso/m365-retorno.php`.
- Permissões delegadas (consentidas): `User.Read`, `Files.ReadWrite.All` (desde 28/09/2026) e `offline_access`. Os painéis leem o SharePoint **em nome de quem está logado**; quem grava precisa poder editar a biblioteca.
- Grupo **Ecguaíba - Equipe** (`ea38a5f4-8cea-4062-ad19-a0b9d0aeba0c`), atribuído ao aplicativo, "Atribuição necessária = Sim". `eh_equipe()` confere o grupo de verdade (claim `groups`, guardada na sessão no login).
- **Grupo Contabsul** (`grupo_contabsul_id`): conferido com `eh_m365()` + `pertence_grupo()`. Quem está só nele não loga, a não ser que o grupo também esteja atribuído ao aplicativo (ver `RTC - Contabsul - ...\servidor_config\config-contabsul.trecho.txt`).
- **Grupos de cliente** (ex.: `Clientes - Grupo Kalata`): convidados B2B, atribuídos ao aplicativo; Object Id em `'clientes_m365'`.
- **Segredo do aplicativo vence em 26/09/2028** (24 meses). Antes: Entra → Registros de aplicativo → Portal Ecguaíba → Certificados e segredos → Novo segredo → colar o **Valor** em `ecg_portal/config.php` (`client_secret`). Sem isso o botão da Microsoft some.
- Nunca guarde segredo nesta pasta (ela sincroniza com o SharePoint).

---

## 4. Tarefas comuns

- **Colaborador novo (ou da Contabsul):** incluir em `Ecguaíba - Equipe`; para a Contabsul, também em `grupo_contabsul_id`. Para os painéis, ele precisa de leitura na biblioteca Base de Dados (edição, se for trocar time no Cadastro ou aplicar RTC).
- **Cliente novo com login Microsoft:** linha em `ECG_CLIENTES` + grupo no Entra + Object Id em `clientes_m365` (roteiro: `Contabil - Painel Grupo Kalata\LEIA-ME.md` e `ENVIAR.md` do staging).
- **Cliente novo só com senha:** pasta com `dados.json` na base, `publicar_intranet.bat`, colar o `htpasswd-para-o-servidor.txt` no arquivo de senhas do servidor, enviar a pasta pela conta `clientes` (`05`).
- **Painel novo:** `02` + `01` §6.
- **Enviar arquivo:** `05` (FTP). Arquivos gerais (`acesso/`, `.htaccess` da raiz, `config.php`): cPanel, com backup em `ecg_portal/bak-.../`.
- **Ver quem acessou:** `ecg_portal/logs/acesso-AAAA-MM.log`.
- **Desfazer atualização do RTC:** `ecg_portal/rtc/historico/`.

---

## 5. Regras que valem para tudo

1. O servidor é o que vale: antes de editar `acesso/` ou o `.htaccess` da raiz, traga a versão do servidor.
2. Endereço oficial em minúsculas e com barra final; o login volta sempre para a área de origem (`01`).
3. Painel novo nasce dentro de `/portal/` ou `/portal_contabsul/` (`04`).
4. Publicação: simular → backup → enviar → teste em janela anônima de **todas** as áreas (`01` §5).
5. Não mudar nomes nem parâmetros dos scripts chamados pelos agentes (`XML - Agentes\...`) e pelos programas de download.
6. Caminhos no Python: raiz achada por `_achar_raiz()`; nunca `parents[n]` (`04` §3).
7. PHP 7.4: sem `match`, `str_contains`, `str_starts_with`, argumentos nomeados, `?->`.
8. Nada de senha no OneDrive.

---

## 6. Pendências abertas (03/10/2026)

- [ ] Teste **logado** (Microsoft) da preparação do `/portal/` (`04` §0).
- [ ] Trazer do servidor para `servidor\`: `_areas.php` (a cópia local está sem as linhas de Gestão e ainda com `/ecguaiba/` em `ECG_INICIOS`), `.htaccess` da raiz e as pastas `portal/` e `portal_contabsul/`.
- [ ] Piloto: Cadastro Ecguaíba → `/portal/cadastro/`.
- [ ] Rodar de novo `RTC - Ecguaiba - Analise Opcao Simples\agendar_exportacao_dominio.bat` (a tarefa agendada guarda o caminho antigo) e conferir outras tarefas (`04` §3).
- [ ] Cópia de validação: NFSe Contabsul e NFSe x Domínio Contabsul (`03` §2).
- [ ] `Segredo.txt` nesta pasta — mover para fora do OneDrive.
- [ ] Pasta GRUPO KALATA no SharePoint tem um link "qualquer pessoa pode editar" — avaliar se continua.
- [ ] Limpar sobras do `public_html` (com backup): `Cadastro_Ecguaiba/`, `sitenovo/`, `novosite2/`, `ecguaiba.com.br/`.

---

## 7. Pastas na máquina local

Tudo fica em `C:\Onedrive\Ecguaiba Contabilidade\`, sincronizado com o SharePoint. Um painel do
portal usa **quatro pastas**, sempre com o mesmo nome de projeto (regra completa em `02` §1):

| Pasta | Papel | O que tem | Quem grava |
|---|---|---|---|
| `XML - Projetos\Portal Ecguaiba\` | **Código** | `.py`/`.bat` de cada painel, estes documentos, `servidor\` | Você (edição de código) |
| `XML - Base de Dados\` | **Dados** + HTML que o site lê | Biblioteca **Base de Dados** do site SharePoint **XML** (`rtc_drive_id`). O PHP do portal lê daqui via Graph | Os scripts Python e o próprio portal (gravações da equipe) |
| `XML - web_validacao\` | **Conferência local** | Cópia do HTML de cada painel, gerada na mesma rodada que a cópia web | Só os scripts; **somente leitura** (`03`) |
| `XML - ecguaiba_com_br\` | **Staging do site** | PHP de cada painel + ferramentas de FTP, pronto para enviar ao `public_html` | Você; depois envio por FTP (`05`) |

Fluxo: **código** gera → **dados** (SharePoint, o site lê) + **validação** (você confere);
o **staging** só muda quando muda o PHP do painel.

**`XML - Base de Dados\`** — além das pastas de painel, guarda bases brutas usadas por vários projetos:

- Por painel (cada uma com `dados_painel\` = HTML publicado, e às vezes `conferencia\`):
  `01 Cadastro Empresas` (atenção: sem "de", ao contrário do projeto; `dados_rfb`, `dados_dominio`, `dados_classificacao` etc.),
  `Painel XML Portal NFSe`, `Painel XML Portal NFSe x Domínio`, `Contabil - Gestão`,
  `Contabil - Gestão - Contabsul` (Gestão Contabsul lê daqui, ver `06`), `DP Pessoal - Gestão`,
  `Contabil - Painel Grupo Kalata` (`cliente.json`), `RTC - *`.
- Fontes externas: `API Digisac`, `API Office 365`, `API Omie Gclick`, `Banco de Dados Dominio`,
  `Dados Abertos RFB`.
- Análises e projetos fora do portal: `Análise *`, `Cobertura XML`, `EFD ICMS IPI`, `Matriz Tributaria`,
  `Projeto Simples Nacional`, `Painel XML Web Service Sefaz*`, `Regras de Importação e Acumulador` etc.
- **Contabsul:** a base equivalente é `XML Contabsul - Base de Dados` (site **XMLContabsul**, `rtc_contabsul_drive_id`).

**`XML - web_validacao\`** — uma subpasta por painel, nomeada como o projeto (com ` - Ecguaiba` /
` - Contabsul` quando o mesmo projeto atende os dois). Arquivos `*_DEMO.html` são painéis de
demonstração. Pares projeto → pasta e pendências: `03` §2. Não edite à mão: a próxima rodada sobrescreve.

**`XML - ecguaiba_com_br\`** — uma subpasta por painel com o PHP (`_<area>.php`, `index.php`,
`api.php`, `.htaccess`) ou `public_html\` + `ENVIAR.md` (Gestão, DP, Grupo Kalata). Especiais:

- `FTP\` — `ftp-ecguaiba.sh`, `contas.conf`, `LEIA-ME.md` (`05`).
- `RTC - * - Analise Opcao Simples\` — `painel_modelo.html` e `mensagens_padrao.json`, gravados pelo
  `gerar_modelo_web.py --publicar` (vão para `ecg_portal/rtc/`, não para `public_html`; `06`).
- `RTC_Ecguaiba\` — só `dominio.ini`; sobra a conferir.

**`XML - Projetos\Portal Ecguaiba\`** — esta pasta (detalhe no §8).

Outras pastas `XML - *` ao lado (`Agentes`, `Ecguaiba Painel Fiscal`, `Documentos Fiscais`…) não
são do portal; `XML - Agentes\` chama os scripts daqui (regra 5 do §5) e `XML - Ecguaiba Painel Fiscal`
guarda os painéis só locais.

---

## 8. Esta pasta

```
LEIA-ME.md, 01-…06-*.md       documentos do portal (acima)
servidor\                     espelho do que é geral no servidor (sem segredos)
  public_html\acesso\         login, porteiro, M365, _areas.php, _inicio.php, escolher.php
  public_html\ecguaiba\, contabsul\   páginas iniciais antigas (as novas: /portal/, /portal_contabsul/)
  public_html\intranet\.htaccess, clientes\.htaccess   modelos (o script gera os reais)
  public_html\htaccess-raiz-trecho.txt
  ecg_portal\config.exemplo.php
<projeto>\                    código Python de cada painel (lista em 04 §3)
_legado\                      versões anteriores dos documentos
```

Código PHP específico de um painel **não** fica em `servidor\`: fica no staging
`XML - ecguaiba_com_br\<projeto>\` (o RTC, em `<projeto>\servidor_php`).
