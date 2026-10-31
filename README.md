# Dashboard de Vendas em Excel

Dashboard interativo que transforma uma base bruta de vendas em indicadores e gráficos para analisar o desempenho comercial e apoiar decisões. Um filtro de região atualiza todo o painel.

Projeto desenvolvido no Desafio de Projeto do curso **Reclame AQUI – Dados e IA na Prática** (DIO).

> A base contém 771 vendas **fictícias** de 2025 de uma loja de eletrônicos e utilidades.

## Como usar

1. Abra `dashboard-vendas.xlsx` na aba **Dashboard**.
2. Escolha uma região na célula amarela (**Todas**, Sudeste, Nordeste, Sul, Centro-Oeste ou Norte).
3. KPIs, destaques e gráficos se atualizam automaticamente.

## Estrutura

| Aba | Conteúdo |
|---|---|
| **Dashboard** | Filtro, 6 KPIs, destaques automáticos e 4 gráficos |
| **Análises** | Tabelas de apoio que alimentam os gráficos, todas respeitando o filtro |
| **Base** | Dados brutos em tabela formatada (`tbVendas`): uma linha por venda |
| **Metas** | Meta de faturamento por mês e região (`tbMetas`) |

### Colunas da base
`ID Pedido`, `Data`, `Mês`, `Vendedor`, `Região`, `Canal`, `Categoria`, `Produto`, `Quantidade`, `Preço unitário`, `Custo unitário`, `Faturamento`, `Custo total`, `Lucro`.

As colunas Mês, Faturamento, Custo total e Lucro são calculadas:
```
Mês         = MÊS(B2)
Faturamento = Quantidade × Preço unitário
Custo total = Quantidade × Custo unitário
Lucro       = Faturamento − Custo total
```

## Como o filtro funciona

A célula de região tem **validação de dados** em lista. Na aba Análises, o intervalo nomeado `FiltroRegiao` converte a escolha em critério:
```
=SE(Dashboard!$D$4="Todas"; "*"; Dashboard!$D$4)
```
O `*` é um curinga: em `SOMASES` e `CONT.SES` ele corresponde a qualquer texto. Assim, a mesma fórmula serve para "Todas" e para uma região específica, sem precisar de `SE` em cada cálculo.

## Explicação dos cálculos

### Tabelas de apoio (aba Análises)

| Tabela | Fórmula principal |
|---|---|
| Faturamento por mês | `=SOMASES(Base!$L:$L; Base!$E:$E; FiltroRegiao; Base!$C:$C; A5)` |
| Meta por mês | `=SOMASES(Metas!$C:$C; Metas!$B:$B; FiltroRegiao; Metas!$A:$A; A5)` |
| % da meta | `=SE(D5=0; 0; C5/D5)` |
| Faturamento e lucro por categoria | `SOMASES` com o critério de categoria |
| Margem por categoria | `=SE(I5=0; 0; J5/I5)` |
| Faturamento por canal e % do total | `SOMASES` e divisão pelo total |

As fórmulas usam colunas inteiras (`Base!$L:$L`), então novas vendas coladas no fim da base entram automaticamente nos cálculos.

### Ranking de vendedores
O gráfico mostra os vendedores do maior para o menor faturamento, e a ordem muda conforme o filtro.

1. Coluna auxiliar para desempate: `=B20 + LIN()/1000000` (evita que dois vendedores com o mesmo valor quebrem a busca).
2. Busca do 1º, 2º, 3º... colocado:
```
=ÍNDICE($A$20:$A$27; CORRESP(MAIOR($C$20:$C$27; E20); $C$20:$C$27; 0))
```
`MAIOR` encontra o n-ésimo maior valor, `CORRESP` encontra a posição dele e `ÍNDICE` devolve o nome.

### KPIs

| KPI | Cálculo |
|---|---|
| Faturamento | Soma do faturamento mensal da aba Análises |
| Lucro | Soma do lucro mensal |
| Margem | Lucro ÷ Faturamento |
| Pedidos | `=CONT.SES(Base!$E:$E; FiltroRegiao; Base!$A:$A; "PED-*")` |
| Ticket médio | Faturamento ÷ Pedidos |
| Meta atingida | Faturamento ÷ Meta do período |

### Destaques automáticos
Textos montados com fórmulas que mudam com o filtro:
```
="Melhor mês: " & ÍNDICE(meses; CORRESP(MÁXIMO(faturamento); faturamento; 0))
="Meses acima da meta: " & CONT.SE(%meta; ">=1") & " de 12"
```

## Gráficos

| Gráfico | Tipo | Por quê |
|---|---|---|
| Faturamento x Meta por mês | Colunas + linha | Evolução no tempo comparada à meta |
| Faturamento por categoria | Barras horizontais | Comparação entre categorias |
| Ranking de vendedores | Barras horizontais ordenadas | Ranking de desempenho |
| Faturamento por canal | Pizza | Participação de 3 partes no total |

## Prints

> Adicione os prints na pasta `images/`.

![Dashboard](images/dashboard.png)
![Dashboard filtrado](images/dashboard-sudeste.png)
![Base de dados](images/base.png)

## Como adaptar para outra base

Cole os dados na aba **Base** mantendo a ordem das colunas. As metas ficam na aba **Metas**, uma linha por mês e região.
