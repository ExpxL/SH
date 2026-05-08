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

| Categoria | Recursos |
|---|---|
| Emissão fiscal | NF-e, NFC-e, NFS-e, CT-e, MDF-e, NFCom, DC-e |
| Eventos fiscais | Cancelamento, consulta e Carta de Correção (CC-e) |
| Distribuição DF-e | Download de XMLs vinculados ao CNPJ |
| Consultas | CNPJ, CPF, CEP e CNAE |
| Armazenamento | Guarda de XMLs pelo período legal |
| Integração | API REST via HTTP/JSON |
| Compatibilidade | PHP, C#, Delphi, Java, Python, Node.js e demais linguagens HTTP |

## Características técnicas

| Categoria | Descrição |
|---|---|
| Arquitetura | SaaS |
| Integração | Desacoplada via REST |
| Atualizações fiscais | Centralizadas pela plataforma |
| Manutenção local | Reduzida |
| Gestão de schemas | Automatizada |

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

| Categoria | Recursos |
|---|---|
| Emissão fiscal | NF-e, NFC-e, NFS-e, CT-e, MDF-e, NFCom, DC-e |
| Comunicação | Webhooks de acompanhamento |
| Distribuição | Envio automático de XML/PDF |
| Certificados digitais | Gestão de certificados A1 |
| White Label | Disponível |
| Armazenamento | Guarda de documentos em nuvem |
| Integração | API REST HTTP/JSON |
| Compatibilidade | PHP, Java, Delphi, C#, Python, Node.js e demais linguagens HTTP |

## Diferenciais operacionais

| Recurso | Descrição |
|---|---|
| Contingência NFC-e | Mecanismos próprios de continuidade operacional |
| Retransmissão automática | Reenvio automático após indisponibilidade |
| Filas assíncronas | Processamento desacoplado |
| Fura Fila | Priorização operacional para determinadas emissões de NFS-e |
| Cobertura municipal | Ampla cobertura multimunicipal |

## Características técnicas

| Categoria | Descrição |
|---|---|
| Arquitetura | SaaS distribuída |
| Processamento | Assíncrono |
| Infraestrutura | Uso intensivo de filas e workers |
| Integração | REST desacoplado |
| Escalabilidade | Voltada a alto volume operacional |

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

## Principais funcionalidades

| Categoria | Recursos |
|---|---|
| Suporte tributário | ICMS, ICMS-ST, PIS, COFINS, IPI, IBS e CBS |
| Reforma Tributária | Workshops e treinamentos |
| APIs auxiliares | Dados fiscais complementares |
| Atualizações regulatórias | Boletins e comunicados técnicos |
| Conteúdo técnico | Modelos, planilhas e exemplos de integração |
| Público-alvo | Software houses, ERPs e equipes tributárias |

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

| Categoria | Limitação |
|---|---|
| Dependência governamental | Nenhuma API controla disponibilidade da SEFAZ ou prefeituras |
| NFS-e | Grande fragmentação entre municípios e provedores |
| Contingência | Continua necessária mesmo em arquiteturas SaaS |
| Retransmissão | Pode ser obrigatória após instabilidade |
| Tributação | Continua dependendo das regras do ERP e parametrização fiscal |
| Oscilações externas | Timeouts e lentidão podem ocorrer independentemente da plataforma |

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

> Fontes: acbr.api.br | projetoacbr.com.br | plugnotas.com.br | tecnospeed.com.br | sacfiscal.com.br
