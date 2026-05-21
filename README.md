# Análise de Sistemas — ERP Moderno (Backend)
> Stack: Laravel 13 · PHP 8.4 · PostgreSQL · Redis · Laravel Queues · Sanctum + Fortify · Spatie Permission · Laravel Reverb

---

## Sumário

1. [Visão Geral da Arquitetura](#1-visão-geral-da-arquitetura)
2. [Stack Tecnológica](#2-stack-tecnológica)
3. [Boas Práticas de Código](#3-boas-práticas-de-código)
4. [Cache e Redis — Estratégia Completa](#4-cache-e-redis--estratégia-completa)
5. [Estratégia de Filas, Workers e Jobs](#5-estratégia-de-filas-workers-e-jobs)
6. [Segurança e Autenticação](#6-segurança-e-autenticação)
7. [Comunicação em Tempo Real (Reverb)](#7-comunicação-em-tempo-real-reverb)
8. [Otimização de Banco de Dados](#8-otimização-de-banco-de-dados)
9. [Observabilidade e Monitoramento](#9-observabilidade-e-monitoramento)
10. [Considerações de Escalabilidade](#10-considerações-de-escalabilidade)

---

## 1. Visão Geral da Arquitetura

O sistema adota uma arquitetura **monolítica modular** (Modular Monolith), organizada por domínios de negócio dentro do Laravel. Essa abordagem equilibra a simplicidade operacional de um monólito com a separação de responsabilidades próxima de microsserviços.

```
+----------------------------------------------------------------------------------+
|                           API RESTful (Laravel 13)                               |
|                                                                                  |
|  app/Modules/Fiscal  |  app/Modules/Financeiro  |  ...                           |
|                                                                                  |
|  +----------------------------------------------------------------------------+  |
|  | Controller -> FormRequest -> Service -> Repository                         |  |
|  |        |                     |                                             |  |
|  |      Events              Jobs/Queues                                       |  |
|  +----------------------------------------------------------------------------+  |
|                                                                                  |
|  +----------------------------------------------------------------------------+  |
|  |                    Infraestrutura Transversal                              |  |
|  | Sanctum/Fortify | Spatie | Reverb | ACBR PHP Wrapper                       |  |
|  +----------------------------------------------------------------------------+  |
+----------------------------------------------------------------------------------+
          │                  │                  │
     PostgreSQL            Redis          ACBr Lib (DLL/SO)
                                       (NF-e, NFC-e, NFS-e,
                                        MDF-e, CT-e, Boletos...)
```

### Padrões Adotados

- **Repository Pattern** — abstração da camada de dados, facilita testes e troca de fonte
- **Service Layer** — regras de negócio isoladas dos controllers
- **DTO (Data Transfer Objects)** — contratos claros entre camadas
- **Events & Listeners** — desacoplamento entre domínios (ex.: `NFeAutorizadaEvent` dispara baixa de estoque e lançamento contábil de forma independente)
- **Jobs & Queues** — toda operação com I/O externo (SEFAZ, bancos, e-mail) roda de forma assíncrona
- **Policies** — controle de acesso granular por recurso e por empresa/filial

---

## 2. Stack Tecnológica

### 2.1 PHP 8.4

**Recursos relevantes para o projeto:**
- **Property hooks** — simplifica getters/setters em entidades de domínio sem boilerplate
- **Asymmetric visibility** (`public private(set)`) — modelos de domínio com escrita controlada
- **Typed class constants** — enums e constantes fortemente tipadas (status, tipos de documento)
- **`array_find()` / `array_find_key()`** nativos — úteis em coleções de itens e validações
- **JIT melhorado** — ganho real em processos de cálculo intensivo (relatórios, tributação)

> **💡 Sugestão alternativa:** Não há substituto recomendado — PHP 8.4 é o topo da linha. Manter sempre na última minor para patches de segurança.

> **⚠️ Observação:** Antes de subir para produção, validar todas as dependências (inclusive o wrapper PHP da ACBr e pacotes fiscais) contra o PHP 8.4. Pacotes de integração fiscal tendem a ter ciclo de atualização mais lento.

---

### 2.2 Laravel 13

**Recursos mais relevantes:**
- **Eloquent** com enums nativos, `withCasts` e `LazyCollection` para volumes grandes
- **Form Requests** com regras de validação complexas e mensagens contextualizadas
- **API Resources** para serialização consistente e versionada das respostas
- **Observers** para auditoria automática em entidades críticas
- **Global Scopes** para isolamento multi-tenant transparente
- **Laravel Pulse** (produção) e **Telescope** (desenvolvimento) para observabilidade

> **💡 Sugestão alternativa:** Para times que preferem Actions em vez de Services, o pacote `lorisleiva/laravel-actions` organiza regras de negócio em classes de uso único, reutilizáveis como Jobs, Controllers ou Commands — reduz o peso dos Services em domínios complexos.

> **⚠️ Observação:** Estruturar os módulos desde o início como `app/Modules/{Dominio}/{Controllers,Services,Repositories,Models,Jobs,Events}`. Refatorar um ERP não modularizado depois é um custo enorme.

---

### 2.3 ACBr Lib (PHP Wrapper)

A **ACBr** é a biblioteca adotada para todas as operações de documentos fiscais e financeiros. Ela oferece suporte nativo e homologado para o mercado brasileiro, cobrindo NF-e, NFC-e, NFS-e, MDF-e, CT-e, geração de boletos bancários, SPED e muito mais — dispensando a manutenção de múltiplos pacotes para cada documento.

A integração com PHP é feita via **wrapper** que se comunica com a biblioteca nativa (DLL no Windows / .so no Linux) por socket TCP ou por chamadas diretas.

> **⚠️ Observação:** O detalhamento de integração, configuração do daemon ACBr, mapeamento de comandos e tratamento de retornos fiscais será abordado em documento específico. Neste documento a ACBr é tratada como componente de infraestrutura já resolvido.

> **⚠️ Observação operacional:** Em ambiente Linux (produção), garantir que o processo do ACBr Monitor esteja rodando como serviço supervisionado (systemd/supervisor) e monitorado. A queda silenciosa desse processo derruba toda a emissão fiscal.

---

### 2.4 PostgreSQL

**Destaques para este projeto:**
- **JSONB** — armazenamento de retornos da SEFAZ, XMLs processados e payloads de auditoria
- **Particionamento** por competência em tabelas de alto volume
- **CTEs recursivas** — plano de contas hierárquico, centros de custo em árvore
- **Window functions** — relatórios contábeis (DRE, balancete) sem subqueries aninhadas
- **Materialized Views** — snapshots de relatórios pesados com refresh agendado
- **Índices parciais** — ex.: `WHERE status = 'pendente'` em tabelas de títulos e NFs

> **💡 Sugestão alternativa:** MySQL 8.4 é viável para volumes menores, mas perde JSONB real, particionamento nativo e CTEs complexas. Não recomendado para projetos com alta carga fiscal.

> **⚠️ Observação:** Planejar particionamento por `competencia` (ano/mês) nas tabelas de NF, lançamentos e movimentações desde a modelagem inicial. Documentos fiscais têm retenção legal de 5 anos — essas tabelas crescem de forma previsível e volumosa.

---

### 2.5 Redis

Cache distribuído, sessões, filas e pub/sub — detalhado na seção 4.

> **💡 Sugestão alternativa:** **Valkey** (fork open-source do Redis mantido pela Linux Foundation, surgido após a mudança de licença do Redis 7.4+) é 100% compatível com o driver do Laravel. Recomendado para projetos que queiram evitar dependência de licença proprietária sem nenhuma mudança de código.

> **⚠️ Observação:** Configurar persistência **RDB + AOF** em produção. A perda da fila em um crash sem persistência significa jobs fiscais perdidos — NFs que estavam em processamento precisarão de tratamento manual.

---

### 2.6 Laravel Queues + Horizon

Filas assíncronas com Redis como driver. O **Laravel Horizon** é tratado como componente obrigatório (não opcional) — ele fornece o dashboard de monitoramento, configuração de workers por fila e métricas de throughput.

> **⚠️ Observação:** Horizon é a camada de visibilidade das filas. Sem ele em produção, falhas em jobs fiscais podem passar despercebidas. Configurar alertas de `failed jobs` para notificação imediata.

---

### 2.7 Sanctum + Fortify

- **Sanctum** — tokens de API com escopos, para integrações externas e futuro app mobile
- **Fortify** — fluxo completo de autenticação web (login, 2FA, reset de senha, confirmação de e-mail)

> **💡 Sugestão alternativa:** Para SSO corporativo (Active Directory, Okta, Google Workspace), adicionar **Laravel Socialite** ou migrar para **Laravel Passport** (OAuth2 completo). Sanctum + Fortify atendem bem o fluxo padrão de usuário/senha + 2FA.

> **⚠️ Observação:** Implementar log de acesso (IP, user-agent, timestamp) em todos os logins — pode ser exigido em auditorias fiscais e de compliance.

---

### 2.8 Spatie Permission

RBAC (Role-Based Access Control) granular por módulo e por empresa.

> **💡 Sugestão alternativa:** O Spatie é o padrão consolidado do ecossistema. Para ACL por registro específico (ex.: "usuário X só acessa filial Y"), combiná-lo com **Policies** do Laravel — eles trabalham em harmonia.

> **⚠️ Observação:** Em ambientes multi-tenant, garantir que o cache de permissões seja isolado por tenant. O Spatie usa um cache global por padrão — customizar o store de cache para incluir o ID da empresa na chave.

---

### 2.9 Laravel Reverb

WebSocket server nativo do Laravel para broadcasting de eventos em tempo real.

> **💡 Sugestão alternativa:** **Pusher** (gerenciado) elimina a operação do servidor WebSocket. O Broadcasting do Laravel é compatível com ambos sem alteração de código. Para ambientes on-premise ou com restrições de dados (LGPD), o Reverb self-hosted é o caminho.

> **⚠️ Observação:** Colocar NGINX como proxy reverso na frente do Reverb para TLS termination e load balancing. Em produção com muitas conexões simultâneas, monitorar consumo de file descriptors e memória do processo Reverb.

---

## 3. Boas Práticas de Código

### 3.1 Estrutura de Módulos

```
app/
└── Modules/
    ├── Fiscal/
    │   ├── Controllers/
    │   ├── Services/          # Regras de negócio
    │   ├── Repositories/      # Abstração de acesso a dados
    │   ├── Jobs/              # Operações assíncronas
    │   ├── Events/            # Eventos de domínio
    │   ├── Listeners/
    │   ├── DTOs/              # Transferência de dados entre camadas
    │   ├── Models/
    │   └── Requests/          # Form Requests com validação
    ├── Financeiro/
    ├── Estoque/
    └── ...
```

### 3.2 Tipagem Estrita e Enums

Usar `declare(strict_types=1)` em todos os arquivos e enums nativos do PHP 8.x para representar estados:

```php
<?php

declare(strict_types=1);

enum StatusDocumentoFiscal: string
{
    case Pendente    = 'pendente';
    case Processando = 'processando';
    case Autorizado  = 'autorizado';
    case Rejeitado   = 'rejeitado';
    case Cancelado   = 'cancelado';
    case Inutilizado = 'inutilizado';

    public function label(): string
    {
        return match($this) {
            self::Pendente    => 'Aguardando envio',
            self::Processando => 'Em processamento na SEFAZ',
            self::Autorizado  => 'Autorizado',
            self::Rejeitado   => 'Rejeitado',
            self::Cancelado   => 'Cancelado',
            self::Inutilizado => 'Inutilizado',
        };
    }

    public function isFinal(): bool
    {
        return in_array($this, [self::Autorizado, self::Cancelado, self::Inutilizado]);
    }
}
```

No Eloquent, usar cast direto para o enum:

```php
protected $casts = [
    'status' => StatusDocumentoFiscal::class,
];
```

### 3.3 DTOs para Transferência entre Camadas

Evitar passar arrays associativos entre camadas — usar DTOs tipados:

```php
<?php

declare(strict_types=1);

final readonly class EmissaoDocumentoDTO
{
    public function __construct(
        public readonly int    $empresaId,
        public readonly string $tipo,           // NF-e, NFC-e, NFS-e...
        public readonly int    $destinatarioId,
        public readonly array  $itens,
        public readonly string $naturezaOperacao,
        public readonly ?string $observacao = null,
    ) {}

    public static function fromRequest(array $data): self
    {
        return new self(
            empresaId:         (int) $data['empresa_id'],
            tipo:              $data['tipo'],
            destinatarioId:    (int) $data['destinatario_id'],
            itens:             $data['itens'],
            naturezaOperacao:  $data['natureza_operacao'],
            observacao:        $data['observacao'] ?? null,
        );
    }
}
```

### 3.4 Service Layer — Responsabilidade Única

Cada Service deve ter responsabilidade clara. Evitar Services "god objects":

```php
// ✅ Correto — responsabilidade única
class EmissaoDocumentoService { ... }
class CancelamentoDocumentoService { ... }
class ConsultaStatusSefazService { ... }

// ❌ Evitar — Service acumulando tudo
class DocumentoFiscalService { /* 1500 linhas */ }
```

### 3.5 Form Requests com Validação Rica

Centralizar validação e autorização nos Form Requests, nunca nos controllers:

```php
<?php

declare(strict_types=1);

class EmitirDocumentoRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('documento-fiscal.emitir');
    }

    public function rules(): array
    {
        return [
            'tipo'               => ['required', Rule::enum(TipoDocumentoFiscal::class)],
            'destinatario_id'    => ['required', 'integer', 'exists:entidades,id'],
            'itens'              => ['required', 'array', 'min:1'],
            'itens.*.produto_id' => ['required', 'integer', 'exists:produtos,id'],
            'itens.*.quantidade' => ['required', 'numeric', 'gt:0'],
            'itens.*.valor'      => ['required', 'numeric', 'gt:0'],
        ];
    }
}
```

### 3.6 Controllers Enxutos

Controllers devem apenas orquestrar — receber, delegar e responder:

```php
<?php

declare(strict_types=1);

class DocumentoFiscalController extends Controller
{
    public function __construct(
        private readonly EmissaoDocumentoService $emissaoService,
    ) {}

    public function store(EmitirDocumentoRequest $request): JsonResponse
    {
        $dto = EmissaoDocumentoDTO::fromRequest($request->validated());
        $documento = $this->emissaoService->iniciar($dto);

        return DocumentoFiscalResource::make($documento)
            ->response()
            ->setStatusCode(202); // Accepted — processamento é assíncrono
    }
}
```

### 3.7 Repositories com Interface

Desacoplar a implementação do contrato facilita testes e eventual troca de fonte de dados:

```php
interface DocumentoFiscalRepositoryInterface
{
    public function findById(int $id): ?DocumentoFiscal;
    public function findPendentesPorEmpresa(int $empresaId): Collection;
    public function salvarComStatus(DocumentoFiscal $doc, StatusDocumentoFiscal $status): void;
}

class EloquentDocumentoFiscalRepository implements DocumentoFiscalRepositoryInterface
{
    // Implementação com Eloquent
}
```

### 3.8 Events e Listeners para Desacoplamento

Usar eventos de domínio para evitar que o Service de emissão conheça os módulos de estoque, contabilidade e notificações:

```php
// Service de emissão apenas dispara o evento
event(new DocumentoFiscalAutorizadoEvent($documento));

// Listeners independentes reagem ao evento
class BaixarEstoqueListener implements ShouldQueue { ... }
class LancarContabilidadeListener implements ShouldQueue { ... }
class EnviarEmailDocumentoListener implements ShouldQueue { ... }
class NotificarViaReverbListener { ... }
```

Registrar os listeners com `ShouldQueue` para que sejam executados assincronamente via fila.

### 3.9 Soft Deletes em Entidades Críticas

Nunca deletar fisicamente registros de documentos fiscais, financeiros ou contábeis:

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class DocumentoFiscal extends Model
{
    use SoftDeletes;
    // deleted_at nunca é nulo em registros "excluídos"
}
```

### 3.10 Transactions para Operações Compostas

Qualquer operação que escreva em mais de uma tabela deve usar transaction:

```php
DB::transaction(function () use ($dto) {
    $titulo = $this->contasReceberRepo->criar($dto);
    $parcelas = $this->parcelaRepo->gerarParcelas($titulo, $dto->condicaoPagamento);
    $this->historicoRepo->registrar($titulo, 'criacao');
    
    event(new TituloReceberCriadoEvent($titulo));
});
```

---

## 4. Cache e Redis — Estratégia Completa

### 4.1 Organização por Responsabilidade

O Redis é usado para **quatro finalidades distintas** — manter databases separados evita conflito de chaves e facilita flush seletivo:

| Database Redis | Uso | Configuração Laravel |
|---|---|---|
| `db 0` | Cache da aplicação | `REDIS_DB=0` |
| `db 1` | Sessões | `REDIS_SESSION_DB=1` |
| `db 2` | Filas (Queues) | `REDIS_QUEUE_DB=2` |
| `db 3` | Pub/Sub do Reverb | `REDIS_REVERB_DB=3` |

### 4.2 Nomenclatura de Chaves de Cache

Adotar convenção hierárquica para facilitar invalidação seletiva:

```
{tenant}:{modulo}:{entidade}:{identificador}:{dados}

Exemplos:
empresa:42:fiscal:config           → configurações fiscais da empresa 42
empresa:42:produto:1837:preco      → preço do produto 1837
empresa:42:relatorio:dre:2025-01   → DRE de janeiro/2025
global:tabela:ibpt:ncm:62030000    → alíquota IBPT do NCM
global:tabela:cfop:5102            → descrição do CFOP
```

### 4.3 Tabela de TTLs por Tipo de Dado

| Chave | TTL | Justificativa |
|---|---|---|
| Configurações fiscais da empresa | 1h | Altera raramente; invalidar no save |
| Tabela IBPT / NCM | 24h | Atualizada mensalmente pela Receita |
| Alíquotas estaduais (ICMS) | 12h | Mudanças em decreto, não em tempo real |
| CFOP — descrições | 48h | Quase imutável |
| Cotação de moedas (PTAX) | 1h | API Banco Central atualiza no fechamento |
| Permissões de usuário (Spatie) | 5min | Invalidar imediatamente em revogação |
| Resultados de relatórios gerenciais | 10min | Aceitável para dashboards |
| Sessões de usuário | 2h (sliding) | Expirar por inatividade |
| Locks de operação (emissão, estoque) | 30s | Apenas para serializar concorrência |

### 4.4 Cache Aside Pattern — Padrão de Uso

```php
// Helper para cache com namespace de tenant
class TenantCache
{
    public static function remember(
        int $empresaId,
        string $chave,
        int $ttlSeconds,
        Closure $callback
    ): mixed {
        $key = "empresa:{$empresaId}:{$chave}";
        return Cache::remember($key, $ttlSeconds, $callback);
    }

    public static function forget(int $empresaId, string $chave): void
    {
        Cache::forget("empresa:{$empresaId}:{$chave}");
    }
}

// Uso no Service
$config = TenantCache::remember(
    empresaId: $empresaId,
    chave: 'fiscal:config',
    ttlSeconds: 3600,
    callback: fn() => ConfiguracaoFiscal::where('empresa_id', $empresaId)->first()
);
```

### 4.5 Invalidação Proativa via Observers

Nunca depender apenas do TTL para dados que mudam por ação do usuário:

```php
class ConfiguracaoFiscalObserver
{
    public function saved(ConfiguracaoFiscal $config): void
    {
        TenantCache::forget($config->empresa_id, 'fiscal:config');
    }
}
```

### 4.6 Locks Distribuídos para Operações Críticas

Evitar race conditions em emissão de documentos e movimentações de estoque:

```php
$lock = Cache::lock("emissao:pedido:{$pedidoId}", 30);

if ($lock->get()) {
    try {
        // Processar emissão
    } finally {
        $lock->release();
    }
} else {
    throw new DocumentoEmProcessamentoException(
        "Documento do pedido #{$pedidoId} já está sendo processado."
    );
}
```

### 4.7 Rate Limiting com Redis

Proteger endpoints de emissão contra duplicidade e abuso:

```php
// Em RouteServiceProvider ou no Controller
RateLimiter::for('emissao-fiscal', function (Request $request) {
    return Limit::perMinute(20)->by($request->user()->empresa_id);
});
```

---

## 5. Estratégia de Filas, Workers e Jobs

### 5.1 Filas por Domínio e Prioridade

| Fila | Prioridade | Workers dedicados | Timeout | Responsabilidade |
|---|---|---|---|---|
| `fiscal-critical` | Máxima | 2+ | 120s | Comunicação com SEFAZ/ACBr, retornos |
| `financial` | Alta | 2 | 60s | Boletos, retorno bancário, PIX |
| `stock` | Média | 1 | 30s | Baixa e reserva de estoque |
| `reports` | Baixa | 1 | 600s | Geração de relatórios pesados, SPED |
| `notifications` | Baixa | 1 | 30s | E-mails, webhooks, Reverb broadcasts |
| `default` | Normal | 1 | 60s | Operações assíncronas gerais |

### 5.2 Configuração dos Workers (Supervisor)

```ini
; /etc/supervisor/conf.d/erp-workers.conf

[program:worker-fiscal]
command=php /var/www/artisan queue:work redis --queue=fiscal-critical --timeout=120 --tries=3 --sleep=1 --max-jobs=500 --max-time=3600
numprocs=2
autostart=true
autorestart=true
redirect_stderr=true

[program:worker-financial]
command=php /var/www/artisan queue:work redis --queue=financial --timeout=60 --tries=3 --sleep=1 --max-jobs=1000 --max-time=3600
numprocs=2
autostart=true
autorestart=true

[program:worker-general]
command=php /var/www/artisan queue:work redis --queue=stock,notifications,default --timeout=60 --tries=3 --sleep=2 --max-jobs=1000 --max-time=3600
numprocs=1
autostart=true
autorestart=true

[program:worker-reports]
command=php /var/www/artisan queue:work redis --queue=reports --timeout=600 --tries=1 --sleep=3 --max-jobs=50 --max-time=3600
numprocs=1
autostart=true
autorestart=true
```

> **Nota:** `--max-jobs` e `--max-time` são importantes para evitar vazamento de memória em workers de longa duração — o worker reinicia sozinho após atingir o limite.

### 5.3 Anatomia de um Job Crítico

```php
<?php

declare(strict_types=1);

class ProcessarRetornoSefazJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public string  $queue    = 'fiscal-critical';
    public int     $tries    = 3;
    public int     $timeout  = 120;
    public int     $maxExceptions = 2;
    public array   $backoff  = [30, 120, 300]; // 30s, 2min, 5min

    public function __construct(
        private readonly int $documentoId,
        private readonly string $recibo,
    ) {}

    public function handle(ConsultaSefazService $sefaz): void
    {
        $documento = DocumentoFiscal::lockForUpdate()->findOrFail($this->documentoId);

        // Idempotência: ignorar se já em status final
        if ($documento->status->isFinal()) {
            return;
        }

        $resultado = $sefaz->consultarRecibo($this->recibo);

        DB::transaction(function () use ($documento, $resultado) {
            $documento->update(['status' => $resultado->status]);
            DocumentoFiscalEvento::create([...]);
        });

        event(new DocumentoFiscalStatusAlteradoEvent($documento));
    }

    public function failed(Throwable $e): void
    {
        DocumentoFiscal::find($this->documentoId)?->update([
            'status' => StatusDocumentoFiscal::Rejeitado,
            'erro_descricao' => $e->getMessage(),
        ]);

        // Notificar equipe de operações
        Notification::route('slack', config('services.slack.ops_channel'))
            ->notify(new JobCriticoFalhouNotification($this, $e));
    }

    public function uniqueId(): string
    {
        return "retorno-sefaz-{$this->documentoId}";
    }
}
```

### 5.4 Idempotência nos Jobs

Jobs críticos **devem ser idempotentes** — reprocessar o mesmo job duas vezes não pode causar duplicidade:

- Verificar estado atual antes de processar (`status->isFinal()`)
- Usar `ShouldBeUnique` + `uniqueId()` para evitar jobs duplicados na fila
- Usar `lockForUpdate()` no Eloquent para serializar acesso concorrente ao mesmo registro

```php
// Garantir que não existam dois jobs do mesmo documento na fila simultaneamente
class EmitirDocumentoJob implements ShouldQueue, ShouldBeUnique
{
    public int $uniqueFor = 600; // segundos

    public function uniqueId(): string
    {
        return "emissao-{$this->documentoId}";
    }
}
```

### 5.5 Horizon — Configuração Recomendada

```php
// config/horizon.php
'environments' => [
    'production' => [
        'supervisor-fiscal' => [
            'connection'  => 'redis',
            'queue'       => ['fiscal-critical'],
            'balance'     => 'simple',
            'processes'   => 2,
            'tries'       => 3,
            'timeout'     => 120,
        ],
        'supervisor-financial' => [
            'queue'     => ['financial'],
            'balance'   => 'auto',
            'processes' => 2,
            'timeout'   => 60,
        ],
        'supervisor-general' => [
            'queue'     => ['stock', 'notifications', 'default'],
            'balance'   => 'auto',
            'minProcesses' => 1,
            'maxProcesses' => 5,
            'timeout'   => 60,
        ],
        'supervisor-reports' => [
            'queue'    => ['reports'],
            'balance'  => 'simple',
            'processes'=> 1,
            'timeout'  => 600,
        ],
    ],
],
```

### 5.6 Scheduled Jobs (Agendados)

```php
// app/Console/Kernel.php
protected function schedule(Schedule $schedule): void
{
    // Consultar lotes pendentes na SEFAZ a cada 2 minutos
    $schedule->job(ConsultarLotesPendentesSefazJob::class, 'fiscal-critical')
             ->everyTwoMinutes()
             ->withoutOverlapping();

    // Processar retornos bancários (CNAB) diariamente
    $schedule->job(ProcessarRetornosBancariosJob::class, 'financial')
             ->dailyAt('06:00')
             ->withoutOverlapping();

    // Verificar boletos vencidos
    $schedule->job(VerificarBoletosVencidosJob::class, 'financial')
             ->dailyAt('07:00');

    // Limpar cache de relatórios antigos
    $schedule->command('cache:prune-stale-tags')
             ->hourly();
}
```

---

## 6. Segurança e Autenticação

### 6.1 Sanctum — Tokens com Escopos

```php
// Criar token com habilidades (escopos)
$token = $user->createToken('integracao-erp', [
    'fiscal:read',
    'fiscal:emitir',
    'financeiro:read',
])->plainTextToken;

// Verificar escopo no controller/middleware
$request->user()->tokenCan('fiscal:emitir');

// Ou via middleware em rotas
Route::middleware(['auth:sanctum', 'abilities:fiscal:emitir'])->group(function () {
    Route::post('/fiscal/documentos', ...);
});
```

### 6.2 Fortify — 2FA Obrigatório por Role

```php
// Forçar 2FA para roles críticos via middleware customizado
class Require2FAMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $user = $request->user();

        if ($user->hasAnyRole(['contador', 'financeiro', 'admin']) 
            && ! $user->hasEnabledTwoFactorAuthentication()) 
        {
            return response()->json([
                'message' => '2FA obrigatório para este perfil de acesso.',
                'redirect' => '/configuracoes/seguranca',
            ], 403);
        }

        return $next($request);
    }
}
```

### 6.3 Auditoria com Spatie Activity Log

```php
class DocumentoFiscal extends Model
{
    use LogsActivity;

    protected static LogOptions $activityLogOptions;

    public static function getActivitylogOptions(): LogOptions
    {
        return LogOptions::defaults()
            ->logAll()
            ->logOnlyDirty()        // Só loga campos que mudaram
            ->dontSubmitEmptyLogs() // Não gera log se nada mudou
            ->useLogName('fiscal');
    }
}
```

### 6.4 Proteção de Dados Sensíveis

```php
class Empresa extends Model
{
    protected $casts = [
        // Certificado digital e dados bancários encriptados em AES-256
        'certificado_conteudo' => 'encrypted',
        'certificado_senha'    => 'encrypted',
        'dados_bancarios'      => 'encrypted:array',
    ];

    protected $hidden = [
        'certificado_conteudo',
        'certificado_senha',
    ];
}
```

---

## 7. Comunicação em Tempo Real (Reverb)

### 7.1 Canais por Domínio

```php
// routes/channels.php

// Canal privado por empresa — validação de acesso obrigatória
Broadcast::channel('empresa.{empresaId}.fiscal', function (User $user, int $empresaId) {
    return $user->empresa_id === $empresaId 
        && $user->can('fiscal.visualizar');
});

Broadcast::channel('empresa.{empresaId}.financeiro', function (User $user, int $empresaId) {
    return $user->empresa_id === $empresaId 
        && $user->can('financeiro.visualizar');
});

// Canal de progresso de job — acessível apenas pelo usuário dono
Broadcast::channel('job-progress.{userId}', function (User $user, int $userId) {
    return $user->id === $userId;
});
```

### 7.2 Eventos Transmitidos

```php
class DocumentoFiscalStatusAlteradoEvent implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function __construct(
        private readonly DocumentoFiscal $documento,
    ) {}

    public function broadcastOn(): array
    {
        return [
            new PrivateChannel("empresa.{$this->documento->empresa_id}.fiscal"),
        ];
    }

    public function broadcastAs(): string
    {
        return 'documento.status-alterado';
    }

    public function broadcastWith(): array
    {
        return [
            'id'     => $this->documento->id,
            'status' => $this->documento->status->value,
            'label'  => $this->documento->status->label(),
        ];
    }
}
```

---

## 8. Otimização de Banco de Dados

### 8.1 Eager Loading — Sem N+1

```php
// ❌ N+1 — executa 1 query + N queries de itens
$documentos = DocumentoFiscal::all();
foreach ($documentos as $doc) {
    echo $doc->destinatario->nome; // query por iteração
}

// ✅ Correto — 2 queries totais
$documentos = DocumentoFiscal::with(['destinatario', 'itens.produto'])->get();
```

Usar `withCount()` para contadores sem carregar a relação:

```php
DocumentoFiscal::withCount('itens')->get();
```

### 8.2 LazyCollections para Volumes Grandes

```php
// ✅ Para exportações e relatórios pesados — processa um registro por vez
DocumentoFiscal::where('competencia', '2025-01')
    ->cursor() // LazyCollection — não carrega tudo na memória
    ->each(function (DocumentoFiscal $doc) {
        $this->spedBuilder->adicionarRegistro($doc);
    });
```

### 8.3 Índices Estratégicos

```php
// Migrations com índices compostos para queries frequentes do ERP
Schema::create('documentos_fiscais', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('empresa_id');
    $table->string('status');
    $table->date('competencia');
    $table->timestamps();

    // Índices compostos — ordem importa (empresa_id sempre primeiro)
    $table->index(['empresa_id', 'status']);
    $table->index(['empresa_id', 'competencia']);
    $table->index(['empresa_id', 'status', 'competencia']);

    // Índice parcial via raw — só indexa registros pendentes
    // (usar migration raw para PostgreSQL)
});

// Migration raw para índice parcial no PostgreSQL
DB::statement("
    CREATE INDEX idx_documentos_pendentes 
    ON documentos_fiscais (empresa_id, created_at) 
    WHERE status IN ('pendente', 'processando')
");
```

### 8.4 Select Explícito — Nunca SELECT *

```php
// ❌ Carrega todos os campos, incluindo XML e dados pesados
$documentos = DocumentoFiscal::all();

// ✅ Apenas campos necessários para a listagem
$documentos = DocumentoFiscal::select(['id', 'numero', 'status', 'valor_total', 'emitido_em'])
    ->where('empresa_id', $empresaId)
    ->paginate(50);
```

### 8.5 Connection Pooling com PgBouncer

Em produção, colocar **PgBouncer** na frente do PostgreSQL para limitar e reciclar conexões — cada worker de fila abre sua própria conexão, e sem pooling o PostgreSQL pode ser sobrecarregado facilmente:

```
Workers (N processos) → PgBouncer (pool fixo) → PostgreSQL
```

Configuração recomendada: modo `transaction` para APIs REST e workers de fila.

---

## 9. Observabilidade e Monitoramento

### 9.1 Stack de Observabilidade

| Ferramenta | Ambiente | Finalidade |
|---|---|---|
| **Laravel Telescope** | Desenvolvimento | Queries, jobs, requests, cache hits, logs |
| **Laravel Pulse** | Produção | Dashboard de saúde, slow queries, exceções |
| **Laravel Horizon** | Produção | Filas, workers, throughput, failed jobs |
| **Sentry** | Produção | Rastreamento de exceções com contexto |
| **Monolog (JSON)** | Produção | Logs estruturados para aggregators (Grafana Loki, etc.) |

### 9.2 Health Check Endpoint

```php
// routes/api.php
Route::get('/health', function () {
    $checks = [
        'database' => DB::connection()->getPdo() !== null,
        'redis'    => Cache::store('redis')->ping(),
        'queue'    => Queue::size('fiscal-critical') < 1000, // alerta se fila acumular
        'storage'  => Storage::disk('local')->exists('.gitignore'),
    ];

    $healthy = ! in_array(false, $checks, true);

    return response()->json([
        'status' => $healthy ? 'ok' : 'degraded',
        'checks' => $checks,
    ], $healthy ? 200 : 503);
});
```

### 9.3 Logs Estruturados

```php
// Padrão de logging com contexto — facilita busca e correlação
Log::channel('fiscal')->info('Documento fiscal autorizado', [
    'empresa_id'   => $doc->empresa_id,
    'documento_id' => $doc->id,
    'tipo'         => $doc->tipo,
    'numero'       => $doc->numero,
    'chave_acesso' => $doc->chave_acesso,
    'duracao_ms'   => $duracaoMs,
    'user_id'      => auth()->id(),
]);
```

---

## 10. Considerações de Escalabilidade

### 10.1 Escalabilidade Horizontal

A stack é projetada para escalar horizontalmente sem alterações de código:
- **Redis centralizado** — cache, sessões e filas compartilhados entre instâncias
- **Múltiplos workers** em servidores diferentes, todos consumindo a mesma fila Redis
- **Storage compartilhado** (S3/MinIO) para XMLs, PDFs e arquivos gerados
- **Reverb** em múltiplas instâncias com pub/sub via Redis como backbone

### 10.2 Multi-tenancy desde o Início

```php
// Global Scope aplicado a todos os models do ERP
class EmpresaScope implements Scope
{
    public function apply(Builder $builder, Model $model): void
    {
        if (auth()->check()) {
            $builder->where('empresa_id', auth()->user()->empresa_id);
        }
    }
}

// Todos os models críticos registram o scope
class DocumentoFiscal extends Model
{
    protected static function booted(): void
    {
        static::addGlobalScope(new EmpresaScope());
    }
}
```

### 10.3 Banco de Dados — Separação de Read/Write

Para volumes elevados, separar conexões de leitura e escrita via réplica:

```php
// config/database.php
'pgsql' => [
    'read'  => ['host' => env('DB_READ_HOST')],
    'write' => ['host' => env('DB_WRITE_HOST')],
    // ... demais configurações
],
```

Relatórios e consultas de listagem vão para a réplica de leitura automaticamente; operações de escrita vão para o primary.

---

*Documento gerado em: maio/2026*  
*Versões de referência: PHP 8.4 · Laravel 13 · PostgreSQL 16 · Redis 7.x · ACBr Lib (PHP Wrapper) · Laravel Reverb 1.x*
