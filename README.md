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
↔️ A integridade do modelo depende de um fluxo de filtro previsível.

* **Direção Única vs. Ambos:** Evite o uso de "Direção do filtro cruzado: Ambos" em relacionamentos 1:*. 
* **Riscos:** Filtros bi-direcionais podem causar propagação indesejada da tabela Fato para a Dimensão, gerando resultados incorretos em medidas e ambiguidade no esquema.
* **Solução:** Utilize a função DAX `CROSSFILTER` para habilitar a bi-direcionalidade apenas em medidas específicas, preservando a performance e a segurança do modelo global.

💣 O Perigo do Relacionamento Muitos para Muitos (*:*)
Relacionamentos Muitos para Muitos introduzem ambiguidade e riscos de integridade.

* **Impacto Técnico:** Filtros ineficientes, métricas duplicadas e perda de performance no Motor VertiPaq.
* **Alternativas Profissionais:**
    * **Hierarquia de Fluxo:** Validar se a Dimensão (1) está corretamente posicionada acima da Fato (*).
    * **Desnormalização:** Em cenários específicos, consolidar dados em uma tabela única para aproveitar a compressão colunar do VertiPaq.
    * **Tabelas Ponte:** Criar tabelas intermediárias para normalizar a relação em 1:*.


# Implementação de RLS Hierárquico com a Função PATH no Power BI

> **Guia de Arquitetura, Governança e Engenharia de Dados para Controle de Acesso Dinâmico.**

## 00. Visão Geral do Projeto

Este repositório contém a documentação técnica e os scripts para a implementação de **RLS (Row-Level Security) Hierárquico** no Power BI. A solução garante que cada usuário logado visualize apenas os dados da sua própria estrutura de liderança (e das equipes abaixo dele), além de tratar exceções de segurança corporativa (RH, Diretoria, Auditoria) sem a necessidade de replicar arquivos `.pbix` ou alterar regras de código manualmente.

### Públicos Atendidos
* **Gestores e Auditoria:** Entendimento conceitual, matriz de segurança, LGPD e governança.
* **Engenheiros e Analistas de Dados:** Implementação prática via Linguagem M, colunas calculadas e DAX avançado.

## 01. Conceito e Arquitetura

### O Problema Operacional
A tentativa tradicional de criar múltiplos arquivos para diferentes áreas gera redundância de manutenção e riscos severos de vazamento de informações. A arquitetura correta baseia-se em **um modelo semântico único** que se adapta dinamicamente à credencial de quem o acessa.

### A Analogia do Crachá
O RLS funciona como o crachá de acesso corporativo: o relatório (prédio) é o mesmo para todos, mas a credencial do usuário define quais andares (linhas de dados) serão liberados. A validação ocorre diretamente no motor do Power BI (VertiPaq) antes do cálculo e renderização dos visuais.

### O Desafio da Hierarquia e a Função `PATH`
Hierarquias possuem profundidades variáveis. A função `PATH` em DAX percorre a cadeia pai/filho e retorna uma string delimitada por pipes (`|`) que mapeia toda a árvore organizacional:

| Cracha | Nome | Cracha da Gestao Imediata | Coluna Calculada RLS (Resultado PATH) |
| :--- | :--- | :--- | :--- |
| **1000** | Ana | *(vazio - topo)* | `1000` |
| **1010** | Bruno | `1000` | `1000|1010` |
| **1011** | Carla | `1010` | `1000|1010|1011` |
| **1012** | Diego | `1011` | `1000|1010|1011|1012` |

Com o caminho gerado, a pergunta *"O Bruno (1010) pode ver o Diego (1012)?"* é resolvida pela função `PATHCONTAINS`, verificando a existência da substring dentro do caminho do colaborador.

## 02. Fluxo da Regra de Segurança

Ao abrir o relatório, a função de segurança executa as seguintes etapas sequenciais:

graph TD
    A[Usuário abre o relatório] --> B[Obtém e-mail via USERPRINCIPALNAME]
    B --> C{Está na tabela desconectada dRLS?}
    C -- Sim --> D[Acesso Total Liberado TRUE]
    C -- Não --> E{Possui cadastro ativo na tPessoas?}
    E -- Não --> F[Acesso Negado FALSE - Painel em Branco]
    E -- Sim --> G{Possui Lider = 'Sim'?}
    G -- Não --> F
    G -- Sim --> H[Aplica PATHCONTAINS na coluna RLS]
    H --> I[Renderiza apenas a própria estrutura]

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
