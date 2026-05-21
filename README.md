# Análise de Sistemas — ERP Moderno (Backend)
> Sistema fiscal e gerencial voltado para o mercado brasileiro  
> Escopo: emissão de NF-e/NFS-e, gestão de estoque, contas a pagar/receber, geração de boletos, relatórios contábeis e documentos fiscais.

---

## Sumário

1. [Visão Geral da Arquitetura](#1-visão-geral-da-arquitetura)
2. [Stack Tecnológica](#2-stack-tecnológica)
3. [Módulos do Sistema](#3-módulos-do-sistema)
4. [Modelagem de Dados (Macro)](#4-modelagem-de-dados-macro)
5. [Fluxos Críticos](#5-fluxos-críticos)
6. [Segurança e Autenticação](#6-segurança-e-autenticação)
7. [Comunicação em Tempo Real](#7-comunicação-em-tempo-real)
8. [Estratégia de Filas e Jobs](#8-estratégia-de-filas-e-jobs)
9. [Cache e Performance](#9-cache-e-performance)
10. [Integrações Externas](#10-integrações-externas)
11. [Observabilidade e Monitoramento](#11-observabilidade-e-monitoramento)
12. [Considerações de Escalabilidade](#12-considerações-de-escalabilidade)

---

## 1. Visão Geral da Arquitetura

O sistema adota uma arquitetura **monolítica modular** (Modular Monolith), organizada por domínios de negócio dentro do Laravel. Essa abordagem equilibra a simplicidade operacional de um monólito com a separação de responsabilidades próxima de microsserviços — adequada para ERPs, onde há forte coesão entre módulos (ex.: estoque ↔ fiscal ↔ financeiro).

```
┌─────────────────────────────────────────────────────────────────┐
│                        API RESTful (Laravel)                     │
│                                                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐   │
│  │  Fiscal  │ │ Estoque  │ │Financeiro│ │   Contabilidade  │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘   │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────────────────┐    │
│  │ Relatórios│ │ Usuários │ │       Documentos/Boletos     │    │
│  └──────────┘ └──────────┘ └──────────────────────────────┘    │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │               Camada de Infraestrutura                    │   │
│  │   Queue Worker │ Sanctum/Fortify │ Spatie │ Reverb       │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
         │               │               │
    PostgreSQL          Redis       Serviços Externos
                                (SEFAZ, bancos, IBGE)
```

### Padrões Adotados

- **Repository Pattern** para abstração da camada de dados
- **Service Layer** para regras de negócio complexas (especialmente fiscal)
- **DTO (Data Transfer Objects)** para transferência entre camadas
- **Events & Listeners** para desacoplamento entre módulos
- **Jobs & Queues** para operações assíncronas (emissão de NF, boletos, relatórios pesados)
- **Policies** para controle de acesso granular por recurso

---

## 2. Stack Tecnológica

### 2.1 PHP 8.4

Linguagem principal do backend.

**Recursos relevantes para o ERP:**
- Property hooks (simplifica getters/setters em entidades de domínio)
- Asymmetric visibility (`public private(set)`) para modelos imutáveis
- Typed class constants (maior segurança em enums de status fiscal/financeiro)
- `array_find()` nativo (útil em coleções de itens de NF)
- JIT melhorado (ganho de performance em relatórios e cálculos de impostos)

> **💡 Sugestão alternativa:** Não há alternativa recomendada — PHP 8.4 é a versão mais atual e estável. Manter sempre na minor version mais recente do 8.x para receber patches de segurança.

> **⚠️ Observação:** Verificar compatibilidade de todas as dependências (especialmente pacotes de NFe/NFS-e) com PHP 8.4 antes de subir para produção. Pacotes fiscais costumam ter atraso de atualização.

---

### 2.2 Laravel 13

Framework principal. Suporte LTS, ecossistema maduro para ERPs.

**Recursos mais relevantes para o projeto:**
- **Eloquent ORM** com `withCasts`, enums nativos e lazy collections (para relatórios volumosos)
- **Form Requests** para validação granular (itens de NF, cálculo de ICMS/PIS/COFINS)
- **Resource Classes** para serialização consistente de respostas
- **Policies + Gates** para controle de acesso por empresa/filial
- **Observers** para auditoria automática de entidades (quem alterou, quando)
- **Telescope** (dev) e **Pulse** (produção) para observabilidade
- **Multi-tenancy** via `stancl/tenancy` ou escopo global (`withGlobalScope`) — essencial se o ERP servir múltiplas empresas

> **💡 Sugestão alternativa:** Para times com experiência em arquiteturas orientadas a domínio, considerar a adoção de um pacote como `lorisleiva/laravel-actions` para organizar regras de negócio em Actions reutilizáveis, reduzindo a obesidade dos Services.

> **⚠️ Observação:** Estruturar os módulos como `app/Modules/{NomeDomínio}` com seus próprios `Controllers`, `Services`, `Repositories` e `Models` desde o início. Refatorar um ERP não modularizado é extremamente custoso.

---

### 2.3 PostgreSQL

Banco de dados relacional principal.

**Por que PostgreSQL para ERP fiscal:**
- **JSONB** para armazenar XMLs de NF-e processados e retornos da SEFAZ
- **Particionamento de tabelas** por período (movimentações, lançamentos contábeis)
- **Window functions** para relatórios contábeis complexos (DRE, balanço)
- **CTEs recursivas** para plano de contas hierárquico
- **Full-text search** nativo para busca de produtos/clientes
- **Transações ACID** robustas — crítico para operações financeiras
- **Materializada Views** para relatórios que precisam de performance sem ETL

**Configurações recomendadas para ERP:**
- `default_transaction_isolation = 'read committed'` (padrão, suficiente para maioria)
- `statement_timeout` configurado por tipo de operação
- Índices parciais em colunas de status (ex.: `WHERE status = 'pendente'`)
- Índice GIN em colunas JSONB de dados fiscais

> **💡 Sugestão alternativa:** Para empresas que já têm DBAs familiarizados com MySQL/MariaDB, o MySQL 8.4 pode ser usado — mas perde-se recursos valiosos como JSONB, CTEs complexas e particionamento nativo. Não recomendado para volume fiscal elevado.

> **⚠️ Observação:** Criar uma política de retenção e arquivamento desde o início. Documentos fiscais têm obrigação legal de guarda de 5 anos. Tabelas de NF-e e lançamentos crescem rapidamente — planejar particionamento por `competencia` já na modelagem inicial.

---

### 2.4 Redis

Cache, sessões, filas e pub/sub.

**Usos no ERP:**
- Cache de tabelas de impostos (IBPT, NCM, CFOP) — raramente alteradas
- Cache de configurações fiscais por empresa/filial
- Rate limiting em endpoints de emissão (evitar duplicidade de NF)
- Sessões de usuário
- Locks distribuídos (`Cache::lock()`) para operações de estoque (evitar race condition em baixa de estoque)
- Backend das filas Laravel (via `QUEUE_CONNECTION=redis`)
- Canal de pub/sub para o Laravel Reverb

> **💡 Sugestão alternativa:** **Valkey** (fork open-source do Redis, mantido pela Linux Foundation após mudança de licença do Redis 7.4+) é um substituto drop-in compatível com a driver do Laravel. Recomendado para novos projetos que queiram evitar dependência de licença proprietária.

> **⚠️ Observação:** Configurar persistência RDB + AOF no Redis de produção. A perda da fila de emissão de NF-e em caso de crash sem persistência pode gerar retrabalho e problemas fiscais (NFs em processamento que precisam ser canceladas ou reemitidas manualmente).

---

### 2.5 Laravel Queues

Sistema de filas assíncrono — crítico para um ERP fiscal.

**Filas recomendadas (por prioridade e domínio):**

| Fila | Prioridade | Responsabilidade |
|---|---|---|
| `fiscal-critical` | Alta | Emissão e cancelamento de NF-e/NFS-e, retorno SEFAZ |
| `financial` | Alta | Geração de boletos, retorno de pagamentos |
| `stock` | Média | Reserva e baixa de estoque |
| `reports` | Baixa | Geração de relatórios pesados (SPED, DRE) |
| `notifications` | Baixa | E-mails, webhooks, notificações internas |
| `default` | Normal | Demais operações assíncronas |

**Jobs críticos identificados:**
- `EmitirNFeJob` — comunicação com SEFAZ (retry com backoff exponencial)
- `CancelarNFeJob` — cancelamento dentro do prazo legal
- `GerarBoletoJob` — integração com bancos (Bradesco, Itaú, BB via API)
- `RetornarStatusNFeJob` — consulta de lotes pendentes
- `GerarSpedFiscalJob` — geração de arquivo SPED (pesado)
- `EnviarEmailNFeJob` — envio de NF-e por e-mail ao destinatário
- `AtualizarCotacaoMoedaJob` — câmbio para operações com importação

> **💡 Sugestão alternativa:** **Laravel Horizon** é fortemente recomendado como complemento obrigatório — não é uma substituição, mas uma adição. Ele fornece dashboard visual para monitoramento das filas, métricas de throughput e configuração de workers por fila. Para ERPs com alto volume de NF-e, é indispensável.

> **⚠️ Observação:** Configurar `failed_jobs` com alertas. Uma NF-e que falha silenciosamente pode gerar passivo fiscal. Implementar `JobFailed` listener global que notifique o time de operações imediatamente via Slack/e-mail.

---

### 2.6 Laravel Sanctum + Fortify

Autenticação e gerenciamento de sessão/tokens.

**Estratégia para ERP:**
- **Sanctum** para emissão de tokens de API (integrações externas, app mobile futuro)
- **Fortify** para o fluxo de autenticação web (login, logout, reset de senha, 2FA)
- Tokens com escopos personalizados por módulo (`fiscal:read`, `fiscal:write`, `financial:*`)
- Expiração de tokens configurável por tipo de cliente (integração vs. usuário humano)
- **2FA obrigatório** para usuários com permissões fiscais e financeiras (TOTP via Google Authenticator)

> **💡 Sugestão alternativa:** Para ERPs que precisarão de SSO corporativo (integração com Active Directory, Okta, Google Workspace), considerar **Laravel Passport** em vez de Sanctum para o fluxo OAuth2 completo, ou adicionar **socialite** para login federado.

> **⚠️ Observação:** Implementar log de acesso com IP, user-agent e timestamp em todos os logins — pode ser exigido em auditorias fiscais. Também implementar bloqueio por tentativas excessivas (já disponível no Fortify via `RateLimiter`).

---

### 2.7 Spatie Permission

Controle de acesso baseado em papéis e permissões (RBAC).

**Estrutura de papéis sugerida para ERP:**

| Role | Escopo |
|---|---|
| `super-admin` | Acesso total ao sistema |
| `admin-empresa` | Acesso total à empresa (multi-tenant) |
| `contador` | Módulos fiscal, contabilidade, relatórios |
| `financeiro` | Contas a pagar/receber, boletos, fluxo de caixa |
| `estoque` | Movimentações, produtos, fornecedores |
| `vendas` | Pedidos, NF-e de saída, clientes |
| `compras` | Pedidos de compra, NF-e de entrada |
| `readonly` | Consulta sem alteração |

**Permissões granulares (exemplos):**
- `nfe.emitir`, `nfe.cancelar`, `nfe.visualizar`
- `boleto.gerar`, `boleto.baixar`
- `estoque.ajustar`, `estoque.visualizar`
- `relatorio.sped`, `relatorio.dre`, `relatorio.balancete`

> **💡 Sugestão alternativa:** O Spatie Permission é a escolha padrão do ecossistema Laravel e não há razão para substituí-lo. Para casos de ACL muito complexa (permissões por registro específico, ex.: "usuário X só pode ver a filial Y"), complementar com **Policies** do Laravel — elas trabalham em harmonia com o Spatie.

> **⚠️ Observação:** Cachear as permissões (o Spatie já faz via `permission.cache` config) e configurar o TTL adequadamente. Em sistemas multi-tenant, garantir que o cache seja invalidado por tenant para evitar vazamento de permissões entre empresas.

---

### 2.8 Laravel Reverb

Servidor WebSocket nativo do Laravel para comunicação em tempo real.

**Casos de uso no ERP:**
- Notificação em tempo real do status de emissão de NF-e (processando → autorizada / rejeitada)
- Atualização de dashboard financeiro (saldo, inadimplência)
- Alertas de estoque mínimo atingido
- Notificações de pagamento confirmado (retorno de boleto)
- Progresso de geração de relatórios pesados (ex.: SPED com barra de progresso)
- Avisos de acesso simultâneo ao mesmo registro (evitar edição conflitante)

> **💡 Sugestão alternativa:** **Pusher** (serviço gerenciado) pode ser usado como substituto se a equipe quiser eliminar a gestão de infraestrutura do Reverb. O Laravel Broadcasting é compatível com ambos sem alteração de código. Para ambientes on-premise ou com restrição de dados (LGPD), o Reverb self-hosted é o mais adequado.

> **⚠️ Observação:** O Reverb é relativamente novo no ecossistema (lançado em 2024). Em produção com alto volume de conexões simultâneas, monitorar consumo de memória e configurar adequadamente o número de workers. Considerar NGINX como proxy reverso na frente do Reverb para TLS e balanceamento.

---

## 3. Módulos do Sistema

### 3.1 Módulo Fiscal

**Responsabilidades:**
- Emissão de NF-e (modelo 55), NFS-e e NFC-e (modelo 65)
- Cancelamento e carta de correção (CC-e)
- Inutilização de numeração
- Consulta de status na SEFAZ
- Cálculo automático de impostos: ICMS, IPI, PIS, COFINS, ISS, DIFAL
- Gestão de CFOP, NCM, CST/CSOSN
- Download e armazenamento de XMLs autorizados

**Pacotes sugeridos:**
- `nfephp-org/sped-nfe` — biblioteca PHP para NF-e
- `nfephp-org/sped-nfse` — para NFS-e (municipal)
- `nfephp-org/sped-ibpt` — tabela IBPT para cálculo de impostos aproximados (obrigatório na NF)

### 3.2 Módulo Financeiro

**Responsabilidades:**
- Contas a pagar (lançamentos, vencimentos, baixas manuais e automáticas)
- Contas a receber (títulos, boletos, cobranças)
- Fluxo de caixa (projetado e realizado)
- Conciliação bancária (via OFX/CNAB)
- Geração de boletos bancários (CNAB 240/400)
- Integração com APIs bancárias (PIX, boleto registrado)
- Controle de centros de custo

**Pacotes sugeridos:**
- `eduardokum/laravel-boleto` — geração de boletos para os principais bancos brasileiros
- `sibs-pt/laravel-cnab` ou similar para leitura de retornos bancários CNAB

### 3.3 Módulo de Estoque

**Responsabilidades:**
- Cadastro de produtos com NCM, unidade de medida, CEST
- Controle de lotes e validade
- Movimentações de entrada/saída (vinculadas a NF-e)
- Estoque mínimo e máximo com alertas
- Inventário físico
- Múltiplos depósitos/localizações

### 3.4 Módulo Contábil

**Responsabilidades:**
- Plano de contas (estrutura hierárquica)
- Lançamentos contábeis automáticos (por evento: venda, compra, pagamento)
- Geração do SPED Contábil (ECD)
- Geração do SPED Fiscal (EFD ICMS/IPI e EFD-Contribuições)
- Geração da ECF (Escrituração Contábil Fiscal)
- DRE, Balanço Patrimonial e DLPA

### 3.5 Módulo de Relatórios

**Responsabilidades:**
- Relatórios gerenciais (vendas, compras, estoque, financeiro)
- Relatórios fiscais obrigatórios
- Exportação em PDF, Excel e CSV
- Relatórios agendados (envio automático por e-mail)
- Dashboard com KPIs em tempo real

**Pacotes sugeridos:**
- `barryvdh/laravel-dompdf` ou `spatie/laravel-pdf` — geração de PDFs
- `maatwebsite/excel` — exportação para Excel

---

## 4. Modelagem de Dados (Macro)

```
empresas
├── filiais
├── usuarios (N:M via permissões)
├── plano_de_contas (árvore hierárquica)
│
├── produtos
│   ├── estoque_movimentacoes
│   ├── lotes
│   └── preco_historico
│
├── clientes / fornecedores (entidades)
│
├── notas_fiscais
│   ├── nfe_itens
│   ├── nfe_impostos
│   ├── nfe_eventos (cancelamentos, CCe)
│   └── nfe_xml (armazenamento JSONB/filesystem)
│
├── pedidos_venda / pedidos_compra
│
├── contas_pagar
│   ├── parcelas
│   └── baixas
│
├── contas_receber
│   ├── parcelas
│   ├── boletos
│   └── baixas
│
├── lancamentos_contabeis
│   ├── partidas (débito/crédito)
│   └── centro_custo
│
└── relatorios_agendados
```

---

## 5. Fluxos Críticos

### 5.1 Emissão de NF-e

```
Request (POST /api/fiscal/nfe)
    │
    ├─ FormRequest (validação dos dados fiscais)
    ├─ NFeService::calcularImpostos()
    ├─ NFeService::gerarXml()
    ├─ NFeService::assinarXml()  (certificado A1/A3)
    │
    └─ EmitirNFeJob (dispatch para fila fiscal-critical)
            │
            ├─ SEFAZ::enviarLote()
            ├─ SEFAZ::consultarRecibo() [com retry/backoff]
            │
            ├─ [Autorizada]
            │   ├─ Salvar XML autorizado (S3/filesystem)
            │   ├─ Atualizar status no banco
            │   ├─ LancamentoContabilJob (dispatch)
            │   ├─ BaixaEstoqueJob (dispatch)
            │   ├─ EnviarEmailNFeJob (dispatch)
            │   └─ Broadcast: NFeAutorizada (Reverb)
            │
            └─ [Rejeitada]
                ├─ Salvar motivo da rejeição
                ├─ Broadcast: NFeRejeitada com descrição do erro
                └─ Notificar usuário emissor
```

### 5.2 Geração de Boleto

```
Request (POST /api/financeiro/boletos)
    │
    ├─ Validar título vinculado (conta a receber)
    ├─ GerarBoletoJob (dispatch para fila financial)
    │       │
    │       ├─ API Banco (registro do boleto)
    │       ├─ Gerar PDF do boleto
    │       ├─ Salvar linha digitável e código de barras
    │       └─ Broadcast: BoletoGerado
    │
    └─ Retorno imediato ao cliente com status "em processamento"
```

---

## 6. Segurança e Autenticação

- **Autenticação:** Sanctum (tokens API) + Fortify (web/SPA)
- **2FA obrigatório** para roles com acesso fiscal e financeiro
- **Certificado Digital A1** armazenado criptografado (AES-256) no banco ou cofre (ex.: HashiCorp Vault)
- **HTTPS obrigatório** em todos os endpoints (TLS 1.2+)
- **Rate limiting** por usuário e por endpoint sensível
- **Auditoria completa** com `spatie/laravel-activitylog` — todos os CRUDs registrados
- **Soft deletes** em todas as entidades críticas (nunca deletar dados fiscais)
- **Criptografia de dados sensíveis** (ex.: dados bancários) com `casts` de criptografia do Laravel
- **CORS** configurado estritamente para domínios do frontend

---

## 7. Comunicação em Tempo Real

O **Laravel Reverb** gerencia os canais de WebSocket:

| Canal | Tipo | Evento |
|---|---|---|
| `empresa.{id}.fiscal` | Private | NFeAutorizada, NFeRejeitada, NFeStatus |
| `empresa.{id}.financeiro` | Private | BoletoGerado, PagamentoConfirmado |
| `empresa.{id}.estoque` | Private | EstoqueMinimoAtingido |
| `relatorio.{jobId}` | Private | RelatorioProgresso, RelatorioConcludo |
| `usuario.{id}` | Private | NotificacaoGeral |

---

## 8. Estratégia de Filas e Jobs

### Workers Recomendados por Ambiente

**Produção (mínimo):**
```bash
# Worker dedicado para fiscal (alta prioridade, timeout maior para SEFAZ)
php artisan queue:work redis --queue=fiscal-critical --timeout=120 --tries=3

# Worker financeiro
php artisan queue:work redis --queue=financial --timeout=60 --tries=3

# Workers gerais
php artisan queue:work redis --queue=stock,notifications,default --timeout=30
```

### Retry e Backoff para Jobs Fiscais

```php
// Exemplo de configuração em EmitirNFeJob
public $tries = 3;
public $backoff = [30, 120, 300]; // segundos (30s, 2min, 5min)
public $timeout = 120;

public function failed(Throwable $e): void
{
    // Notificar equipe de operações imediatamente
    // Reverter status da NF para "erro_emissao"
}
```

---

## 9. Cache e Performance

### Estratégias de Cache (Redis)

| Dado | TTL | Justificativa |
|---|---|---|
| Tabela IBPT/NCM | 24h | Atualizada mensalmente |
| Config. fiscal da empresa | 1h | Altera raramente |
| Permissões de usuário | 5min | Spatie cache |
| Cotação de moedas | 1h | API do Banco Central |
| Alíquotas estaduais (ICMS) | 12h | Alteram raramente |
| Resultados de relatórios | 10min | Relatórios idênticos repetidos |

### Eager Loading

Sempre usar `with()` em queries que envolvam itens de NF, parcelas e lançamentos para evitar N+1 — crítico em relatórios e listagens volumosas.

### Lazy Collections

Usar `LazyCollection` e `cursor()` para exportações e relatórios com grandes volumes (ex.: SPED com anos de lançamentos).

---

## 10. Integrações Externas

| Integração | Finalidade | Comunicação |
|---|---|---|
| SEFAZ Estadual | Emissão/consulta NF-e | SOAP/XML (via sped-nfe) |
| SEFAZ Nacional | NF-e modelo 55, contingência | SOAP/XML |
| Prefeituras | NFS-e municipal | REST ou SOAP (varia por município) |
| APIs Bancárias | Boletos, PIX, retorno CNAB | REST (Open Finance / APIs proprietárias) |
| Receita Federal | Consulta CNPJ, certidões | REST / scraping homologado |
| ViaCEP / Correios | Autocompletar endereços | REST |
| Banco Central | Cotações de moeda (PTAX) | REST |
| Serasa/SPC | Consulta de crédito | REST (contratual) |

---

## 11. Observabilidade e Monitoramento

- **Laravel Pulse** — dashboard de saúde da aplicação em produção (queries lentas, jobs, erros)
- **Laravel Horizon** — monitoramento das filas (throughput, failed jobs, workers)
- **Sentry** — rastreamento de exceções com contexto de usuário/empresa
- **Logs estruturados** (JSON) com `monolog` — nível de log por ambiente
- **Health checks** em `/api/health` (banco, redis, fila, SEFAZ)
- **Alertas** de jobs falhos em filas críticas (fiscal, financeiro) via Slack/e-mail

---

## 12. Considerações de Escalabilidade

### Horizontal (mais servidores)
- Sessões e cache centralizados no Redis (stateless)
- Filas no Redis (múltiplos workers em diferentes máquinas)
- Storage centralizado (S3/MinIO para XMLs de NF e PDFs)
- Reverb com suporte a múltiplas instâncias via pub/sub Redis

### Vertical (servidor maior)
- PostgreSQL responde bem ao aumento de RAM (shared_buffers, work_mem)
- Connection pooling com **PgBouncer** para evitar esgotamento de conexões

### Multi-tenancy
- Recomendado desde a modelagem inicial: todas as tabelas com `empresa_id`
- Global Scopes no Eloquent para isolamento automático por tenant
- Índices compostos `(empresa_id, campo_de_busca)` em todas as tabelas principais

---

*Documento feito em: maio/2026*  
*Versões de referência: PHP 8.4 | Laravel 13 | PostgreSQL 16 | Redis 7.x | Laravel Reverb 1.x*
