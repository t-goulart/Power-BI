# 📊 Power BI Portfolio - Business Intelligence & Analytics

Bem-vindo ao meu portfólio de Power BI. Aqui apresento soluções que unem **Engenharia de Dados (M)**, **Modelagem Performática** e **Storytelling** para suporte à tomada de decisão.

---

## 🏆 Projeto em Destaque: Dashboard Financeiro Autoadaptável
*Uma solução resiliente ao Schema Drift com ingestão dinâmica de dados.*

<p align="center">
  <video src="https://github.com/user-attachments/assets/36cd1130-aa49-4e65-9802-b6a5689e6958" width="100%" autoplay loop muted playsinline>
    Seu navegador não suporta a tag de vídeo.
  </video>
</p>

**🚀 Diferenciais Técnicos:**
* **ETL Dinâmico:** Código em M que identifica e expande novas colunas automaticamente.
* **UX/UI:** Design focado em usabilidade com navegação por menus e filtros inteligentes.
* **Link Interativo:** [Acesse o Dashboard aqui](https://app.powerbi.com/view?r=eyJrIjoiNTZhZjEwNjgtOWVmNC00MWEyLWJhMDQtOTZhZDZjYjMzNTU1IiwidCI6IjVlYzdjNDViLWQxYzgtNGIwNS1hMmVjLTJjYzE1OWI5ZDZiMiJ9)

---

## 🧠 Guia de Arquitetura e Melhores Práticas
*Este repositório não é apenas um portfólio visual, mas um guia de implementação técnica. Abaixo, as premissas de performance utilizadas:*

### 📌 Índice de Navegação Técnica
1. [Otimização de Chaves (IDs)](#-otimização-de-chaves-ids)
2. [Preservação do Query Folding](#-preservação-do-query-folding)
3. [Enriquecimento de Dados (NestedJoin vs. LOOKUPVALUE)](#-enriquecimento-de-dados)
4. [🆘 Guia de Troubleshooting (Solução de Problemas)](./Troubleshooting.md)

### 🔹 Otimização de Chaves (IDs)
Uso de **Int64 (Inteiro)** em chaves de relacionamento para maximizar a compressão do motor **VertiPaq** e reduzir o consumo de memória RAM.

### 🔹 Preservação do Query Folding
Garantia de que as transformações pesadas sejam processadas na fonte (SQL/DB), evitando o tráfego desnecessário de linhas brutas.

### 🔹 Enriquecimento de Dados
Substituição de `LOOKUPVALUE` e colunas calculadas por **Table.NestedJoin**. Isso garante integridade (evita erros de MAX/MIN) e melhora drasticamente a performance do modelo.

## ⛓️ Relacionamentos e Fluxo de Filtro
A integridade do modelo depende de um fluxo de filtro previsível.

* **Direção Única vs. Ambos:** Evite o uso de "Direção do filtro cruzado: Ambos" em relacionamentos 1:*. 
* **Riscos:** Filtros bi-direcionais podem causar propagação indesejada da tabela Fato para a Dimensão, gerando resultados incorretos em medidas e ambiguidade no esquema.
* **Solução:** Utilize a função DAX `CROSSFILTER` para habilitar a bi-direcionalidade apenas em medidas específicas, preservando a performance e a segurança do modelo global.

⛓️ O Perigo do Relacionamento Muitos para Muitos (*:*)
Relacionamentos Muitos para Muitos introduzem ambiguidade e riscos de integridade.

* **Impacto Técnico:** Filtros ineficientes, métricas duplicadas e perda de performance no Motor VertiPaq.
* **Alternativas Profissionais:**
    * **Hierarquia de Fluxo:** Validar se a Dimensão (1) está corretamente posicionada acima da Fato (*).
    * **Desnormalização:** Em cenários específicos, consolidar dados em uma tabela única para aproveitar a compressão colunar do VertiPaq.
    * **Tabelas Ponte:** Criar tabelas intermediárias para normalizar a relação em 1:*.

---

## 📂 Organização do Repositório

* **[Medidas DAX](./Medidas):** Biblioteca de fórmulas (Time Intelligence e Projeções).
* **[Projetos/Autoadaptavel](./Projetos/Autoadaptavel):** Arquitetura resiliente a mudanças de fonte.
* **[Filtros](./Filtros):** Lógicas de segmentação dinâmica.

---

## 🛠️ Stack Tecnológica
* **Power BI:** Desktop & Service.
* **Linguagem M:** ETL avançado e dinâmico.
* **DAX:** Star Schema e medidas complexas.
* **Integração:** SQL, Excel, Python e SharePoint.

---
**Desenvolvido por [Tiago Almeida Goulart](https://www.linkedin.com/in/tiago-agoulart)**
