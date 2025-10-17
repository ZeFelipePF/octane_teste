## **Sistema de Pesquisa de Tendências para YouTube com n8n e IA**

Este projeto é a minha solução para o desafio **"Sistema de Pesquisa de Tendências para YouTube"**. O objetivo é criar um sistema automatizado que, a partir de um nicho/sub-nicho, utilize dados reais do YouTube para identificar padrões, analisar a concorrência e gerar 15 ideias de vídeos com alto potencial de viralização, completas com justificativas baseadas em dados.

### **Visão Geral da Solução**

O sistema foi construído inteiramente dentro do **n8n**, orquestrando uma série de etapas que vão desde a coleta de dados brutos até a análise sofisticada por um Large Language Model (LLM). A solução é robusta, modular e 100% automatizada, transformando um único input (o nicho) em um output rico e acionável.

---

## **1\. Planejamento e Pesquisa**

### **1.1. Documentação de Concepção**

A arquitetura do workflow foi projetada para seguir um funil lógico, onde os dados são progressivamente coletados, enriquecidos, analisados e, finalmente, sintetizados em insights criativos.

A estrutura pode ser dividida em 7 fases principais:

1. **Fase 1: Coleta de Dados Brutos:** O processo inicia com uma busca ampla na API do YouTube usando o nicho fornecido (ex: "weak legs"). O objetivo é coletar uma amostra inicial de até 25 vídeos que são mais relevantes por contagem de visualizações, nos dando um primeiro panorama do que faz sucesso.  
2. **Fase 2: Enriquecimento de Dados:** A lista inicial de vídeos é insuficiente. Cada vídeo da amostra é então processado individualmente para buscar dados detalhados, como estatísticas completas (likes, comentários), e também para buscar informações sobre os canais que os publicaram (número de inscritos, total de vídeos). Isso transforma uma simples lista de IDs de vídeo em um dataset rico.  
3. **Fase 3: Análise Estatística (Outlier Detection):** Esta é uma etapa crucial. Um script de código personalizado calcula estatísticas descritivas para as métricas chave (views, likes, comments, inscritos) e utiliza o método **Z-Score** para identificar *outliers*—vídeos que performam significativamente acima da média. Além disso, um **Score de Engajamento** ponderado é calculado para cada vídeo. Vídeos com alto Z-Score ou alto engajamento são classificados como "high performers".  
4. **Fase 4: Análise de Títulos e Padrões:** Com os "high performers" identificados, um segundo script analisa exclusivamente os títulos desses vídeos de sucesso. Ele extrai as palavras-chave mais frequentes e identifica padrões estruturais (uso de números, emojis, perguntas, gatilhos emocionais como "segredo", "rápido", "erro fatal").  
5. **Fase 5: Análise de Concorrência:** Para entender o quão saturado é o nicho, o sistema extrai os padrões e palavras-chave de sucesso e os utiliza para fazer novas buscas no YouTube. Um script calcula um **Score de Concorrência** para cada palavra-chave, analisando o volume de resultados e o engajamento médio dos vídeos concorrentes. Isso nos permite identificar "oceanos azuis": temas com alto interesse (baseado nos *high performers*) mas com baixa concorrência.  
6. **Fase 6: Síntese e Geração com LLM (Dupla Camada):** Todos os dados e análises (outliers, padrões de títulos, oportunidades de baixa concorrência) são compilados e formatados em um grande contexto JSON.  
   * **LLM 1 (O Analista Criativo):** Este contexto é enviado para um primeiro agente de IA com um prompt para atuar como um "analista de YouTube especializado em viralização". Sua missão é analisar todos os dados e gerar 15 ideias de vídeos, seguindo regras estritas para basear cada justificativa em métricas reais extraídas das fases anteriores.  
   * **LLM 2 (O Formatador Preciso):** A saída de texto livre do primeiro LLM é passada para um segundo agente, cujo único trabalho é formatar a resposta em um JSON limpo e válido, garantindo que a estrutura final seja sempre consistente e livre de erros.  
7. **Fase 7: Validação e Saída:** Um script final valida a saída do LLM (verifica se há 15 itens, se os scores são únicos, etc.). Se a validação falhar, ele pode acionar um re-try. Se for bem-sucedido, os dados são divididos em dois fluxos de saída: um JSON detalhado e uma versão tabular, que é enviada diretamente para uma planilha no Google Sheets.

### **1.2. Diagrama da Solução (Mermaid)**

Snippet de código  
graph LR
    subgraph "1. Entrada e Busca Inicial"
        direction LR
        id_062b5bd8("💬 When chat message received")
        id_acad55d8("🔧 Globals")
        id_342a98ce("🌐 HTTP Request - Busca Vídeos")
        id_5fc212bc("쪼 Split Out")
    end

    subgraph "2. Coleta de Dados Detalhados"
        direction LR
        id_fe8395e4("🌐 request_canais")
        id_e726e4f2("🌐 request_videos")
        id_3528aa3c("⚙️ Dados dos Canais")
        id_b6cdfe63("⚙️ Dados_dos_Vídeos")
    end

    subgraph "3. Análise de Dados e Títulos"
        direction LR
        id_f5849a82("🔄 Merge")
        id_b3c8c5cf("🐞 debug node")
        id_74707c60("💻 outlier detection node")
        id_5e37aaaf("💻 analise_de_titulos")
        id_f12a5df5("💻 Code - Formata p/ LLM")
    end

    subgraph "4. Análise de Concorrência"
        direction LR
        id_013f9048("💻 extrair_padroes_para_concorrência")
        id_d8177aa0("🌐 busca_concorrencia_yt")
        id_6a3a6600("💻 preparar_videos_id")
        id_0f0c9cab("🌐 HTTP Request - Stats Concorrência")
        id_f7aa5517("💻 preparar_dados_analise")
        id_b4987d61("💻 calcular_score_concorrencia")
    end

    subgraph "5. Junção e Preparação para IA"
        direction TB
        id_b89703a4("🔄 Merge2")
        id_2aa65783("💻 preparar_dados_prompt")
    end

    subgraph "6. Geração de Ideias com IA"
        direction TB
        id_f10bfabb("🧠 Basic LLM Chain")
        id_f7da21f4("🤖 AI Agent")
        id_1e155057("🧠 OpenAI Chat Model")
        id_fb12d123("📄 Structured Output Parser")
    end

    subgraph "7. Validação e Roteamento"
        direction TB
        id_9b3d0697("💻 validação_output_llm")
        id_843143c8{{"🔀 Switch"}}
    end

    subgraph "8. Saída dos Dados"
        direction TB
        id_c8caa116("💻 export_json_final")
        id_d23c5663("💻 padroniza_saida_planilha")
        id_ee07768a("📊 Append or update row in sheet")
    end

    %% Conexões Principais
    id_062b5bd8 --> id_acad55d8
    id_acad55d8 --> id_342a98ce
    id_342a98ce --> id_5fc212bc
    id_5fc212bc --> id_fe8395e4
    id_5fc212bc --> id_e726e4f2
    id_fe8395e4 --> id_3528aa3c
    id_e726e4f2 --> id_b6cdfe63
    id_3528aa3c --> id_f5849a82
    id_b6cdfe63 --> id_f5849a82
    id_f5849a82 --> id_b3c8c5cf
    id_b3c8c5cf --> id_74707c60
    id_74707c60 --> id_5e37aaaf
    id_5e37aaaf --> id_f12a5df5
    
    id_f12a5df5 --> id_013f9048
    id_013f9048 --> id_d8177aa0
    id_d8177aa0 --> id_6a3a6600
    id_6a3a6600 --> id_0f0c9cab
    id_0f0c9cab --> id_f7aa5517
    id_f7aa5517 --> id_b4987d61

    id_b4987d61 --> id_b89703a4
    id_f12a5df5 --> id_b89703a4
    id_b89703a4 --> id_2aa65783
    
    id_2aa65783 --> id_f10bfabb
    id_f10bfabb --> id_f7da21f4
    id_f7da21f4 --> id_9b3d0697
    id_9b3d0697 --> id_843143c8

    id_843143c8 -- "retry" --> id_f10bfabb
    id_843143c8 -- "success" --> id_c8caa116
    id_843143c8 -- "success" --> id_d23c5663
    id_d23c5663 --> id_ee07768a

    %% Conexões da IA (LangChain)
    id_1e155057 -.->|Language Model| id_f10bfabb
    id_1e155057 -.->|Language Model| id_f7da21f4
    id_fb12d123 -.->|Output Parser| id_f7da21f4
    
    %% Estilos
    style id_062b5bd8 fill:#9d9,stroke:#333,stroke-width:2px
    style id_f10bfabb fill:#C9B8F2,stroke:#333,stroke-width:2px
    style id_f7da21f4 fill:#B8C2F2,stroke:#333,stroke-width:2px
    style id_1e155057 fill:#F2B8B8,stroke:#333,stroke-width:2px
    style id_fb12d123 fill:#F2E2B8,stroke:#333,stroke-width:2px
    style id_843143c8 fill:#f9f,stroke:#333,stroke-width:4px
    classDef codeNode fill:#d5fada,stroke:#333,stroke-width:2px
    class id_acad55d8,id_3528aa3c,id_b6cdfe63,id_b3c8c5cf,id_74707c60,id_5e37aaaf,id_f12a5df5,id_013f9048,id_6a3a6600,id_f7aa5517,id_b4987d61,id_2aa65783,id_9b3d0697,id_c8caa116,id_d23c5663 codeNode
    classDef httpNode fill:#add8e6,stroke:#333,stroke-width:2px
    class id_342a98ce,id_fe8395e4,id_e726e4f2,id_d8177aa0,id_0f0c9cab httpNode
    classDef otherNode fill:#f0e68c,stroke:#333,stroke-width:2px
    class id_5fc212bc,id_f5849a82,id_b89703a4,id_ee07768a otherNode

### ***1.3. Fontes de Dados Utilizadas e Por Quê***

*A única fonte de dados utilizada neste projeto foi a **API de Dados do YouTube v3**, pelos seguintes motivos:*

1. ***Obrigatoriedade do Desafio:** O requisito principal era claro: "é obrigatório o uso de dados reais do próprio Youtube". A API oficial é a forma mais direta, confiável e estruturada de cumprir essa exigência.*  
2. ***Riqueza e Precisão dos Dados:** A API fornece acesso direto e granular a todas as métricas necessárias para uma análise de tendências aprofundada. Foram utilizados os seguintes endpoints:*  
   * ***`search.list`**: Para a fase inicial de coleta. Permite buscar vídeos por palavra-chave (o nicho), ordenando por relevância ou, como no nosso caso, por `viewCount`, para capturar imediatamente os conteúdos de maior sucesso.*  
   * ***`videos.list`**: Para a fase de enriquecimento. Ao passar uma lista de IDs de vídeo, este endpoint retorna estatísticas detalhadas como contagem de visualizações, likes e comentários, que são a base para os cálculos de Z-Score e Score de Engajamento.*  
   * ***`channels.list`**: Também na fase de enriquecimento. Permite obter dados sobre o canal que publicou o vídeo, como o número de inscritos, fornecendo um contexto valioso sobre a autoridade e o alcance do criador.*  
3. ***Confiabilidade e Escalabilidade:** Utilizar a API oficial garante que os dados são precisos e atualizados, eliminando os problemas de inconsistência que podem ocorrer com métodos alternativos como scraping. Além disso, a estrutura da API é projetada para lidar com um grande volume de requisições de forma escalável.*

## **2\. Sistema Funcionando**

### **2.1. Prompts Utilizados (Parte Fundamental)**

Foram utilizados dois prompts principais em sequência para garantir tanto a qualidade da análise quanto a confiabilidade do output.

#### **Prompt 1: O Analista de YouTube (Nó `Basic LLM Chain`)**

Este prompt instrui o LLM a agir como um especialista, analisando o JSON de dados brutos e gerando as ideias em um formato de texto estruturado. Ele é focado na criatividade e na profundidade da análise.

\=Você é um analista de YouTube especializado em identificar padrões virais.

CONTEXTO:  
Recebi dados estatísticos completos de vídeos do YouTube sobre um nicho específico, incluindo high performers, métricas de engagement, palavras-chave mais usadas e análise de concorrência.

DADOS FORNECIDOS:  
{{ $json.dados\_prompt }}

SUA MISSÃO:  
Analisar os dados acima e gerar PELO MENOS 15 ideias de vídeos inovadoras e baseadas em dados reais.

PARA CADA IDEIA, FORNEÇA:

1\. TÍTULO: Chamativo, específico, com gatilho emocional ou número  
2\. SCORE: De 55 a 95 (varie bastante, não repita valores)  
3\. JUSTIFICATIVA: Cite SEMPRE métricas reais dos dados:  
   \- Nome do vídeo de referência  
   \- Z-Score específico (views, likes ou comments)  
   \- Engagement score  
   \- Número de views  
   \- Explique POR QUE aquele padrão funciona  
4\. PALAVRAS-CHAVE: 3 a 5 palavras relevantes  
5\. CONCORRÊNCIA: baixa, média ou alta (varie: \~5 de cada tipo)

ESTRATÉGIA DE ANÁLISE:

IDEIAS 1-5 (Score 85-95):  
\- Use os TOP 5 performers do ranking  
\- Foque em vídeos com Z-Score \> 2.5  
\- Cite engagement scores acima de 75

IDEIAS 6-10 (Score 70-84):  
\- Use performers médios/altos  
\- Z-Score entre 1.5 e 2.5  
\- Explore padrões de títulos recorrentes

IDEIAS 11-15 (Score 55-69):  
\- Use oportunidades de baixa concorrência  
\- Variações criativas de high performers  
\- Combine palavras-chave de formas únicas

REGRAS OBRIGATÓRIAS:

✅ SEMPRE cite vídeos reais dos dados fornecidos  
✅ SEMPRE inclua números específicos (Z-Score, engagement, views)  
✅ SEMPRE explique o padrão ou estratégia por trás da ideia  
✅ Varie os scores (nenhum score repetido)  
✅ Explore TODA a lista de high performers, não apenas os 5 primeiros  
✅ Misture níveis de concorrência (não faça todas "baixa" ou todas "alta")

❌ NUNCA invente métricas que não estão nos dados  
❌ NUNCA use justificativas genéricas ("é um bom tema")  
❌ NUNCA repita o mesmo padrão de título 15 vezes

FORMATO DE SAÍDA:

Liste as ideias de forma clara e estruturada. Exemplo:

\---  
IDEIA 1  
Título: Por Que Suas Pernas Não Crescem? 5 Erros Fatais  
Score: 92  
Justificativa: Baseado no vídeo "Why You Have Small Legs" que possui engagement de 57 e Likes Z-Score de 2.89. O padrão "Problema \+ Número Específico" aumenta CTR em 40% e gera alta retenção por promover curiosidade imediata. O formato de "erros" cria urgência e senso de descoberta.  
Palavras-chave: pernas, crescer, erros, treino, musculação  
Concorrência: alta  
\---

\[Continue com mais 13 ideias...\]

IMPORTANTE: Não se preocupe em retornar JSON. Apenas liste as ideias de forma clara como no exemplo acima. O formato será ajustado depois.

Comece a análise agora\!

#### **Prompt 2: O Formatador JSON (Nó `AI Agent`)**

Este prompt é mais técnico. Ele recebe a saída do "Analista" e tem uma única missão: convertê-la para um JSON perfeitamente válido, seguindo um schema rigoroso. Ele também tem regras para corrigir pequenas falhas, como garantir 15 itens e scores únicos.

\=Você é um formatador especializado em converter texto estruturado em JSON válido.

TEXTO RECEBIDO DO GERADOR:  
{{ $json.text }}

SUA MISSÃO:  
Converter o texto acima em um objeto JSON válido com EXATAMENTE 15 ideias no formato especificado.

FORMATO OBRIGATÓRIO:  
{  
  "nicho": "weak legs",  
  "data\_analise": "{{ $now.format('YYYY-MM-DD') }}",  
  "ideias": \[  
    {  
      "titulo": "string",  
      "score": number,  
      "justificativa": "string",  
      "palavras\_chave": \["string", "string"\],  
      "concorrencia": "baixa" | "média" | "alta"  
    }  
  \]  
}

REGRAS DE FORMATAÇÃO:

1\. QUANTIDADE:  
   \- Se o texto tiver MENOS de 15 ideias: crie variações das existentes para completar 15  
   \- Se tiver MAIS de 15 ideias: selecione as 15 melhores (maiores scores)  
   \- NUNCA retorne menos de 15 ou mais de 15

2\. SCORES:  
   \- Todos devem ser números entre 55 e 95  
   \- NUNCA repita o mesmo score  
   \- Se houver scores repetidos, ajuste \+1 ou \-1 para tornar únicos

3\. JUSTIFICATIVAS:  
   \- Mantenha TODOS os números e métricas do texto original  
   \- Preserve citações de vídeos e Z-Scores  
   \- Se a justificativa estiver vaga, NÃO invente números \- mantenha o texto original

4\. PALAVRAS-CHAVE:  
   \- Sempre um array de strings  
   \- Mínimo 3, máximo 5 palavras por ideia  
   \- Converter para lowercase

5\. CONCORRÊNCIA:  
   \- Apenas valores válidos: "baixa", "média", "alta"  
   \- Distribuição sugerida: \~5 baixa, \~5 média, \~5 alta  
   \- Se o texto não especificar, infira baseado no score:  
     \* Score 85-95: "alta"  
     \* Score 70-84: "média"    
     \* Score 55-69: "baixa"

6\. FORMATO:  
   \- Retorne APENAS o objeto JSON  
   \- SEM \`\`\`json ou markdown  
   \- SEM explicações adicionais  
   \- SEM comentários no JSON

CHECKLIST ANTES DE RETORNAR:  
\- \[ \] Exatamente 15 objetos no array "ideias"  
\- \[ \] Todos os scores são únicos  
\- \[ \] Todas as justificativas têm os números originais preservados  
\- \[ \] Todos os campos obrigatórios estão presentes  
\- \[ \] Concorrência apenas com valores válidos  
\- \[ \] JSON válido (sem erros de sintaxe)

RETORNE APENAS O JSON. NADA MAIS.

### **2.2. Scripts Auxiliares (Nós de Código)**

O workflow utiliza diversos nós de código para manipulação e análise de dados. Os mais importantes são:

* **`outlier detection node`**: O coração da análise estatística. Ele ingere os dados de todos os vídeos, calcula média, desvio padrão e Z-Score para as principais métricas. Define o que é um "high performer", que é a base para todas as análises subsequentes.  
* **`analise_de_titulos`**: Focado em engenharia reversa de conteúdo. Ele processa apenas os títulos dos vídeos de sucesso, remove *stop words*, conta a frequência das palavras-chave mais importantes e identifica padrões estruturais e gatilhos emocionais.  
* **`calcular_score_concorrencia`**: Mede a saturação do nicho. Ele recebe os resultados da busca por palavras-chave de sucesso e calcula um score de 0 a 100 baseado no volume de vídeos concorrentes e no engajamento médio deles.  
* **`validação_output_llm`**: Garante a qualidade do entregável final. Este script funciona como um "controle de qualidade", verificando a estrutura do JSON gerado pela IA, a quantidade de ideias, a unicidade dos scores e a presença de campos obrigatórios.

## **3\. Demonstração**

Um vídeo de demonstração do tipo Loom, mostrando o workflow em funcionamento do início ao fim, foi gravado e está disponível para visualização. Ele complementa esta documentação, exibindo na prática a execução de cada etapa descrita.

*https://www.loom.com/share/863f24290cc94442be37f05586de54c8?sid=a2c1414d-f661-4456-8aff-463fa48351ef* 

## **3\. Planilha com resultados:**

Segue abaixo a planilha do google docs com os resultados obtidos:
 
https://docs.google.com/spreadsheets/d/13LPtaHZclFRiyGOCSc-H8Y6X4m94NIllHqZ2r6F-rJ8/edit?gid=0#gid=0

