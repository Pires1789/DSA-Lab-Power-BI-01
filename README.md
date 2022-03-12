# DSA Academy Laboratório de Power BI

## Introdução ao Laboratório

A [DSA Academy](https://www.datascienceacademy.com.br) é um portal de cursos e treinamentos online especializado em Ciência de Dados (e todas as suas subcategorias) em português. Para multiplicar o conhecimento no Brasil, o portal disponibiliza alguns cursos gratuitos e, entre eles, o curso *Microsoft Power BI para Data Science, versão 2.0*. Ainda que o curso seja gratuito, ele conta com suporte para o usuário e uma infinidade de exercícios e desafios que compreendem: ETL, DAX e elaboração visual.

Meu primeiro contato com o campo de data visualization ocorreu através do Tableau e ainda que eu tenha prática com Power BI, este curso serve para consolidar e ampliar meu conhecimento da ferramenta, bem como fortalecer skills de análise de dados. 

## Informações sobre o Negócio

O contexto deste desafio é o seguinte: como analista de dados de uma operadora de plano de saúde, precisamos responder através dos dados algumas perguntas levantadas pela direção da empresa. Os diretores perceberam que os gastos de seguro saúde aumentaram de forma consideração e precisam monitorar a evolução dos gastos.

As perguntas elencadas foram:

1. Qual o gasto total da operadora?
2. Qual a idade média dos usuários da operadora?
3. Qual o gasto médio por região?
4. Qual faixa etária possui maior gasto com seguro saúde por região?
5. Crianças tem gasto maior que adultos?
6. Qual a proporção de crianças por região?
7. O aumento da idade influencia no imc?
8. Quem tem maior gasto, homens ou mulheres?
9. Se o usuário for mulher, o imc é acima ou abaixo da média?

## Dataset

### Conhecendo os dados
Não há informações sobre a origem dos dados e, portanto, não podemos aferir que os dados são reais (retirados do [Kaggle](www.kaggle.com)). 
Os dados foram enviados em formato CSV.

Em todo o caso, a base de dados considera o seguinte:


| Campos do Dataset | Descrição
|-------------------| ---------- |
| Idade | Idade do beneficiário |
| Sexo | Sexo do beneficiário |
| IMC | IMC do títular do plano de saúde |
| Criança? | Quantos filhos o beneficiário possui |
| Fumante? | O beneficiário é fumante?
| Região  | Qual a região em que o beneficiário está localizado |
| Valor do Seguro Saúde | O valor total gasto pela operadora |

### Identificação de problemas e soluções

Ao explorar os dados, identificamos:

a. três beneficiários não possuíam informações na coluna "Sexo";
b. um beneficiário não informou se é ou não "Fumante";
c. dois beneficiários não possuía informação sobre a "Região";

Nossa opção foi a de incluir o marcador "n/a" nos campos supracitados e, nos gráficos que utilizamos essas dimensões como eixo principal, retiramos o "n/a" através do filtro nativo do Power BI. Optamos por esta solução porque entendemos que a necessidade da liderança é compreender melhor o cenário do custo e assim estamos trazendo todo o custo da empresa. Se retirássemos as linhas, esse KPI não estaria 100% completo.

identificamos ainda algumas diferenças no formato dos números (IMC e Valor do Seguro Saúde). Realizamos as limpezas necessárias no próprio arquivo CSV.

## Dashboard

**Visão Gerencial**

![Visão Gerencial](https://user-images.githubusercontent.com/81444128/158016361-5e6046df-4e9f-43a7-b07f-ec736d75dd52.png)


**Perfil do Usuário**


![Perfil do Usuário](https://user-images.githubusercontent.com/81444128/158016011-4e774928-79ca-4b8c-96b0-45ef755ef7f7.png)

### Campos calculados

Na nossa estrutura, optamos por identificar os campos calculados da seguinte forma:

a. aux_nome-do-campo: campos auxiliares que indicam segmentações específicas para gráficos específicos;
b. kpi_nome-do-campo: principal indicador do negócio

Ainda que algumas regras pudessem ser feitas direto com a tabela (por exemplo, o KPI do Gasto Total), por boa prática incluimos um campo calculado. Dessa forma, todo o time de análise de dados poderá identificar com mais agilidade qual a lógica que aplicamos no desenvolvimento do dashboard.

O campo **KPI** desenvolvido foi:

a. **kpi_gasto-total**: somatório do total do valor gasto com o seguro

```

kpi_gasto-total = sum(seguro_saude_v2[valor_seguro_saude])

```

Os campos **auxiliares** desenvolvidos foram:

a. **aux_beneficiarios-com-filhos**: segmentação dos beneficiários que possuem ou não filhos

```
aux_beneficiarios-com-filhos = SWITCH(
    TRUE(),
    seguro_saude_v2[crianca] > 0, "Com Filhos",
    seguro_saude_v2[crianca] = 0, "Sem Filhos"
)

```

b. **aux_faixa-etaria**: criamos grupos de faixa etária para facilitar a organização do histograma

```
aux_faixa-etaria = SWITCH(
    TRUE(),
    seguro_saude_v2[idade] < 20, "0-19",
    seguro_saude_v2[idade] < 29, "20-29",
    seguro_saude_v2[idade] < 30, "30-39",
    seguro_saude_v2[idade] < 40, "40-49",
    seguro_saude_v2[idade] < 50, "50-59",
    seguro_saude_v2[idade] < 60, "60-69",
    seguro_saude_v2[idade] < 70, "70-79",
    seguro_saude_v2[idade] < 80, "80-89"
)

```

c. **aux_genero**: segmentação por gênero "Masculino" ou "Feminino"

```
aux_genero = SWITCH(
    TRUE(),
    seguro_saude_v2[sexo] = "masculino", "Masculino",
    seguro_saude_v2[sexo] = "feminino", "Feminino"
)

```

d. **aux_imc-feminino**: média do IMC feminino

```
aux_imc-feminino = CALCULATE(
    AVERAGE(seguro_saude_v2[imc]),
    FILTER(seguro_saude_v2, seguro_saude_v2[aux_genero] = "Feminino")
)
```

e. **aux_imc-masculino**: média do IMC masculino

```
aux_imc-masculino = CALCULATE(
    AVERAGE(seguro_saude_v2[imc]),
    FILTER(seguro_saude_v2, seguro_saude_v2[aux_genero] = "Masculino")
)
```

f. **aux_imc-obesidade**: número do IMC que a sociedade médica compreende como o início da obesidade. 

```
aux_imc-obesidade = 30

```

g. **aux_total-beneficiarios**: quantos beneficários foram enviados na amostra. Neste caso, aplicamos uma coluna de Índice 

```

aux_total-beneficiarios = MAX(seguro_saude_v2[Índice])

```

## Insights do business

Em nosso laboratório, respondemos a todas as perguntas de negócio levantadas pelos stakeholders. Neste segmento do nosso repositório estamos elencando apenas alguns insights que julgamos mais interessantes.

- A Região Norte consome 271 Bi do orçamento da empresa e os clientes entre 60-69 anos são os que consomem mais recursos da operadora. Contrariando o senso comum, os clientes com mais de 70 anos não consomem tanto o plano de saúde;
- O perfil de gasto por idade é bastante parecido nas Regiões Norte, Nordeste e Sul. Na Região Sudeste, porém, os pacientes entre 20-29 são os que mais consomem o plano de saúde;
- Não há uma diferença significativa entre o valor gasto por homens e mulheres;
- O IMC médio dos segurados indica obesidade em grau leve;
- A obesidade está presente nos usuários com mais de 40 anos. Sabemos que a obesidade é fator de risco para várias doenças e consequentemente o beneficiário usará mais o plano de saúde. Talvez seja interessante ter um programa de combate a obesidade;
- Beneficiários com 1 ou mais filhos representam 60% do gasto da empresa;
