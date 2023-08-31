# Problema
O objetivo é prever o crescimento do índice GDP de cada país nos anos de 2024-2028, posteriormente comparando-os com o previsto pelo [Statistica](https://www.statista.com/). Além disso, é necessário substituir os campos "no data" por valores numéricos, utilizando inferências de sua escolha.

> Dataset overview

|Variable|Meaning|
|---|---|
|Real GDP growth (Annual percent change)| Nome do país ou grupo de países|
|1980| GDP do ano de 1980|
|...|...|
|2028| GDP do ano de 2028|

# Premissas
- Os valores numéricos de GDP são valores percentuais;
- O dataset possui variáveis com a informação 'no data' que obrigatoriamente devem ser substituídas por valores numéricos;
- os valores de GDP de 2024 a 2028 também sâo resultados de uma previsão de serie temporal e o método é desconhecido.

# Planejamento da solução

1. **Entendimento do problema:** Observando o conjunto de dados e buscar entender o comportamento dos dados, como estão distribuídos, se há valores faltantes, se há outliers, etc.

2. **Coleta de dados:** dados disponíveis no formato excel.

4. **Análise gráfica:** Os dados dos países foram plotados em gráficos de linha para observar o comportamento dos dados ao longo do tempo, além de observar a distribuição dos dados e a presença de outliers. A análise foi executada por grupos de países, para visualizar quais tinham comportamentos semelhantes, para auxiliar na substituição dos valores nulos.

3. **Dados Faltantes:** Após a substituição dos valores 'no data' por valores nulos, utilizando a função do numpy `.nan`, foi possível observar que 62 paíse e regiões possuem dados faltantes com variação de 1 até 32 linhas com valores nulos, assim foi necessário estudar métodos para lidar com valores nulos de forma que o preenchimento mantesse os dados com uma boa variabilidade para não enviesar completamente o modelo para que ele observasse comportamento diferente entre países e regiões.
<ul>

- **Países e regiões com 2 ou menos valores nulos:** foi utilizado o método de preenchimento de valores nulos com a média dos valores da coluna, pois a variabilidade dos dados não foi comprometida.
- **Países da antiga união soviética:** possuem dados de GDP disponíveis somente a partir de 1991, o ano da dissolução da União Soviética. A exceção é Belarus, que possui dados anteriores a 1991. Portanto, para os anos anteriores a 1991, os valores de PIB desses países serão preenchidos com o mesmo valor de Belarus para o ano de 1980. Em seguida, uma interpolação dos valores ausentes será realizada para todos os países, com o objetivo de manter uma tendência de crescimento semelhante.
- **Regiões com valores nulos:** Será feita a susbstituição com a média dos países que pertencem região, sendo a média dos países em cada ano
- **Para os demais países**
Para os outros países a susbstituição será feita da seguinte maneria:
    
    - A média dos países do mesmo grupo é calculada
    - É feita uam contagem para saber se o país possui dados em sua maioria acima ou abaixo dessa média
    - Se o reultado for acima da média:
        - Os valores nulos serão preenchidos com a média do grupo + o valor do desvio padrão do próprio país
    - Se o reultado for abaixo da média:
        - Os valores nulos serão preenchidos com a média do grupo - o valor do desvio padrão do próprio país
- **Paises com variação alta:** Como alguns paises possuem variações extremas ao longo do tempo, foi optado pela substituição apenas da média sem adição ou subtração de valores nulos

- **Países com valores futuros nulos:** Para países com dados de 2024 a 2028 faltantes será feita a susbstituição pela média do próprio país, para ter mais segurança nas métricas de comparação quando for feita a modelagem dos dados
</ul>

4. Decomposição sazonal: Para observar a sazonalidade dos dados foi feita a decomposição sazonal dos dados de GDP de cada país, para observar se há uma tendência de crescimento ou decrescimento ao longo do tempo, além de observar a sazonalidade dos dados. Foi utilizado a biblioteca `statsmodels` para fazer a decomposição sazonal dos dados. E foi observado que os dados **não possuem sazonalidade nem tendência definidos**. caracteristica essa se deve a própria natureza dos dados econômicos
<ul>

- Foi utilizada o modo de decomposição aditivo;
- Dentro da pasta [images](https://github.com/lavinomenezes/Desafio_DS_Series_Temporais/tree/main/images) estão todas as imagens geradas durante a análise de decomposição sazonal dos dados.
</ul>

5. **Teste de estacionariedade:** Para observar se os dados são estacionários foi utilizado o teste de Dickey-Fuller aumentado 
<ul>

- O teste de Dickey-Fuller aumentado é um teste de hipótese onde a hipótese nula é que a série temporal possui uma raiz unitária, ou seja, que a série não é estacionária. A hipótese alternativa (rejeitada se o valor-p for menor que o nível de significância) é que a série temporal é estacionária. 
- Foi utilizado a biblioteca `statsmodels` para fazer o teste de Dickey-Fuller aumentado. 
- E foi observado que pelo teste que **81% dos dados são estacionários, e 19% não são estacionários.** Mas pela natureza dos dados econômicos, foi optado por continuar com a modelagem considerando **todos os dados como estacionários.**

</ul>

6. **Modelagem dos dados:** Para a modelagem dos dados foram utilizados 3 modelos de time series diferentes, sendo eles:
<ul>    

- Simple Exponential Smoothing
- auto_arima 
- Facebook Prophet

Todos os modelos foram treinados com os dados de 1980 a 2023 e testados com os dados de 2024 a 2028, para observar a acurácia de cada modelo.

Ao final o modelo escolhido foi o **Simple Exponential Smoothing**, pois foi o modelo que obteve a melhor acurácia, além de ser o modelo mais simples e rápido de ser treinado.
</ul>



7. **Métricas de performance:** Para observar a performance de cada modelo foram utilizadas três métricas de avaliação, sendo elas:
<ul>

- MAE (Erro Médio Absoluto):
    O Erro Médio Absoluto (MAE) é uma métrica simples e intuitiva que mede a média das diferenças absolutas entre as previsões e os valores reais. Em outras palavras, ele calcula o tamanho médio dos erros de previsão sem considerar sua direção. O MAE é útil quando você deseja entender o tamanho médio dos erros de previsão em unidades originais. Ele não é sensível a outliers e é fácil de interpretar.

- RMSE (Raiz do Erro Quadrático Médio):
    O Erro Quadrático Médio (RMSE) é uma métrica que mede a raiz quadrada da média dos erros ao quadrado entre as previsões e os valores reais. O RMSE penaliza erros maiores mais do que erros menores devido à natureza quadrática do cálculo. Ele fornece uma ideia da dispersão dos erros e é mais sensível a outliers do que o MAE. O RMSE é útil quando você deseja ter uma noção da variação dos erros.

- MAPE (Erro Percentual Médio Absoluto):
    O Erro Percentual Médio Absoluto (MAPE) é uma métrica que mede a média das porcentagens absolutas dos erros em relação aos valores reais. Ele expressa os erros relativos como uma porcentagem do valor real, tornando-o útil para entender a precisão relativa do modelo em diferentes escalas de valores. No entanto, o MAPE pode ser problemático quando os valores reais são muito próximos de zero, pois isso pode levar a divisões por zero ou valores extremamente altos.

Sendo o MAE como métrica principal pelo fácil entendimento e o RMSE e MAPE como métricas secundárias.

</ul>



8. **Fine tuning:** Para isso foi utilizado a função de ParameterGrid do sklearn, que permite a criação de um grid de parâmetros para serem testados em um modelo. e o fine tuning foi feito para cada país e região, assim cada um possui um modelo com os melhores parâmetros para ele.

# Performance dos modelos

Para este projeto foi utiliza o MAE como métrica principal, pois é uma métrica de fácil entendimento e interpretação, além de ser menos sensível a outliers.

Como era necessário verificar a performace para cada país e região, cada um era treinado individualmente, e as métricas eram avaliadas para cada um. em seguida os resultados eram salvos em um dataframe para serem comparados. ao final era feito uma analise estatística dos resultados para observar a performance dos modelos. e foi decido pela média do MAE para todos os paises e regiões para comparar os modelos.

Para os valores de MAE:
| Model            | mean     | median   | std      | min      | max       | cv       |
|------------------|----------|----------|----------|----------|-----------|----------|
| prophet          | 1.642346 | 1.003945 | 1.961299 | 0.055700 | 18.717070 | 1.194206 |
| simple_smoothing | 1.288721 | 0.697400 | 2.617642 | 0.019480 | 27.734450 | 2.031194 |
| auto_arima       | 1.482224 | 0.898300 | 2.156974 | 0.000000 | 20.569550 | 1.455228 |

RMSE:

| Model            | mean     | median   | std      | min      | max       | cv       |
|------------------|----------|----------|----------|----------|-----------|----------|
| prophet          | 1.726555 | 1.032785 | 2.052355 | 0.080230 | 19.187040 | 1.188699 |
| simple_smoothing | 1.379727 | 0.745805 | 2.881058 | 0.019480 | 29.500960 | 2.088136 |
| auto_arima       | 1.598677 | 0.971070 | 2.431278 | 0.000000 | 24.303880 | 1.520806 |

MAPE:
| Model            | mean      | median    | std       | min      | max        | cv       |
|------------------|-----------|-----------|-----------|----------|------------|----------|
| prophet          | 57.816669 | 32.222000 | 76.873557 | 0.936100 | 706.453190 | 1.329609 |
| simple_smoothing | 46.013746 | 22.037015 | 82.442300 | 0.695720 | 687.014190 | 1.791688 |
| auto_arima       | 46.247781 | 31.770445 | 54.678141 | 0.000000 | 562.484240 | 1.182287 |

O modelo com a melhor performance foi o **Simple Exponential Smoothing**, pois obteve a menor média de MAE, RMSE e MAPE, além das outras métricas estatisticas mostram que os os resultados estaõ consistentes. 

Mas para uma melhor analise de performace foi feito um cross validation para os modelos auto_arima e simple_smoothing, para observar se os resultados se mantinham consistentes.

## cross validation 

Para o cross validation fou utilizada a ferramenta TimeSeriesSplit do sklearn, que permite fazer o cross validation para dados de series temporais. Aplicado o cross validation para cada país individual e calculado a média das métrica para ao final calcular a média de cada MAE média do cross validation para cada modelo. Assim os resultados foram:

Simple Smoothing:

- Mean MAE for all countrys: 3.75
- Median MAE for all countrys: 3.05
- Std MAE for all countrys: 2.37
- Max MAE for all countrys: 21.96
- Min MAE for all countrys: 1.05
- CV MAE for all countrys: 0.63
 
 distribuição das médias de MAE para o simple smoothing:
 
 ![](https://github.com/lavinomenezes/Desafio_DS_Series_Temporais/blob/main/images/distribuicao_das_medias_para_o_simple_smoothing.png)

Auto Arima:

- Mean MAE for all countrys: 3.61
- Median MAE for all countrys: 3.02
- Std MAE for all countrys: 2.23
- Max MAE for all countrys: 19.5
- Min MAE for all countrys: 1.09
- CV MAE for all countrys: 0.61

distribuição das médias de MAE para o auto arima:
![](https://github.com/lavinomenezes/Desafio_DS_Series_Temporais/blob/main/images/distribuicao_das_medias_para_o_auto_arima.png)

Apesar do auto_arima ter performado melhor que o simple_smoothing, a diferença entre eles é muito pequena, e o simple_smoothing é um modelo mais simples e rápido de ser treinado, além de ser mais fácil de lidar com os parametros, por isso foi escolhido o **simple_smoothing como modelo final**.


