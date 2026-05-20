# Projeto Dashboard Comercial

Dashboard em Power BI para acompanhamento executivo de desempenho comercial, com foco em receita, lucro, margem, ticket médio, quantidade de vendas e análise por produto, vendedor, período e estado.

## Arquivos

- `Projeto Dashboard Comercial.pbix`: arquivo principal do Power BI.
- `README.md`: documentação funcional do relatório, modelo de dados, relacionamentos e medidas DAX.

## Objetivo

O relatório consolida indicadores comerciais em uma visão executiva para apoiar a leitura rápida de performance. A página principal permite acompanhar faturamento, lucratividade, evolução temporal, desempenho por categoria, ranking geográfico e produtividade por vendedor.

## Bases e tabelas

O modelo do PBIX contém as seguintes tabelas:

| Tabela | Tipo | Finalidade |
| --- | --- | --- |
| `fVendas` | Fato | Base transacional de vendas. Concentra os registros usados para calcular receita, quantidade, lucro, ticket médio e demais indicadores comerciais. |
| `dClientes` | Dimensão | Cadastro ou atributos dos clientes, incluindo `estado`, usado no ranking de vendas por UF. |
| `dProdutos` | Dimensão | Cadastro de produtos, incluindo `categoria`, usado nos filtros e na visão de faturamento por categoria. |
| `dVendedores` | Dimensão | Cadastro da força comercial, incluindo `nome` e `equipe`, usados na tabela dinâmica e no filtro por equipe. |
| `dCalendario` | Dimensão | Calendário analítico, incluindo `Mes Nome` e `Trimestre`, usado na série temporal e nos filtros de período. |
| `Medidas` | Medidas | Tabela auxiliar para centralizar as medidas DAX do relatório. |

## Relacionamentos

O desenho do modelo segue uma estrutura estrela: a tabela fato `fVendas` fica no centro das análises e se relaciona com as dimensões de clientes, produtos, vendedores e calendário.

Relacionamentos esperados no modelo:

| Origem | Destino | Cardinalidade esperada | Uso analítico |
| --- | --- | --- | --- |
| `fVendas` | `dClientes` | Muitos para um | Permite analisar vendas por localização e atributos de cliente. |
| `fVendas` | `dProdutos` | Muitos para um | Permite analisar faturamento e volume por produto/categoria. |
| `fVendas` | `dVendedores` | Muitos para um | Permite analisar performance por vendedor e equipe. |
| `fVendas` | `dCalendario` | Muitos para um | Permite analisar evolução por mês, trimestre e demais recortes de data. |

## Página do relatório

### Executivo

Página principal do dashboard. Contém:

- Cards de indicadores: `Receita Total`, `Lucro Total`, `Margem Lucro %`, `Ticket Médio` e `Crescimento KPI`.
- Gráfico combinado: `Receita Total x Margem de Lucro`, usando `dCalendario.Trimestre`.
- Gráfico de barras: `TOP 7 Vendas por Estado`, usando `dClientes.estado` e `Qtd Vendas`.
- Gráfico de rosca: `Faturamento por categoria`, usando `dProdutos.categoria` e `Faturamento Produtos`.
- Tabela dinâmica por vendedor, usando `dVendedores.nome`, `Qtd Vendas`, `Faturamento Total` e `Qtd Itens Vendidos`.
- Segmentações por `dCalendario.Mes Nome`, `dProdutos.categoria` e `dVendedores.equipe`.

## Medidas DAX

As medidas abaixo representam a lógica de negócio documentada para o relatório. Caso os nomes físicos das colunas mudem na base, ajuste apenas as referências de coluna mantendo o conceito da medida.

```DAX
Receita Total =
SUM ( fVendas[valor_total] )
```

Soma o valor total vendido no contexto de filtros aplicado.

```DAX
Faturamento Total =
[Receita Total]
```

Medida de apoio para exibir o faturamento total em visuais de vendedor e performance comercial.

```DAX
Faturamento Produtos =
[Receita Total]
```

Medida usada para distribuir o faturamento por categoria de produto.

```DAX
Qtd Vendas =
COUNTROWS ( fVendas )
```

Conta a quantidade de registros de venda no contexto atual. Se a base possuir uma coluna de pedido/venda única, a alternativa recomendada é `DISTINCTCOUNT ( fVendas[id_venda] )`.

```DAX
Qtd Itens Vendidos =
SUM ( fVendas[quantidade] )
```

Soma a quantidade de itens vendidos.

```DAX
Ticket Médio =
DIVIDE ( [Receita Total], [Qtd Vendas] )
```

Calcula o valor médio por venda, evitando erro de divisão por zero.

```DAX
Lucro Total =
SUM ( fVendas[lucro] )
```

Soma o lucro das vendas. Quando o lucro não existir como coluna na base, a medida pode ser calculada por diferença entre receita e custo:

```DAX
Lucro Total =
[Receita Total] - SUM ( fVendas[custo_total] )
```

```DAX
Margem Lucro % =
DIVIDE ( [Lucro Total], [Receita Total] )
```

Calcula a margem de lucro percentual.

```DAX
Crescimento Mês % =
VAR ReceitaAtual = [Receita Total]
VAR ReceitaMesAnterior =
    CALCULATE (
        [Receita Total],
        DATEADD ( dCalendario[data], -1, MONTH )
    )
RETURN
    DIVIDE ( ReceitaAtual - ReceitaMesAnterior, ReceitaMesAnterior )
```

Compara a receita do período atual contra o mês anterior.

```DAX
Crescimento KPI =
[Crescimento Mês %]
```

Medida usada no card executivo para destacar a variação percentual do faturamento.

## Interações e filtros

O relatório permite filtrar a análise por:

- Mês, usando `dCalendario.Mes Nome`.
- Categoria de produto, usando `dProdutos.categoria`.
- Equipe comercial, usando `dVendedores.equipe`.

Esses filtros afetam os KPIs, gráficos e tabela da página executiva, permitindo comparações por período, mix de produtos e time de vendas.

## Observações técnicas

- O PBIX foi criado no Power BI Service/Cloud e salvo com release `2026.03`.
- O tema visual registrado no arquivo é `CY26SU02`.
- O relatório contém recurso de imagem registrado em `Report/StaticResources/RegisteredResources`.
- As medidas do layout são: `Receita Total`, `Faturamento Total`, `Faturamento Produtos`, `Lucro Total`, `Margem Lucro %`, `Ticket Médio`, `Qtd Vendas`, `Qtd Itens Vendidos`, `Crescimento Mês %` e `Crescimento KPI`.
