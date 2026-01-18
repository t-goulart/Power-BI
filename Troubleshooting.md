# 🛠️ Power BI Troubleshooting Guide

Este guia compila soluções para erros comuns de ambiente, interface e performance enfrentados durante o desenvolvimento no Power BI Desktop e Power Query.

---

## ⌨️ 1. IntelliSense (Autopreenchimento) não funciona
Se o Power Query parou de sugerir nomes de funções, colunas ou parâmetros, siga esta ordem de resolução:

### Nível 1: Configurações de Interface
1. Vá em **Arquivo** > **Opções e Configurações** > **Opções**.
2. Na seção **Global**, selecione **Editor de Consulta**.
3. Certifique-se de que a opção **"Habilitar IntelliSense"** está marcada. 
   * *Dica: Se já estiver marcada, desmarque, clique em OK, e marque-a novamente para resetar o serviço.*

### Nível 2: Limpeza de Cache de Metadados
1. Vá em **Arquivo** > **Opções e Configurações** > **Opções**.
2. Na guia **Global**, selecione **Carregamento de Dados**.
3. Em **Opções de Cache de Gerenciamento de Dados**, clique no botão **Limpar Cache**.

### Nível 3: O "Pulo do Gato" (Microsoft Store)
Caso as opções acima falhem, abra a **Microsoft Store**, pesquise por **Power BI Desktop** e clique no botão **Abrir** diretamente pela loja. Este processo força a revalidação de componentes do sistema que o atalho padrão do Windows pode ignorar.

---

## 🔒 2. Erros de Níveis de Privacidade
Frequentemente ocorre ao tentar combinar dados de fontes diferentes (ex: SQL + Excel Local).

* **Sintoma:** "Information is needed regarding data privacy".
* **Solução:** 1. Vá em **Arquivo** > **Opções e Configurações** > **Configurações da Fonte de Dados**.
  2. Clique em **Editar Permissões** e defina o Nível de Privacidade para **Organizacional** ou **Público**.
  3. Em casos de desenvolvimento isolado, você pode marcar "Ignorar os Níveis de Privacidade" em **Opções** > **Privacidade**, mas tenha cautela com a segurança dos dados.

---

## ⚡ 3. Lentidão na Visualização e Processamento (Enriquecimento de Dados)
Muitos problemas de performance surgem ao tentar "cruzar" tabelas de forma ineficiente.

### Problema: Uso excessivo de DAX para buscas (LOOKUPVALUE/CALCULATE)
O uso de `LOOKUPVALUE` ou `CALCULATE` com `MAX/MIN` para buscar dados de outras tabelas cria colunas calculadas que não comprimem bem e aumentam o consumo de memória RAM. Além disso, funções de agregação podem mascarar duplicidades nos dados.

### Solução: Table.NestedJoin no Power Query
Sempre que possível, realize o enriquecimento dos dados no ETL usando `Table.NestedJoin`.
* **Vantagem:** O motor **VertiPaq** comprime o resultado final, reduzindo o tamanho do arquivo.
* **Integridade:** Permite identificar duplicidades (expansão de linhas) ainda na fase de tratamento.
* **Atenção:** A ordem das colunas nas chaves de comparação `{ }` deve ser idêntica em ambas as tabelas para evitar valores nulos.

### Preview Lento no Power Query
* Utilize `Table.FirstN(fonte, 10)` logo após a navegação para trabalhar com uma amostra durante o desenvolvimento, removendo a etapa antes da publicação.

---

## 🔄 4. Erros de Memória ou Timeout na Atualização
* **Causa:** Frequentemente causado por tipos de dados inadequados (Chaves como Texto) ou quebra do **Query Folding**.
* **Solução:** Verifique se as chaves de relacionamento são do tipo **Int64.Type** (Inteiro) e evite transformações que impeçam o banco de dados de processar a consulta nativamente.


## 🎨 5. Design e Experiência do Usuário (UX)
A interface é o "contrato" final entre o dado processado e o tomador de decisão. Um design ineficiente quebra a confiança no dado.

* **Carga Cognitiva:** Evite o "efeito cena no escuro" — excesso de cores escuras e baixo contraste frustram o usuário e impedem a rápida absorção dos insights.
* **Acessibilidade e Contraste:** Mantenha alto contraste entre o fundo e os rótulos de dados. O uso estratégico do cinza permite que as cores de destaque (vermelho/verde) cumpram seu papel sem poluir o visual.
* **Semântica das Cores:** A cor deve comunicar, não decorar. Utilize o vermelho para atenção/desvios e o verde para metas, mantendo a consistência em todo o projeto.

### 📚 Leituras Recomendadas para Storytelling:
* **Storytelling com Dados** (Cole Nussbaumer Knaflic): Foco em eliminar a desordem visual.
* **Show Me the Numbers** (Stephen Few): A ciência por trás da percepção visual e gráficos.

---
**Documentação mantida por [Tiago Almeida Goulart](https://www.linkedin.com/in/tiago-agoulart)**
