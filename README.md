<p align="center">
  <img src="imagens/logo.png" alt="Max PDV" width="320">
</p>

# Max PDV: Sistema de Ponto de Venda

> **Código-fonte privado.** Este repositório é uma vitrine do projeto: funcionalidades,
> arquitetura e telas. O código pode ser apresentado sob solicitação.

Sistema de ponto de venda completo para o varejo, **em produção** em um comércio com
catálogo de ~600 produtos. Cobre o dia inteiro da loja: abertura de caixa, venda no
balcão, emissão de **NFC-e** direto na SEFAZ, impressão térmica, estoque, entrada de
notas de fornecedores e fechamento com conferência.

Distribuído como **um único `.exe`**: a pessoa da loja dá duplo clique e o sistema
se instala sozinho, com banco de dados embutido e sem depender de internet para vender.

![Python](https://img.shields.io/badge/Python_3.12-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React_18-20232A?logo=react&logoColor=61DAFB)
![Ant Design](https://img.shields.io/badge/Ant_Design_5-0170FE?logo=antdesign&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white)
![NFC-e](https://img.shields.io/badge/NFC--e-SEFAZ_4.00-orange)

---

## Em números

| | |
| --- | --- |
| ~99 endpoints de API | FastAPI assíncrono |
| ~15 mil linhas de código | 6 mil em Python + 9 mil em React |
| ~600 produtos | com classificação fiscal (NCM, CEST, CFOP, CSOSN) |
| 1 executável | instalação automática com MongoDB portátil |

---

## Telas

> Todas as telas usam dados de demonstração: loja, usuários e vendas fictícios.

### Dashboard
<p align="center">
  <img src="imagens/dashboard.png" width="900"><br>
  <sub>Caixa aberto, faturamento do dia, alertas de estoque, resumo por forma de pagamento e vendas recentes</sub>
</p>

### Venda no balcão
<p align="center">
  <img src="imagens/pdv.png" width="900"><br>
  <sub>Grade por categoria, busca por nome ou código de barras, carrinho e pedidos em espera</sub>
</p>

<table>
  <tr>
    <td align="center"><img src="imagens/pagamento.png" width="260"><br><sub>Pagamento com troco e emissão de NFC-e</sub></td>
    <td align="center"><img src="imagens/pedido-espera.png" width="300"><br><sub>Pedido em espera por cliente ou mesa</sub></td>
    <td align="center"><img src="imagens/login.png" width="300"><br><sub>Acesso por usuário</sub></td>
  </tr>
</table>

### Caixa
<p align="center">
  <img src="imagens/caixa.png" width="900"><br>
  <sub>Sessão, saldo inicial, resumo financeiro por forma de pagamento, sangria e movimentos</sub>
</p>

<table>
  <tr>
    <td align="center"><img src="imagens/abertura-caixa.png" width="420"><br><sub>Abertura com saldo em dinheiro e maquininha</sub></td>
    <td align="center"><img src="imagens/fechar-caixa.png" width="300"><br><sub>Fechamento com conferência esperado × real</sub></td>
  </tr>
</table>

### Produtos e estoque
<table>
  <tr>
    <td align="center"><img src="imagens/produtos.png" width="440"><br><sub>Catálogo com código de barras, preço, estoque e mínimo</sub></td>
    <td align="center"><img src="imagens/novo-produto.png" width="260"><br><sub>Cadastro com dados fiscais (NCM, CEST, CFOP, CSOSN)</sub></td>
  </tr>
  <tr>
    <td align="center"><img src="imagens/estoque.png" width="440"><br><sub>Visão geral: abaixo do mínimo, zerados e ações rápidas</sub></td>
    <td align="center"><img src="imagens/estoque-avancado.png" width="440"><br><sub>Controle avançado: alertas, simulação, preditiva, NF-e e compras</sub></td>
  </tr>
</table>

<p align="center">
  <img src="imagens/entrada-nfe.png" width="820"><br>
  <sub>Entrada de mercadoria por NF-e: busca na SEFAZ, chave de acesso ou arquivo XML</sub>
</p>

### Gestão, fiscal e acessos
<p align="center">
  <img src="imagens/relatorio-estoque.png" width="900"><br>
  <sub>Relatórios gerencial, de vendas, financeiro, de estoque e de maquininhas, com exportação</sub>
</p>

<table>
  <tr>
    <td align="center"><img src="imagens/fiscal.png" width="440"><br><sub>Configuração fiscal: empresa, certificado, e-mail, notas e impressora</sub></td>
    <td align="center"><img src="imagens/controle-acessos.png" width="440"><br><sub>Permissões por usuário e por tela</sub></td>
  </tr>
</table>

---

## Arquitetura

```mermaid
flowchart LR
    subgraph Loja["Computador da loja (PDV.exe)"]
        UI[React + Ant Design] --> API[FastAPI]
        API --> DB[(MongoDB portátil)]
        API --> IMP[Impressora térmica<br/>ESC/POS]
        API --> CERT[Certificado A1]
    end

    API -- NFC-e assinada<br/>SOAP --> SEFAZ[SEFAZ<br/>MG · SP · RJ]
    API -. alternativa .-> FOCUS[Focus NFe]
    API -- Distribuição DFe --> NAC[SEFAZ Nacional<br/>notas de fornecedores]
    API -- relatório de fechamento --> MAIL[E-mail]
```

---

## Funcionalidades

### Venda no balcão
- Grade de mais vendidos, filtro por categoria e busca por **código de barras**.
- Carrinho com desconto, **várias formas de pagamento** (dinheiro, PIX, crédito, débito) e troco.
- **Pedidos em espera** por nome do cliente/mesa, para atender vários ao mesmo tempo.
- CPF/CNPJ na nota, quando o cliente pede.

### Fiscal (NFC-e modelo 65, layout 4.00)
- **Emissão direta na SEFAZ** via SOAP, com XML montado e **assinado digitalmente** com o
  certificado e-CNPJ A1 (lxml + signxml), ou por **API externa** (Focus NFe), configurável.
- Suporte a homologação e produção, com URLs por estado (MG, SP, RJ).
- Cancelamento de nota, DANFE com **QR Code** e controle de custo por nota emitida.
- **Entrada de NF-e de fornecedores** por XML, chave de acesso ou download automático via
  **Distribuição DFe** da SEFAZ Nacional.

### Caixa
- Abertura com saldo inicial (dinheiro e maquininha), sangrias e suprimentos.
- **Fechamento com conferência** do valor esperado × real por forma de pagamento, com
  justificativa obrigatória em caso de diferença.
- Relatório de fechamento enviado **por e-mail** automaticamente.
- Taxas de maquininha por bandeira, para o resultado líquido real.

### Estoque
- Movimentações individuais e **em lote**, com histórico.
- Alertas de estoque mínimo, **simulação** e visão **preditiva** de reposição.
- Sugestão de compras a partir do giro.

### Gestão
- Dashboard com indicadores do dia e do mês (Chart.js).
- Relatórios gerenciais de vendas, estoque e financeiro.
- **Controle de acesso** por usuário, com matriz de permissões por tela.

### Engenharia
- **Impressão térmica ESC/POS sem biblioteca de impressão**: os bytes do protocolo são
  montados à mão, e o QR Code é rasterizado direto no formato da impressora, sem Pillow.
- **Auto-instalação**: na primeira execução o `.exe` gera uma chave JWT única por loja,
  sobe o MongoDB portátil (e instala o runtime C++ se faltar) e restaura o banco inicial.
  Nas seguintes, só reaproveita o que já existe.
- Ferramentas de apoio ao contador: planilhas de **validação fiscal** geradas direto do
  banco, agrupadas por NCM + CSOSN + CEST, para validar dezenas de grupos em vez de centenas
  de linhas.
- Autenticação JWT com senhas em **bcrypt**.

---

## Stack

| Camada | Tecnologias |
| --- | --- |
| **Back-end** | Python 3.12, FastAPI, Uvicorn, Pydantic 2, Motor (MongoDB assíncrono), PyJWT, bcrypt |
| **Fiscal** | lxml, signxml, cryptography/pyOpenSSL (certificado A1), zeep (SOAP), ReportLab, qrcode |
| **Front-end** | React 18, Ant Design 5, Chart.js, React Router, Axios |
| **Distribuição** | PyInstaller (executável único), MongoDB portátil, instalação automática |
| **Hardware** | Impressoras térmicas ESC/POS (Epson TM-T20X e compatíveis) |

---

## Contato

Desenvolvido por **Halyson Henrique**. Quer ver o código ou uma demonstração?
Entre em contato pelo [LinkedIn](https://www.linkedin.com/) ou pelo [GitHub](https://github.com/halysonhenrique).
