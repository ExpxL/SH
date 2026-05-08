# ACBr API, TecnoSpeed e SAC Fiscal
> Guia comparativo para Software Houses — integração fiscal com PHP

---

## 1. ACBr API — API REST Oficial do Projeto ACBr

API REST SaaS lançada pelo próprio Projeto ACBr, voltada a quem quer **fugir da manutenção local** dos componentes.

**O que faz:**
- Emissão, consulta, cancelamento e carta de correção de: NF-e, NFC-e, NFS-e, CT-e, MDF-e, NFCom, DC-e
- Distribuição DF-e (download de XMLs emitidos contra o CNPJ)
- Consultas de CNPJ, CPF, CEP e listagem por CNAE
- Armazenamento dos documentos pelo período legal

**Diferencial comercial:**
- Modelo **anual com limite liberado desde o início** — sem franquia mensal, sem créditos que vencem, sem surpresas no boleto.
- Atualizações fiscais, mudanças de schema e novas prefeituras são gerenciadas pela ACBr API, não pela software house.

### 💰 Custos — ACBr API

| Item | Detalhe |
|---|---|
| **Implementação** | Gratuita — ambiente sandbox/homologação disponível após cadastro |
| **Plano de uso** | **Anual**, com limite definido no início do contrato |
| **Modelo de cobrança** | Sem franquia mensal; sem crédito que vence; sem cobrança por CNPJ ou terminal |
| **Preço público** | O modelo comercial é baseado em créditos de utilização, podendo o consumo variar conforme tipo de operação fiscal executada. |
| **Período de testes** | Sandbox disponível gratuitamente |

> ⚠️ Para obter valores, acesse **acbr.api.br** e crie uma conta gratuita. A tabela de preços é enviada diretamente no e-mail.

---

## 2. TecnoSpeed (PlugNotas) — API + Consultoria

API REST SaaS madura, com mais de 18 anos no mercado fiscal e mais de 1.985 software houses atendidas.

**Integração:** JSON via HTTP — compatível com PHP, C#, Java, Python, Delphi e qualquer linguagem.

**Documentos suportados:** NF-e, NFC-e, NFS-e (+1.600 prefeituras), CT-e, MDF-e, NFCom, DC-e.

**Diferenciais:**
- Contingência offline (NeverStop para NFC-e) — emite mesmo sem internet ou SEFAZ instável
- Auditor online com validações técnicas e tributárias
- Gestão de certificados A1
- White Label para a software house
- Envio automático de XML e PDF por e-mail ao destinatário
- Armazenamento em nuvem pelo período legal (11 anos — Ajuste SINIEF nº 2/2025)
- Fura Fila para priorizar autorização na prefeitura (NFS-e)
- Webhook para notificação automática de status de emissão

**Consultoria fiscal:** Equipe de consultores técnicos e tributários disponível desde o onboarding, com suporte especializado em legislação fiscal para NF-e, NFC-e, NFS-e, CT-e e demais documentos.

### 💰 Custos — TecnoSpeed PlugNotas

| Item | Detalhe |
|---|---|
| **Implementação** | Gratuita — sandbox mock disponível sem custo (token público para testes) |
| **Plano de uso** | Sob consulta — modelo **por volume de emissões** ou contrato customizado |
| **Modelo de cobrança** | Por documento emitido ou pacote negociado com o comercial |
| **Preço público** | ⚠️ Não divulgado — requer contato para orçamento |
| **Período de testes** | Sandbox gratuito disponível |

> ⚠️ A TecnoSpeed não exibe tabela pública de preços. O orçamento é feito diretamente pelo time comercial em **plugnotas.com.br** ou **tecnospeed.com.br**. O modelo por volume significa que quanto mais sua base de clientes cresce, maior o custo mensal — fator importante para planejamento financeiro.

---

## 3. SAC Fiscal — Assessoria Fiscal Especializada (sem emissão)

O SAC Fiscal **não emite notas** — é um serviço de **consultoria, suporte fiscal técnico e arquitetura fiscal** para software houses.

**O que contempla a assinatura:**
- Suporte em regras fiscais e tributação (ICMS, ICMS-ST, PIS, COFINS, IPI, IBS, CBS) via abertura de chamados individualizados
- Treinamento em Arquitetura Fiscal de Software e adequação à Reforma Tributária
- 5 workshops especializados (API de Tributação, ICMS + Base PIS/COFINS, PAF-NFCe SC, Super MEI para SWH e outros)
- Boletins informativos de mudanças fiscais, notas técnicas e demandas regulatórias federais, estaduais e municipais
- APIs do SACFiscal.io (dados fiscais para enriquecer o ecossistema de software)
- Encontros ao vivo sobre Reforma Tributária
- Conteúdo técnico com planilhas, modelos de dados e exemplos em Delphi, C#, PHP e JavaScript

### 💰 Custos — SAC Fiscal

| Item | Detalhe |
|---|---|
| **Implementação** | Sem custo adicional de setup |
| **Plano de uso** | **Mensal** com contrato de **12 meses** (renovação automática) |
| **Valor mensal** | **R$ 147,00 / mês** |
| **Valor anual estimado** | ~R$ 1.764,00 / ano |
| **Acessos inclusos** | **2 usuários** por assinatura (3º em diante tem cobrança adicional) |
| **Cancelamento** | A qualquer momento, mediante solicitação |
| **Conteúdo avulso** | Não vendido separadamente — apenas via assinatura |

> ✅ É o único dos três com **preço público e transparente**. Indicado como custo complementar para quem usa ACBr API ou TecnoSpeed e precisa de profundidade no suporte fiscal e tributário.

---

## 4. Resumo Comparativo

| Critério | ACBr API | TecnoSpeed PlugNotas | SAC Fiscal |
|---|:---:|:---:|:---:|
| **Tipo de solução** | API REST SaaS | API REST SaaS | Consultoria pura |
| **Integração PHP** | ✅ HTTP/JSON | ✅ HTTP/JSON | ✅ plano PHP |
| **Emissão Fiscal Completa** | ✅ | ✅ | ❌ |
| **NFS-e multimunicipal** | ✅ (+5.000) | ✅ (+1.600) | N/A |
| **Contingência offline** | ❌ | ✅ NeverStop | N/A |
| **White Label** | ❌ | ✅ | N/A |
| **Consultoria tributária** | ✅ | ✅ | ✅ especializado |
| **Suporte fiscal para PHP** | ✅ | ✅ | ✅ nativo |
| **Atualização fiscal automática** | ✅ | ✅ | N/A |
| **Preço público** | ❌ sob consulta | ❌ sob consulta | ✅ R$ 147/mês |
| **Modelo de cobrança** | Anual s/ franquia | Por volume/contrato | Mensal (12 meses) |
| **Tempo no mercado** | Recente (lançamento ACBr) | +18 anos | ~8 anos |
| **Sandbox gratuito** | ✅ | ✅ | N/A |

---

## 5. Quando Usar Cada Um?

| Cenário | Recomendação |
|---|---|
| PHP ou qualquer linguagem, quer API SaaS sem manutenção, modelo previsível | **ACBr API** |
| Precisa de White Label, contingência offline e NFS-e em +1.600 municípios | **TecnoSpeed PlugNotas** |
| Base de clientes grande com muitas emissões/mês | Avaliar **TecnoSpeed** (volume) vs **ACBr API** (teto anual fixo) |
| Precisa só de suporte fiscal e tributário especializado | **SAC Fiscal** |
| Quer API + consultoria profunda em uma única contratação | **TecnoSpeed** ou **ACBr API + SAC Fiscal** |
| Quer custo previsível sem surpresas por crescimento de clientes | **ACBr API** (modelo anual) + **SAC Fiscal** |

---

## 6. ⚖️ Opinião Final

> Esta análise considera: **preço**, **transparência comercial**, **tempo de mercado**, **estabilidade**, **suporte**, **escalabilidade** e **adequação a software houses PHP**.

### ACBr API — Promissora, mas ainda nova

O maior trunfo da ACBr API é o **modelo de cobrança**: anual com limite definido desde o início, sem franquia mensal e sem cobrança por CNPJ ou terminal. Para uma software house em crescimento, isso representa **previsibilidade financeira real** — algo raro no mercado. A base técnica do Projeto ACBr é sólida e amplamente testada há anos.

O ponto de atenção é que a ACBr API é um lançamento recente. Ainda não tem o histórico de estabilidade e de cobertura municipal que uma solução consolidada tem. Para quem já usa o ecossistema ACBr, é a evolução natural e mais econômica. Para quem está começando do zero, é uma aposta tecnicamente segura, mas que merece acompanhamento.

**Recomendado para:** software houses que já usam ACBr, que buscam custo fixo anual e que têm volume de emissões previsível.

---

### TecnoSpeed PlugNotas — Referência consolidada, custo variável

A TecnoSpeed é, hoje, o player mais maduro e robusto do mercado para APIs fiscais no Brasil. Com mais de 18 anos de operação, mais de 1.985 software houses atendidas, contingência offline, White Label e cobertura em mais de 1.600 municípios para NFS-e, ela entrega o que promete com baixíssimo risco de instabilidade.

O ponto de atenção é o **modelo por volume**: à medida que sua base cresce, o custo cresce junto. Sem tabela pública de preços, a negociação é feita diretamente com o comercial — o que pode ser uma vantagem (flexibilidade) ou uma desvantagem (falta de transparência). Para software houses com volume alto e variável, vale comparar cuidadosamente o TCO anual contra o modelo fixo da ACBr API.

**Recomendado para:** software houses que precisam de máxima confiabilidade, White Label, contingência offline ou que atendem muitos municípios com NFS-e.

---

### SAC Fiscal — Complemento indispensável, não substituto

O SAC Fiscal não concorre diretamente com as APIs — ele **complementa** qualquer uma delas. A R$ 147/mês com 2 acessos inclusos, é um investimento baixo para o que entrega: suporte fiscal individualizado, treinamentos em Reforma Tributária, boletins e APIs de dados fiscais.

Para qualquer software house que lida com tributação complexa (ICMS-ST, Simples Nacional, retenções de NFS-e, adequação à Reforma Tributária), ter acesso a especialistas fiscais sem precisar contratar um contador interno é um diferencial estratégico claro.

**Recomendado para:** toda software house que emite qualquer documento fiscal e quer segurança jurídica e técnica nas decisões tributárias.

---

### 🏆 Combinação recomendada para uma Software House PHP

**Cenário ideal de custo-benefício:**

```
ACBr API (emissão fiscal, modelo anual fixo)
    +
SAC Fiscal (suporte tributário, R$ 147/mês)
```

Essa combinação entrega emissão fiscal completa com custo previsível, suporte técnico e assessoria tributária especializada — tudo sem depender de manutenção local e com alinhamento natural ao ecossistema ACBr.

Se o negócio exigir **White Label**, **contingência offline** ou uma base de municípios muito ampla de NFS-e desde o primeiro dia, substitua a ACBr API pelo **TecnoSpeed PlugNotas** e mantenha o SAC Fiscal como camada de suporte tributário.

---

*Fontes: acbr.api.br · projetoacbr.com.br · plugnotas.com.br · tecnospeed.com.br · sacfiscal.com.br — Consultado em maio/2026*
*⚠️ Preços podem sofrer alterações. Sempre consulte as páginas oficiais antes de contratar.*
