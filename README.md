# ACBr, ACBr API, TecnoSpeed e SAC Fiscal
> Guia comparativo para Software Houses — integração fiscal com PHP

---

## 1. ACBr + PHP: Formas de Integração

O ACBr oferece **três formas** de integração com PHP:

- **ACBrLib via FFI** — chama as bibliotecas nativas do ACBr diretamente no PHP usando Foreign Function Interface. Requer habilitar o FFI no ambiente e seguir os métodos da ACBrLib.
- **ACBrMonitorPLUS via TXT ou TCP/IP** — a aplicação PHP escreve comandos em arquivos `.txt` num diretório monitorado; o ACBrMonitor processa e devolve o retorno. Via TCP/IP o fluxo é o mesmo, só que por socket em tempo real.
- **ACBrLib com Wrappers de Alto Nível** — binários assinados digitalmente com classes de alto nível prontas para consumo em várias linguagens, incluindo PHP.

---

## 2. Funcionalidades do ACBr (2025)

| Categoria | Recursos |
|---|---|
| **Documentos Fiscais** | NF-e, NFC-e, NFS-e, CT-e, MDF-e, NFCom, DC-e, SAT/MF-e |
| **Financeiro** | Boletos bancários com registro online, Pix (criação/cancelamento/consulta) |
| **Meios de Pagamento** | TEF (CliSiTef, PayGo, ACBrTEFAPI), Pix, SmartPOS |
| **Hardware** | Impressoras, ECF, balanças, leitores, gavetas, PDV Android |
| **Obrigações Acessórias** | e-Social, EFD Reinf, SPED |
| **Consultas** | CNPJ, CPF, CEP, listagem por CNAE |
| **Novidades** | Integrações bancárias por API, extratos, IA em ERP |

---

## 3. ACBr API — API REST Oficial do Projeto ACBr

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

## 4. ACBr tem Suporte de Assessoria Fiscal/Tributária?

**Sim — via ACBr Pro e SAC Fiscal:**

### ACBr Pro
Assinatura exclusiva para software houses que inclui:
- ACBrMonitorPLUS, ACBrLib e Fontes com atualizações semanais assinadas
- Sem cobrança adicional por CNPJ ou terminal
- Cursos exclusivos (incluindo o de PHP+FFI)
- Suporte e consultoria direta com a equipe ACBr via fórum e Discord

### SAC Fiscal *(serviço independente, parceiro do ACBr)*
Primeiro e único suporte fiscal **multidisciplinar** do Brasil para software houses:
- Dúvidas sobre legislação, notas técnicas, regras de validação e tributação (ICMS, ICMS-ST, PIS, COFINS, IPI)
- Plano exclusivo de dúvidas fiscais **ou** combo por tecnologia: **Delphi+ACBr**, **C#+ZeusDFe** ou **PHP+SpedNFe**
- Chamados respondidos em até 72h
- Conteúdo estruturado com planilhas, exemplos práticos em Delphi, C#, PHP e JavaScript
- Encontros ao vivo sobre Reforma Tributária
- Assinatura anual com 2 acessos inclusos

---

## 5. TecnoSpeed (PlugNotas) — API + Consultoria

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

## 6. SAC Fiscal — Somente Assessoria (sem emissão)

O SAC Fiscal **não emite notas** — é um serviço puro de **consultoria e suporte fiscal técnico** para software houses:

- Suporte nas regras fiscais e tributação para todos os documentos
- Atendimento via chamados (até 72h de resposta)
- Conteúdo técnico com modelos de dados e exemplos de código
- Encontros ao vivo sobre Reforma Tributária
- Assinatura anual, 2 acessos inclusos

---

## 7. Resumo Comparativo

| Critério | ACBr Lib/Monitor | ACBr API | TecnoSpeed PlugNotas | SAC Fiscal |
|---|:---:|:---:|:---:|:---:|
| **Tipo de solução** | Componente local | API REST SaaS | API REST SaaS | Consultoria pura |
| **Integração PHP** | ✅ FFI ou TCP | ✅ HTTP/JSON | ✅ HTTP/JSON | ✅ plano PHP |
| **Self-hosted** | ✅ | ❌ | ❌ | N/A |
| **Emissão Fiscal Completa** | ✅ | ✅ | ✅ | ❌ |
| **NFS-e multimunicipal** | ✅ | ✅ | ✅ (+1.000) | N/A |
| **Boleto / TEF / Pix** | ✅ | Parcial | ❌ | N/A |
| **Hardware (ECF, balança)** | ✅ | ❌ | ❌ | N/A |
| **Consultoria tributária** | ✅ (ACBr Pro) | ✅ | ✅ | ✅ especializado |
| **Suporte fiscal para PHP** | ✅ (via SAC Fiscal) | ✅ | ✅ | ✅ nativo |
| **Atualização fiscal automática** | Manual (ou Pro) | ✅ | ✅ | N/A |
| **Modelo de cobrança** | Assinatura anual | Anual s/ franquia | Por volume/contrato | Assinatura anual |
| **Open Source (base)** | ✅ LGPL | ❌ | ❌ | ❌ |

---

## 8. Quando Usar Cada Um?

| Cenário | Recomendação |
|---|---|
| Sistema Delphi/Lazarus já existente | **ACBr Fontes + ACBr Pro** |
| PHP ou qualquer linguagem, quer rodar local | **ACBrLib via FFI + ACBr Pro** |
| PHP ou qualquer linguagem, quer API SaaS sem manutenção | **ACBr API** ou **TecnoSpeed PlugNotas** |
| Precisa de TEF, hardware e fiscal no mesmo componente | **ACBrMonitorPLUS** |
| Precisa só tirar dúvidas fiscais e tributárias | **SAC Fiscal** |
| Quer consultoria fiscal + API madura e com histórico | **TecnoSpeed PlugNotas** |

---

*Fontes: projetoacbr.com.br · acbr.api.br · tecnospeed.com.br · sacfiscal.com.br — Consultado em maio/2026*
