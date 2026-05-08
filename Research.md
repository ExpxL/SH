# ACBr API, TecnoSpeed PlugNotas e SAC Fiscal
> Guia comparativo para Software Houses — APIs fiscais, contingência e suporte tributário

---

# 1. Visão Geral

O mercado fiscal brasileiro exige atualização constante, adaptação rápida às mudanças legislativas e alta disponibilidade operacional. Para software houses, isso significa lidar simultaneamente com:

- emissão fiscal eletrônica;
- integrações municipais e estaduais;
- contingência operacional;
- armazenamento legal de XMLs;
- suporte tributário;
- mudanças frequentes de schema e legislação.

Este documento apresenta uma análise técnica comparativa entre:

- **ACBr API**
- **TecnoSpeed PlugNotas**
- **SAC Fiscal**

com foco em:

- integração via API REST;
- software houses PHP (mas aplicável a qualquer linguagem);
- previsibilidade financeira;
- escalabilidade;
- suporte técnico e tributário;
- operação SaaS moderna.

---

# 2. ACBr API — API REST Oficial do Projeto ACBr

A ACBr API é a plataforma SaaS REST oficial do ecossistema ACBr, criada para simplificar integrações fiscais sem depender da manutenção local tradicional dos componentes.

## Principais funcionalidades

- Emissão de:
  - NF-e
  - NFC-e
  - NFS-e
  - CT-e
  - MDF-e
  - NFCom
  - DC-e

- Consulta e cancelamento de documentos
- Carta de Correção Eletrônica (CC-e)
- Distribuição DF-e
- Consultas de:
  - CNPJ
  - CPF
  - CEP
  - CNAE

- Armazenamento de XMLs pelo período legal
- API REST via HTTP/JSON

## Características técnicas

- Modelo SaaS
- Integração desacoplada
- Atualizações fiscais centralizadas
- Gerenciamento automático de schemas e layouts fiscais
- Compatível com:
  - PHP
  - C#
  - Delphi
  - Java
  - Python
  - Node.js
  - qualquer linguagem HTTP/JSON

## Modelo comercial

O principal diferencial da ACBr API é o modelo de cobrança baseado em contrato anual com limite previamente definido, reduzindo imprevisibilidade de crescimento por emissão individual.

### Custos — ACBr API

| Item | Detalhe |
|---|---|
| Implementação | Gratuita |
| Sandbox/Homologação | Disponível |
| Modelo de cobrança | Contrato anual |
| Franquia mensal | Não informado |
| Cobrança por terminal/CNPJ | Não divulgada |
| Preço público | Não divulgado |
| Contratação | Sob consulta |

> ⚠️ Os valores comerciais são fornecidos diretamente pela equipe da ACBr API após cadastro.

---

# 3. TecnoSpeed PlugNotas — API Fiscal SaaS

O PlugNotas é a plataforma fiscal SaaS da TecnoSpeed, focada em emissão fiscal em larga escala, contingência operacional e integração multimunicipal.

## Principais funcionalidades

- Emissão de:
  - NF-e
  - NFC-e
  - NFS-e
  - CT-e
  - MDF-e
  - NFCom
  - DC-e

- Webhooks de acompanhamento
- Envio automático de XML/PDF
- Gestão de certificados digitais A1
- White Label
- Armazenamento em nuvem
- API REST HTTP/JSON

## Diferenciais operacionais

### Contingência NFC-e

O PlugNotas possui mecanismos próprios voltados à continuidade operacional em cenários de indisponibilidade da SEFAZ ou conectividade.

Isso inclui:
- contingência offline;
- filas de retransmissão;
- reprocessamento automático;
- mecanismos de priorização operacional.

### Fura Fila (NFS-e)

O recurso conhecido comercialmente como “Fura Fila” atua como priorização interna de processamento para determinadas emissões de NFS-e, reduzindo o tempo de espera em cenários de alta demanda.

### Cobertura municipal

A plataforma possui ampla cobertura multimunicipal para NFS-e, com suporte a centenas de provedores e grande quantidade de municípios integrados.

## Características técnicas

- Arquitetura SaaS distribuída
- Processamento assíncrono
- Uso intensivo de filas e workers
- Integração REST desacoplada
- Compatível com:
  - PHP
  - Java
  - Delphi
  - C#
  - Python
  - Node.js
  - outras linguagens HTTP

## Custos — PlugNotas

| Item | Detalhe |
|---|---|
| Implementação | Gratuita |
| Sandbox | Disponível |
| Modelo comercial | Sob consulta |
| Modelo de cobrança | Volume de emissões ou contrato customizado |
| Preço público | Não divulgado |

> ⚠️ O custo pode variar conforme volume de emissões, municípios utilizados e serviços contratados.

---

# 4. SAC Fiscal — Consultoria e Arquitetura Tributária

O SAC Fiscal não é uma plataforma de emissão fiscal.

Trata-se de um serviço especializado em:
- suporte tributário;
- arquitetura fiscal;
- consultoria técnica;
- atualização legislativa.

## O que contempla

- Suporte tributário especializado
- Apoio em:
  - ICMS
  - ICMS-ST
  - PIS
  - COFINS
  - IPI
  - IBS
  - CBS

- Workshops técnicos
- Treinamentos sobre Reforma Tributária
- APIs auxiliares de dados fiscais
- Boletins regulatórios
- Modelos técnicos e exemplos de integração

## Público-alvo

- Software houses
- ERPs
- Desenvolvedores fiscais
- Equipes tributárias internas

## Custos — SAC Fiscal

| Item | Detalhe |
|---|---|
| Modelo | Assinatura mensal |
| Contrato | 12 meses |
| Valor mensal público | R$ 147/mês |
| Usuários inclusos | 2 |
| Setup | Sem custo |

> ⚠️ Valores sujeitos a alteração pelo fornecedor.

---

# 5. Comparativo Consolidado

| Critério | ACBr API | PlugNotas | SAC Fiscal |
|---|:---:|:---:|:---:|
| Tipo de solução | API REST SaaS | API REST SaaS | Consultoria |
| Emissão fiscal | ✅ | ✅ | ❌ |
| Integração HTTP/JSON | ✅ | ✅ | N/A |
| NFC-e | ✅ | ✅ | ❌ |
| NFS-e multimunicipal | ✅ | ✅ | ❌ |
| Webhooks | Parcial | ✅ | ❌ |
| White Label | Não divulgado | ✅ | ❌ |
| Armazenamento legal | ✅ | ✅ | ❌ |
| Contingência NFC-e | Suportada via ecossistema ACBr | ✅ mecanismos próprios |
| Consultoria tributária | Limitada | Parcial | ✅ especializada |
| Atualização fiscal automática | ✅ | ✅ | N/A |
| Sandbox gratuito | ✅ | ✅ | N/A |
| Preço público | ❌ | ❌ | ✅ |
| Modelo de cobrança | Contrato anual | Volume/contrato | Mensal |
| Perfil principal | Custo previsível | Escalabilidade operacional | Apoio tributário |

---

# 6. Limitações Comuns de APIs Fiscais

Independentemente do fornecedor escolhido, existem limitações inerentes ao ecossistema fiscal brasileiro.

## Dependência da SEFAZ e Prefeituras

Nenhuma API fiscal controla:
- disponibilidade da SEFAZ;
- estabilidade de provedores municipais;
- lentidão de autorização;
- indisponibilidade de webservices governamentais.

## Oscilações municipais

NFS-e possui grande fragmentação nacional:
- diferentes layouts;
- diferentes provedores;
- diferentes regras tributárias;
- diferentes tempos de resposta.

## Necessidade de contingência

Mesmo em arquiteturas modernas SaaS:
- contingência continua necessária;
- retransmissões continuam obrigatórias;
- filas assíncronas continuam essenciais.

## Tributação continua complexa

A API resolve transmissão.

A responsabilidade tributária da emissão continua dependendo:
- do ERP;
- da parametrização fiscal;
- das regras de negócio;
- da legislação vigente.

---

# Combinações Recomendadas

## Cenário focado em previsibilidade financeira

```text
ACBr API
+
SAC Fiscal
```

Indicado para:
- software houses em crescimento;
- operações com orçamento previsível;
- equipes pequenas;
- redução de manutenção local.

---

## Cenário focado em escalabilidade operacional

```text
PlugNotas
+
SAC Fiscal
```

Indicado para:
- alto volume de emissões;
- forte dependência de NFS-e;
- múltiplos municípios;
- necessidade de White Label;
- operações críticas.

---

# Fontes Consultadas

- acbr.api.br
- projetoacbr.com.br
- plugnotas.com.br
- tecnospeed.com.br
- sacfiscal.com.br

> Consultado em maio/2026.
> Informações comerciais podem sofrer alterações sem aviso prévio.
