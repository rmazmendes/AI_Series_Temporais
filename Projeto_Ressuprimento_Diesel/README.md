# Título. Previsão de Séries Temporais para Estoque de Diesel em Unidade de Produção de Petróleo
# Subtítulo. Abordagem mediante extração de dados utilizando API, previsão via modelos Random Forest e BILSTM (Bidirectional Long Short-Term Memory) e gravação de alertas, via API, em base de dados para visualização.

#### Aluno: [RENATO MAZONI ANDRADE MARÇA MENDES] (https://github.com/rmazmendes)
#### Orientadora: [PROFESSORA MANOELA KOHLER] (https://github.com/manoelakohler).

---

Trabalho apresentado ao curso [BI MASTER](https://ica.puc-rio.ai/bi-master) como pré-requisito para conclusão de curso e obtenção de crédito na disciplina "Projetos de Sistemas Inteligentes de Apoio à Decisão".

---

### Resumo

Atualmente a rotina de trabalho do apoio operacional, da empresa em questão, para ressuprimento de diesel é executada mediante o método convencional de análise do estoque atual, estoque mínimo e demanda futura por parte dos operadores do processo. O presente projeto tem por objetivo final, para cada uma das 09 (nove) plantas de produção, gerar alertas erro de alimentação de dados de estoque de diesel e alertas de estoque atual ou futuro (12 passos à frente) abaixo do estoque mínimo. O primeiro alerta é usado para interação com os operadores das plantas nos sentido da correção da informação de estoque do dia. O segundo alerta, para provocar ações de checagem e programação do ressuprimento, uma vez que a logística da empresa demanda recursos com altas tarifas diárias e está sujeita também à condições meteoceanográficas.  


### Abstract

Currently, the operational support work routine of the company in question for diesel resupply is carried out using the conventional method of analyzing current stock, minimum stock and future demand by process operators. The final objective of this project is, for each of the 9 (nine) production plants, to generate diesel stock data feed error alerts and current or future stock alerts (12 steps ahead) below the minimum stock. The first alert is used to interact with plant operators to correct the day's stock information. The second alert, to trigger checking actions and resupply scheduling, since the company's logistics demand resources with high daily rates and are also subject to meteoceanographic conditions.


### 1. Introdução

O primeiro desafio foi a obtenção do dados relacionados ao Diesel, um insumo de produção e de geração de energia. O sistema fonte, disponível na intranet da empresa, é alimentado diariamente pelos operadores das 09 plantas de produção sendo que a qualidade da informação, antes deste desenvolvimento, era checada de forma amostral e à medida da percepção humana de erros de alimentação. Desta forma, desenvolvemos um script em python e exploramos inicialmente uma  API desenvolvida por profissional da TI, embora ainda não aplicada em ETLs na minha unidade de produção (que reúne as 09 plantas de produção), obtendo-se sucesso após várias testes. O formato de saída é json e o script versou sobre estabelecer acesso ao API da TI, extrair dados (sempre da última versão do dia liberada pela operação), tratar e verificar problemas de alimentação de dados, gerar alertas de confiabilidade (onde foi utilizada também a métrica z-score) e gravar no sistema PI AF (PI Asset Framework; da empresa AVEVA) os alertas de erros de preenchimento, via PIWEBAPI. O script também produz um relatório final por planta de produção com as informações diárias e mensais dos últimos 2 anos. Finalmente, o script foi agendado para gerar resultados a cada 30 min. 

De posse das informações geradas pelo script acima, foi gerado um segundo script em python para previsão de séries temporais para o estoque atual das plantas de produção, uma vez que os dados de estoque são dependentes do tempo e possuem ciclos. Após testes com uma unidade de produção, ampliamos o script para atender, numa única rodada, as 09 (nove) plantas de produção. Também utilizamos a produção líquida destas plantas, como dado exôgeno, para conferir melhor previbilidade nos resultados. Analisamos graficamente também a possibilidade de utilização de um 2o dado exógeno, o mês, mas este não se mostrou relevante para ser aplicado o script. Finalmente os scripts deste temática adotaram a previsão de estoque com 12 passos à frente. 

Como espera-se, com a recente implantação da rotina de alertas de erros de preenchimento de dados pela operação, que a qualidade dos dados diários de estoque melhore, o script construído irá automaticamente, via agendamento, retreinar o modelo diariamente, um vez que nos testes não se mostrou custoso adotarmos o modelo Random Forest. Por fim o resultado final testado foi a geração de alertas indicando a quantidade de dias restantes para que o estoque fique igual ou abaixo do estoque mínimo.


### 2. Modelagem

Foram utilizados dois modelos: Random Forest e BILSTM, para efeito comparativo. Optamos pelo Random Forest para verificar como este modelo baseado em árvores de decisão se comporta. Também testamos o BILSTM para verificar o comportamento do rede neural, uma variante da RNN (Recurrent Neural Network), particularmente explorando sua capacidade, sua "memória, de capturar padrões complexos e, atendendo interesse do trabalho, analisar os dados em ambas as direções (do passado para o futuro). Ambos os modelos enfrentaram o desafio de proceder previsões frente ao fato de que as plantas de produção apresentam um perfil de variação de estoque, com ciclos não uniformes e sem sazonalidade; ou seja, dados da "vida real".  

Podemos destacar no desenvolvimento do script para os modelo Random Forest e BILSTM:
- Uso intenso de dicionários dado que o objetivo foi criar modelos para 09 (nove) plantas de produção num único script;
- Tratamento de missings;
- Gráficos de estoque e produção líquida versus tempo;
- Gráficos de análise comparativo do estoque para cada mês em diferentes anos;
- Gráficos de análise comparativo do estoque para cada faixa de produção líquida de óleo;
- Gráficos de autocorreção (ACF); 
- Gráficos de PartialAutocorrelation (PACF);
- Definição do lag para cada planta de produção com janelamento personalizado e incluindo o dado exógeno de produção líquida do próximo passo; 
- Utilização das métricas de erro RMSE, MSE, MAPE e R2;
- Predição de um passo à frente;
- Predição de doze passos à frente;  
- Plotagem dos gráficos de cada planta de produção com dados históricos e predição de 12 dias à frente. Foi utilizada a biblioteca streamlit para este fim;
- Requisição post utilizando o piwebapi para gravação de alerta no sistema PI AF;

Podemos destacar no desenvolvimento do script para o modelo Random Forest:
- Adoção da função RandomForestRegressor da biblioteca Scikit Learn;

Podemos destacar no desenvolvimento do script para o modelo BILSTM:
- Normalização utilizando o MinMaxScaler. A normalização foi necessária para melhorar a estabilidade e a performance do treinamento da rede neural. Isso porque redes LSTM são sensíveis à escala dos dados;
- Adoção da função Bidirectional LSTM da biblioteca Tensorflow. Construção de uma rede neural com a seguinte topologia, após testes:
-- 1a camada LSTM bidirecional com 100 neurônios;
-- Dropout de 20%;
-- 2a camada LSTM bidirecional com 80 neurônios;
-- Dropout de 20%;
-- 3a camada LSTM bidirecional com 50 neurônios;
-- Dropout de 20%;
-- 4a camada Dense de 1 neurônio (dada a saída ser uma previsão); 
-- Otimizador Adam e loss MSE; dada a convergência rápida do otimizador e a capacidade de minimização de grandes desvios proporcionada pelo MSE..
- Desnormalização utilizando o inverse_transform de forma a visualizar-se o resultado na escala original



### 3. Resultados

Para entender tanto o perfil gráfico dos estoque bem como os resultados, abaixo é importante ressaltar que a consumo de diesel do tanque principal, onde medimos o estoque em questão, das plantas de produção não se dá continuamente. São feitas operações de transferência para outros tanques menores ou para linhas de produção, conforme a demanda. Geralmente o diesel é demandado para intervenções de condicionamento e limpeza poços, e para necessidade eventual de geração de energia elétrica em geradores a diesel e no uso em guindastes e bombas de combate a incêndio. Assim, por vezes, há situações de alguns dias sem variação de estoque e outros onde há variações bruscas de estoque.

Noutro aspecto para compreensão do perfil do estoque, numa avaliação preliminar dos gráficos de estoque e produção líquida versus o tempo, verificou-se em duas planta um comportamento sensivelmente diferente das demais. Isto se deve ao fato que estas duas plantas entraram em ramp up de produção em meados de 2023. Desta forma o estoque de diesel esteve, antes disso, ligado às atividades de preparação para entrada em produção. Quanto à demais unidades de produção, verificou-se um aumento do estoque após evento de parada de produção motivado, possivelmente, pela demanda de geração com diesel e condicionamento de poços.*

*Da análise comparativa do estoque para cada mês em diferentes anos, vemos que os comportamentos são, de certa forma, díspares; ou seja, não foi verificado correlação do mesmo mês para cada ano (2022, 2023, 2024) e o valor do estoque. Já no gráfico de análise comparativa do estoque para cada faixa de produção líquida de óleo verifica-se, pelo perfil das curvas uma maior semelhança comparando-se 2023 e 2024, sendo que, em 2024, os valores de estoque apresentaram-se, por vezes, maiores que 2023, denotando-se maior consumo (embora existam uma exceção de uma planta de produção onde em 2024 teve menor consumo para as mesmas faixas de produção, dado que boa parte do período a mesma esteve com parada de produção). Outro ponto importante é que o estoque passou a ser melhor preenchido no ritmo diário a partir de 2024 com a implantação de um novo processo de alertas, conforme já mencionado acima.

Utilizamos o gráficos de autocorreção (ACF) para verificar a importância dos dias passados para prever os dias seguintes de forma a auxiliar na definição do janelamento (window). Verificou-se, entre as diferentes plantas de produção, variação no range de valores (valores da abcissa) que são estatisticamente relevantes: variando entre cerca de 8 a 24 valores da abcissa (dias). De forma geral, diminui a correlação com passar do tempo,  diminuindo a significância estatística. Já nos gráficos de PartialAutocorrelation (PACF) (relação direta) verificou-se para todas as plantas de produção alta correção entre Xt-1 e Xt. 
Isto era esperado pois a variação de estoque entre um dia e o dia anterior é fortemente correlacionada pois dada a rotina operacional e que um estoque
é dependente do outro.

Abaixo serão apresentados os resultados. Foram adotadas as métricas RMSE, MSE, MAPE e R2. E analisados comparativamente diversos resultados quanto ao conjunto das métricas. Destacamos abaixo o R2, que mede a variância do dados, onde 1 significa que o modelo explica toda a variância, mais adequado para comparar os resultados entre plantas de produção; menos dependentes da escala. Demais métricas são sensíveis à escala dos valores da série temporal.

Para o modelo Random Forest, inicialmente adotamos, para efeito de teste, uma janela de 14 dias e constatamos o seguintes resultados de R2 comparando a predição e o teste. Observação: inicialmente a janela de 14 dias foi testada na planta 7 (script de uma planta de produção) e, na sequência, mantendo o mesmo parâmetro, foi criado e adotado no script de n plantas de produção. O objetivo foi apenas de comparação.
'Planta_1': R2 = 0.975548592203009
'Planta_2': R2 = 0.9359796259193833
'Planta_3': R2 = 0.7950263069409902
'Planta_4': R2 = 0.7997647599862465
'Planta_5': R2 = 0.8837030748743981
'Planta_6': R2 = 0.5157512153910955
'Planta_7': R2 = 0.9080180420961802
'Planta_8': R2 = 0.872705932986833
'Planta_9': R2 = 0.7999255924604841

Na sequência, para o modelo Random Forest, após verificação das métricas de erro, adotamos a seguinte configuração para o janelamento e, destacamos, os melhores resultados R2 (dentro do universo de testes):
'Planta_1': window = 18; R2 = 0.9857670847576434
'Planta_2': window = 10; R2 = 0.9717160993873933
'Planta_3': window = 18; R2 = 0.7817996614782878
'Planta_4': window = 10; R2 = 0.8385781512960577
'Planta_5': window = 14; R2 = 0.8784860134534392
'Planta_6': window = 3;  R2 = 0.7294153805580821
'Planta_7': window = 14; R2 = 0.9080180420961802
'Planta_8': window = 10; R2 = 0.9229032115398118
'Planta_9': window = 16; R2 = 0.9115438785943308


Na sequência, para o modelo BILSTM, após verificação das métricas de erro, adotamos a seguinte configuração para o janelamento e, destacamos, os melhores resultados R2 (dentro do universo de testes). Adotamos 100 épocas, mediante avaliação do loss e val_loss. Segue:
'Planta_1': window = 18; R2 = 0.9445486253581505; menor que o RF;
'Planta_2': window = 10; R2 = 0.9777784588132314; aprox. igual ao RF;
'Planta_3': window = 18; R2 = 0.8825600150613298; melhor que o RF;
'Planta_4': window = 3; R2 = 0.9083078829812927; melhor que o RF;
'Planta_5': window = 12; R2 = 0.8946513075390413; pouco melhor que o RF;
'Planta_6': window = 10; R2 = 0.7097578429779995; menor que o RF;
'Planta_7': window = 14; R2 = 0.9179018735544565; pouco melhor que o RF;
'Planta_8': window = 10; R2 = 0.885036421157047; menor que o RF;
'Planta_9': window = 12; R2 = 0.9346460587575542; pouco melhor que o RF.


Por fim, foi feito um último teste para o modelo BILSTM. Em se tratando de um modelo LSTM, vamos testar um lag maior para permitir que a rede neural tenha mais informações na sua "memória". Considerando que a partir de março iniciou-se um esforço de melhoria da alimentação dos dados por parte das operações, testamos um lag único de 120 registros (4 meses para trás). Se forma geral, vemos que o lag de 120 dias não produziu, para nosso contexto de dados com ciclos com alguma irregularidades, melhora no resultados em relação ao modelo BILSTM com lags personalizados. 
'Planta_1': R2 = 0.8985195049427095
'Planta_2': R2 = 0.9782655344208397
'Planta_3': R2 = 0.8659870281122526
'Planta_4': R2 = 0.8097162235421128
'Planta_5': R2 = 0.7907765442454076
'Planta_6': R2 = 0.6162545200093489
'Planta_7': R2 = 0.8238321627452105
'Planta_8': R2 = 0.8374424144083684
'Planta_9': R2 = 0.7084541174541314


### 4. Conclusões

Concluímos que os modelos Random Forest e BILSTM se mostraram satisfatórios para a maioria da plantas de produção para atendimento ao objetivo de serem gerados alertas para a equipe de apoio operacional ao ressuprimento de diesel em 09 (nove) plantas de produção. Pelo novo processo, o apoio ainda irá confirmar com as operações a real necessidade e disponibilidade em receber, para o momento, dado que outras variáveis logísticas e de produção são ainda necessárias para decisão final da operação em autorizar o ressuprimento. De qualquer forma, o alerta preditivo do estoque, objetivo deste trabalho, poderá suscitar, com antecedência, atividades de checagem e programação de embarcações para o ressuprimento da plantas, minimizando custos e riscos.

Vimos que, a despeito de perfis de estoque muito diferentes entre as planta de produção, os modelos Random Forest e BILSTM se apresentaram versáteis na adaptação. Com relação à métrica R2 (Coeficiente de Determinação) de avaliação dos modelos, vimos que os modelos Random Forest apresentaram, na avaliação dos resultados de predição versus base de teste, como melhor resultado 0.99, sendo a maior parte das plantas de produção acima de 0,9,  duas plantas acima de 0,8; e exceções para duas plantas que apresentaram 0,73 e 0,78. Já com os modelos BILSTM, o melhor resultado foi 0,98, sendo a maior parte das plantas de produção acima de 0,9, três plantas próximas de 0,9 e uma planta com 0,71. O pior resultado se apresentou na 'Planta_6' em ambos os modelos, possivelmente porque este planta apresentou no seu dataset valores de estoque com ciclos curtos e pouca variação na sua amplitude, bem como períodos sem variação (flat) sendo necessário um janelamento curto. As plantas que apresentaram melhores resultados para ambos os modelos continham períodos longos sem variação de estoque. Como se trata de datasets reais, existe limitação de obtenção de resultados melhores com os dados presentes.

Notamos, dentro deste cenário de testes, que quando o perfil de ciclos de estoque obedece a melhor uniformidade nos ciclos, o modelo Random Forest atende melhor que o BILSTM. Já o BILSTM se apresentou mais aderente num perfil de ciclos mais heterogênios, possivelmente por ser mais adequado para série temporais com padrões complexos e não lineares; embora as diferenças com relação aos resultados do RF sejam relativamente pequenas. 

Apesar o modelo BILSTM apresentar resultados ligeiramente melhores que o modelo Random Forest, iremos adotar o script Random Forest para colocação em produção, dado a performance significativamente melhor deste último. O fator performance é relevante neste momento porque é esperado, com a adoção deste trabalho na frente de apoio operacional, que a alimentação do estoque se torne cada vez mais assertiva ao longo do tempo, com uma melhor uniformidade nos ciclos e, assim, melhorando o dataset e a acurácia do modelo. Assim faz-se necessário adaptar dinamicamente o modelo e, por isso, iremos colocar o modelo Random Forest em produção procedendo seu treinamento uma vez ao dia, via agendamento, e demandando menor recurso computacional. 

Por fim, verifica-se grande potencial para utilização desta metodologia em outros processos envolvendo ressuprimento de insumos de produção e variáveis de produção e de equipamentos com comportamento cíclico.



Matrícula: 221101074

Pontifícia Universidade Católica do Rio de Janeiro

Curso de Pós Graduação *Business Intelligence Master*

