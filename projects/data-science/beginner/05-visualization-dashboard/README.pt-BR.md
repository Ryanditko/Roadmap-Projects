# Dashboard de Visualização de Dados

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Um gráfico estático responde a uma pergunta; um dashboard deixa o usuário fazer a própria. Neste projeto você pega um conjunto de dados e constrói um pequeno dashboard interativo onde o leitor pode filtrar, selecionar e aprofundar nos dados para chegar a conclusões que você não pré-escreveu. O desafio é a contenção: um bom dashboard tem um propósito claro e três ou quatro visões bem escolhidas, não quinze gráficos disputando atenção. Você vai praticar escolher o gráfico certo por pergunta, ligar filtros que atualizam todas as visões de uma vez e desenhar um layout que se lê de cima para baixo como um argumento.

## Pré-requisitos

- Python básico e pandas
- Um framework de dashboard (Streamlit ou Dash) e uma biblioteca de plotagem (Plotly, Matplotlib ou Seaborn)
- Conforto para carregar um conjunto de dados em um dataframe
- Um conjunto de dados com categorias para filtrar e números para agregar — um CSV de vendas, clima ou esportes do [Kaggle](https://www.kaggle.com/datasets) funciona bem

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Definir o propósito único de um dashboard e as perguntas que ele deve responder
- Associar cada pergunta a um tipo de gráfico apropriado
- Ligar filtros interativos que atualizam múltiplas visões vinculadas juntas
- Agregar dados em tempo real em resposta às seleções do usuário
- Dispor resumos de KPI e gráficos para que a história se leia com clareza

## Requisitos Funcionais

1. O dashboard deve carregar um conjunto de dados e exibir ao menos três tipos distintos de gráfico.
2. Deve prover ao menos um filtro (dropdown, slider ou intervalo de datas) que atualize as visões.
3. Mudar um filtro deve atualizar cada gráfico afetado, não apenas um.
4. Deve mostrar ao menos dois números-resumo de KPI (totais, médias, contagens).
5. Gráficos devem ter títulos, rótulos de eixo e legendas legíveis.
6. O layout deve agrupar visões relacionadas e ler em uma ordem sensata.
7. Deve tratar um resultado de filtro vazio com elegância (sem travar, com uma mensagem clara).

## Marcos Sugeridos

1. **Marco 1 — Visões estáticas:** Carregue os dados e renderize os gráficos e KPIs principais sem interatividade.
2. **Marco 2 — Interatividade:** Adicione filtros e ligue-os para que todas as visões atualizem a partir da mesma seleção.
3. **Marco 3 — Acabamento:** Organize o layout, trate estados vazios e adicione detalhe no hover ou anotações.

## Esboço de Dados e Interface

```text
Layout do dashboard
  +-----------------------------------------------------+
  |  Título + propósito em uma linha                    |
  |  [Filtro: categoria v] [Filtro: intervalo de datas] |
  +------------------+----------------+-----------------+
  | KPI: total       | KPI: média     | KPI: contagem   |
  +------------------+----------------+-----------------+
  |  Gráfico de tendência (linha, tempo no x)           |
  +-----------------------------------------------------+
  | Decomposição (barra) | Distribuição (histograma/box)|
  +-----------------------------------------------------+

Modelo de interação
  mudança de filtro -> re-consulta dataframe -> recomputa KPIs -> redesenha gráficos
  resultado vazio   -> mostra "Sem dados para esta seleção" em vez de gráficos em branco
```

## Desafios Extras

- Adicione drill-down: clicar em uma barra filtra os outros gráficos para aquele segmento.
- Adicione um botão de exportação que baixa os dados filtrados atuais como CSV.
- Adicione uma comparação de intervalo de datas (período atual vs anterior) com indicadores de delta.
- Publique o dashboard (Streamlit Community Cloud ou similar) e compartilhe o link.

## Definição de Pronto

- [ ] O dashboard tem um propósito declarado e três ou mais tipos de gráfico.
- [ ] Ao menos um filtro atualiza toda visão dependente simultaneamente.
- [ ] KPIs recomputam corretamente quando os filtros mudam.
- [ ] Todo gráfico é rotulado e legível sem explicação.
- [ ] Uma seleção de filtro vazia mostra uma mensagem, não um travamento ou tela em branco.

## Armadilhas Comuns

- Enfiar todo gráfico possível em vez dos poucos que servem ao propósito.
- Filtros que atualizam um gráfico mas deixam os KPIs ou outras visões desatualizados.
- Recomputar o conjunto completo a cada tecla, deixando o dashboard lento.
- Escolhas de cor bonitas mas que não codificam nada, ou que falham para usuários daltônicos.

## Recursos

- [Streamlit documentation](https://docs.streamlit.io/) — o caminho mais rápido para um dashboard em Python.
- [Plotly Python graphing library](https://plotly.com/python/) — gráficos interativos com hover e zoom embutidos.
- [Dash documentation](https://dash.plotly.com/) — um framework de dashboard mais customizável.
- [Google Material: Data visualization](https://m2.material.io/design/communication/data-visualization.html) — orientação de layout e cor.
