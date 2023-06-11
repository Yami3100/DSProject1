## Projeto acerca de dados disponibilizados on-line sobre uma franquia de hotelaria.

Dados disponibilizados em .xlsx (planilha do Excel), carregados no SQL Server 2022. SQL Server Managament Studio 19.1.

Contexto:

Franquia de Hotelaria interessada em comparar e confrontar os dados entre City Hotel e Resort Hotel, investigar a receita. Responder a pergunta: vale a pena investir no estacionamento? 

Nos dados, não há uma coluna já com a informação da receita, mas é possível estimá-la com a seguinte query
```
SELECT 
	arrival_date_year AS ano,
	hotel,
	ROUND(SUM((stays_in_weekend_nights+stays_in_week_nights)*adr), 2) AS receita
FROM 
	hotels
GROUP BY
	arrival_date_year, hotel
```
A estadia foi separada em duas colunas: em dias de semana e nos finais de semana; somando-as, obtemos a estadia total em dias, que nos dá a receita quando multiplicado pelo ADR (Average Daily Rate), KPI da área de hotelaria.

Entretanto, entre as diferentes tabelas, há a necessidade de calcular gastos com descontos e refeições. A partir daqui, quis trabalhar no Power BI. Fiz a conexão com o SQL Server, mas trabalhei com um JOIN de todas as tabelas.
```
WITH hotels AS (
SELECT * FROM dbo.['2018$']
UNION
SELECT * FROM dbo.['2019$']
UNION
SELECT * FROM dbo.['2020$'])

SELECT * FROM hotels 
LEFT JOIN dbo.market_segment$ AS tb_mseg 
ON hotels.market_segment = tb_mseg.market_segment
LEFT JOIN  dbo.meal_cost$ 
ON hotels.meal = dbo.meal_cost$.meal
```
Colunas duplicadas foram excluídas no Power Query e a coluna Revenue (receita) foi calculada usando a seguinte fórmula em DAX:
```
Revenue = ([stays_in_week_nights]+[stays_in_weekend_nights])*([adr]*[Discount]+[Cost])
```
Cost se refere ao custo diário das refeições durante a estadia, mas o desconto não se aplica. Desse modo, temos: "Estadia em dias" * "(Taxa diária + Refeições Diárias)", obtendo uma receita precisa. 

Dados devidamente transformados, iniciou-se o processo de design do dashboard para a visualização dos dados. Medidas e filtros foram adicionados. Uma série temporal foi adicionada para confrontar a receita ganha entre City Hotel e Resort Hotel ao longo do tempo. Nota-se que, em 2019, de Julho ao final de Agosto, o Resort Hotel obteve alta histórica. Cabe investigar os eventos durante esta época que possam ter contribuído.

Para responder se vale a pena expandir o estacionamento, foi criada uma medida acerca da porcentagem de vagas de estacionamento requisitadas. 
```
Porcentagem de Estacionamento = SUM(Consulta1[required_car_parking_spaces])/[Noites totais]
``` 
Onde [Noites totais] é:
```
Noites totais = SUM(Consulta1[stays_in_week_nights])+(SUM(Consulta1[stays_in_weekend_nights]))
```
