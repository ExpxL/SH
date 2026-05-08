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

---

## 2. TecnoSpeed (PlugNotas) — API + Consultoria

API REST SaaS madura, com mais de 18 anos no mercado fiscal.

**Integração:** JSON via HTTP — compatível com PHP, C#, Java, Python, Delphi e qualquer linguagem com HTTP.

**Documentos suportados:** NF-e, NFC-e, NFS-e (+1.000 prefeituras), CT-e, MDF-e, NFCom, DC-e.

**Diferenciais:**
- Auditor online com validações técnicas e tributárias
- Gestão de certificados A1 e A3
- White Label para a software house
- Envio automático de XML por e-mail ao destinatário
- Armazenamento em nuvem pelo período legal (11 anos — Ajuste SINIEF nº 2/2025)
- Fura Fila para priorizar autorização na prefeitura (NFS-e)

**Consultoria fiscal:** Equipe de consultores técnicos e tributários disponível desde o onboarding, com suporte especializado em legislação fiscal para NF-e, NFC-e, NFS-e, CT-e e demais documentos.

---

## 3. SAC Fiscal — Assessoria Fiscal Especializada (sem emissão)

O SAC Fiscal **não emite notas** — é um serviço puro de **consultoria e suporte fiscal técnico** para software houses:

- Suporte em regras fiscais e tributação (ICMS, ICMS-ST, PIS, COFINS, IPI) para todos os documentos
- Plano exclusivo de dúvidas fiscais **ou** combo por tecnologia: **Delphi+ACBr**, **C#+ZeusDFe** ou **PHP+SpedNFe**
- Atendimento via chamados respondidos em até 72h
- Conteúdo técnico com planilhas, modelos de dados e exemplos em Delphi, C#, PHP e JavaScript
- Encontros ao vivo sobre Reforma Tributária
- Assinatura anual com 2 acessos inclusos

---

## 4. Resumo Comparativo

| Critério | ACBr API | TecnoSpeed PlugNotas | SAC Fiscal |
|---|:---:|:---:|:---:|
| **Tipo de solução** | API REST SaaS | API REST SaaS | Consultoria pura |
| **Integração PHP** | ✅ HTTP/JSON | ✅ HTTP/JSON | ✅ plano PHP |
| **Emissão Fiscal Completa** | ✅ | ✅ | ❌ |
| **NFS-e multimunicipal** | ✅ | ✅ (+1.000) | N/A |
| **Consultoria tributária** | ✅ | ✅ | ✅ especializado |
| **Suporte fiscal para PHP** | ✅ | ✅ | ✅ nativo |
| **Atualização fiscal automática** | ✅ | ✅ | N/A |
| **White Label** | ❌ | ✅ | N/A |
| **Modelo de cobrança** | Anual s/ franquia | Por volume/contrato | Assinatura anual |

---

## 5. Quando Usar Cada Um?

| Cenário | Recomendação |
|---|---|
| PHP ou qualquer linguagem, quer API SaaS sem manutenção | **ACBr API** ou **TecnoSpeed PlugNotas** |
| Precisa de White Label e NFS-e em mais de 1.000 municípios | **TecnoSpeed PlugNotas** |
| Quer API com modelo anual sem franquia por documento | **ACBr API** |
| Precisa só de suporte fiscal e tributário especializado | **SAC Fiscal** |
| Quer consultoria fiscal + API em uma única contratação | **TecnoSpeed PlugNotas** ou **ACBr API + SAC Fiscal** |

---

*Fontes: acbr.api.br · tecnospeed.com.br · sacfiscal.com.br — Consultado em maio/2026*
