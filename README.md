# painelpopulacionalibge
Montando painel com dados dos Censos 2000, 2010 e 2022, conjuntamente com as Estimativas de população enviadas ao TCU do IBGE


---
title: "populacao_censo(estimativas)"
author: "Joao Erick Costa"
date: "2024-12-12"
output: html_document
---


Declarando diretorio

```{r}
setwd("C:/Users/Erick/OneDrive/Arquivos Computador(C) Joao/1- FJP/inputs/censos/popcenso(estimativas)")

```

Pacotes necessarios:

```{r}
library(sidrar)
library(dplyr)
library(tidyverse)
library(gt)
library(ggplot2)
library(haven)
```


Baixando população estimada para os Municipios de Minas Gerais:
Site: https://sidra.ibge.gov.br/tabela/6579

Os filtros selecionados nesta tabelas foram
Variavel: Populaçao residente estimada (Pessoas)
Ano: 2001 a 2024 (exceto anos do censo)
Unidade territorial: Municipio (853) So municipios de Minas Gerais

```{r}
popestima = get_sidra (api = "/k/67872011") 

```


Visualizando os nomes corretos das variaveis:


```{R}
colnames(popestima)  

```



Renomeando variaveis para ser compativel com a base de dados snis:

```{r}

# Renomear as colunas, garantindo o uso de backticks para contornar espaços
popestima <- popestima %>% 
  rename(
    nivel_terricod = `Nível Territorial (Código)`,
    nivel_terri = `Nível Territorial`,  
    unid_medcod = `Unidade de Medida (Código)`,
    unid_med = `Unidade de Medida`,
    pop = `Valor`, 
    codigoibge = `Brasil, Unidade da Federação e Município (Código)`,
    municipio = `Brasil, Unidade da Federação e Município`,  
    vari_cod = `Variável (Código)`,
    variavel = `Variável`, 
    ano_cod = `Ano (Código)`,
    ano = `Ano`
  )

# Atribuir os labels corretamente usando 'attr()'
attr(popestima$nivel_terricod, "label") <- "Código do Nível Territorial"
attr(popestima$nivel_terri, "label") <- "Nome do Nível Territorial"
attr(popestima$unid_medcod, "label") <- "Código da Unidade de Medida"
attr(popestima$unid_med, "label") <- "Unidade de Medida"
attr(popestima$pop, "label") <- "População Estimada"
attr(popestima$codigoibge, "label") <- "Código IBGE (Brasil, UF, Município)"
attr(popestima$municipio, "label") <- "Nome do Município"
attr(popestima$vari_cod, "label") <- "Código da Variável"
attr(popestima$variavel, "label") <- "Nome da Variável"
attr(popestima$ano_cod, "label") <- "Código do Ano"
attr(popestima$ano, "label") <- "Ano"

```


```{R}
# Ver o tipo de cada variável do dataframe
sapply(popestima, class)

```

Transformando variaveis que sao chr em numeric:


```{r}

popestima$nivel_terricod <- as.numeric(popestima$nivel_terricod)
popestima$nivel_terri <- as.numeric(popestima$nivel_terri)
popestima$codigoibge <- as.numeric(popestima$codigoibge)
popestima$ano <- as.numeric(popestima$ano)
popestima$unid_medcod <- as.numeric(popestima$unid_medcod)
popestima$unid_med <- as.numeric(popestima$unid_med)
popestima$vari_cod <- as.factor(popestima$vari_cod)
popestima$pop <- as.numeric(popestima$pop)
popestima$ano_cod <- as.numeric(popestima$ano_cod)




# Ver a estrutura após as conversões
sapply(popestima, class)


```


```{R}
saveRDS(popestima, "popestima.RDS")

```



Baixando censo populaçao residente total e urbana do censo 2022 para Municipios de MG

Site: https://sidra.ibge.gov.br/tabela/9923 

Selecionei da seguinte forma os filtros:
Variavel: População residente(Pessoas)
Situação do domicílio: Total e Urbana
Ano: 2022
Nivel Territorial: Municipios (853/5570)

Com base na seleçao desses filtros voce obtem os seguinte api: /k/1505583102



```{R}
popcenso2022 = get_sidra(api = "/k/1505583102") 

```



Visualizando os nomes corretos das variaveis para renomear logo em seguida:


```{R}
colnames(popcenso2022)  

```


Renomeando variaveis para ser compativel com a base de dados snis:

```{r}

# Renomear as colunas
popcenso2022 <- popcenso2022 %>%
  rename(
    nivel_terricod = `Nível Territorial (Código)`,
    nivel_terri = `Nível Territorial`,  
    unid_medcod = `Unidade de Medida (Código)`,
    unid_med = `Unidade de Medida`,
    pop = `Valor`, 
    codigoibge = `Município (Código)`,  # Corrigido
    municipio = `Município`,  # Corrigido
    varcod = `Variável (Código)`,
    var = `Variável`, 
    anocod = `Ano (Código)`,
    ano = `Ano`,
    situacaodomicod = `Situação do domicílio (Código)`,
    tipopop = `Situação do domicílio`
  )

# Atribuindo os labels às variáveis do DataFrame popcenso2022
attr(popcenso2022$nivel_terricod, "label") <- "Código do Nível Territorial"
attr(popcenso2022$nivel_terri, "label") <- "Nome do Nível Territorial"
attr(popcenso2022$unid_medcod, "label") <- "Código da Unidade de Medida"
attr(popcenso2022$unid_med, "label") <- "Unidade de Medida"
attr(popcenso2022$pop, "label") <- "População"
attr(popcenso2022$codigoibge, "label") <- "Código IBGE (Brasil, UF, Município)"
attr(popcenso2022$municipio, "label") <- "Nome do Município"
attr(popcenso2022$varcod, "label") <- "Código da Variável"
attr(popcenso2022$var, "label") <- "Nome da Variável"
attr(popcenso2022$anocod, "label") <- "Código do Ano"
attr(popcenso2022$ano, "label") <- "Ano"
attr(popcenso2022$situacaodomicod, "label") <- "Código da Situação do Domicílio"
attr(popcenso2022$tipopop, "label") <- "Situação do Domicílio"
```



Vamos observar qual o atributo de cada variavel no df popcenso2022

```{r}
# Ver o tipo de cada variável do dataframe
sapply(popcenso2022, class)   #é possivel observar que tudo esta em chr. Nessario transformar quem é numeric em numeric. 

```

deixando variaveis que estao em chr em numeric

```{R}
 
# Transformando as variáveis específicas para numeric
popcenso2022 <- popcenso2022 %>%
  mutate(
    # Remover caracteres não numéricos e transformar em numeric
    pop = as.numeric(gsub("[^0-9]", "", as.character(pop))),
    codigoibge = as.numeric(gsub("[^0-9]", "", as.character(codigoibge))),
    ano = as.numeric(gsub("[^0-9]", "", as.character(ano))),
    unid_medcod = as.numeric(gsub("[^0-9]", "", as.character(unid_medcod))),
    varcod = as.numeric(gsub("[^0-9]", "", as.character(varcod))),
    anocod = as.numeric(gsub("[^0-9]", "", as.character(anocod))),
    situacaodomicod = as.numeric(gsub("[^0-9]", "", as.character(situacaodomicod)))
  )

# Verificando a classe das variáveis após a conversão
sapply(popcenso2022, class)


```






salvando pop censo 2022 municipios de MG:

```{r}
saveRDS(popcenso2022, "popcenso2022.RDS")
```


Dividindo um df para populacao total e urbana:


```{R}
poptotal22 <- popcenso2022 %>%
  filter(tipopop == "Total")
  
popurb22 <- popcenso2022 %>%
  filter(tipopop == "Urbana")

```



Salvando df para populacao urbana e total do censo 2022:

```{r}
saveRDS(poptotal22, "poptotal22.RDS")
saveRDS(popurb22, "popurb22.RDS")

```





Populacao residente censo 2000 e 2010 para municipios de MG. Para selecionar essa tabela, marquei na variavel Situação do domicílio: Total e Urbana. Porque teremos como obter um df com uma coluna para populacao total e uma para populacao urbana.

site: https://sidra.ibge.gov.br/tabela/1552


```{r}
popcenso0010 <- get_sidra(api = "/k/1084591774")

```



Visualizando os nomes corretos das variaveis para renomear logo em seguida:


```{R}
colnames(popcenso0010)  

```




Renomeando e atribuindo label:


```{R}
# Renomear as colunas do DataFrame popcenso0010
popcenso0010 <- popcenso0010 %>%
  rename(
    nivel_terricod = `Nível Territorial (Código)`,
    nivel_terri = `Nível Territorial`,  
    unid_medcod = `Unidade de Medida (Código)`,
    unid_med = `Unidade de Medida`,
    pop = `Valor`, 
    codigoibge = `Município (Código)`,
    municipio = `Município`,  
    varcod = `Variável (Código)`,
    var = `Variável`, 
    anocod = `Ano (Código)`,
    ano = `Ano`,
    situacaodomicod = `Situação do domicílio (Código)`,
    tipopop = `Situação do domicílio`,
    sexocod = `Sexo (Código)`,
    sexo = `Sexo`, 
    formdeclidadecod = `Forma de declaração da idade (Código)`,
    formdeclidade = `Forma de declaração da idade`,
    idadecod = `Idade (Código)`,
    idade = `Idade`
  )

# Atribuindo os labels às variáveis do DataFrame popcenso0010
attr(popcenso0010$nivel_terricod, "label") <- "Código do Nível Territorial"
attr(popcenso0010$nivel_terri, "label") <- "Nome do Nível Territorial"
attr(popcenso0010$unid_medcod, "label") <- "Código da Unidade de Medida"
attr(popcenso0010$unid_med, "label") <- "Unidade de Medida"
attr(popcenso0010$pop, "label") <- "População"
attr(popcenso0010$codigoibge, "label") <- "Código IBGE (Brasil, UF, Município)"
attr(popcenso0010$municipio, "label") <- "Nome do Município"
attr(popcenso0010$varcod, "label") <- "Código da Variável"
attr(popcenso0010$var, "label") <- "Nome da Variável"
attr(popcenso0010$anocod, "label") <- "Código do Ano"
attr(popcenso0010$ano, "label") <- "Ano"
attr(popcenso0010$situacaodomicod, "label") <- "Código da Situação do Domicílio"
attr(popcenso0010$tipopop, "label") <- "Situação do Domicílio"
attr(popcenso0010$sexocod, "label") <- "Código do Sexo"
attr(popcenso0010$sexo, "label") <- "Sexo"
attr(popcenso0010$formdeclidadecod, "label") <- "Código da Forma de Declaração da Idade"
attr(popcenso0010$formdeclidade, "label") <- "Forma de Declaração da Idade"
attr(popcenso0010$idadecod, "label") <- "Código da Idade"
attr(popcenso0010$idade, "label") <- "Idade"


```` 





Vamos observar qual o atributo de cada variavel no df popcenso0010

```{r}
# Ver o tipo de cada variável do dataframe
sapply(popcenso0010, class)   #é possivel observar que todas variavei estao no formato chr. Nessario transformar quem é numeric em numeric. 

```


```{R}
# Transformando as variáveis específicas para numeric
popcenso0010 <- popcenso0010 %>%
  mutate(
    nivel_terricod = as.numeric(as.character(nivel_terricod)),
    unid_medcod = as.numeric(as.character(unid_medcod)),
    pop = as.numeric(as.character(pop)),
    codigoibge = as.numeric(as.character(codigoibge)),
    varcod = as.numeric(as.character(varcod)),
    anocod = as.numeric(as.character(anocod)),
    ano = as.numeric(as.character(ano)),
    situacaodomicod = as.numeric(as.character(situacaodomicod))
  )

# Verificando a classe das variáveis após a conversão
sapply(popcenso0010, class)



```` 


```{R}
saveRDS(popcenso0010, "popcenso0010.RDS")

````


Nessa base de dados popcenso0010 precisamos selecionar apenas variaveis que nos interessa e tambem transformar criar duas variaveis novas, uma para populaçao total e uma para populacao urbana. A variavel tipopop tem duas classificaçoes, uma para total e outra para urbana e o df quadriplicando as observaçoes. Por exemplo, abaete era so para aparecer no ano 2000 e 2010, mas esta aparecendo 2 vezes no ano 2000 e duas vezes no ano 2010, por conta de haver total e urbana. 


```{R}

# Criar o novo dataframe apenas com as linhas onde tipopop é "Total"
popcenso0010tot <- popcenso0010 %>%
  filter(tipopop == "Total")

#Para populacao urbana

popcenso0010urb <- popcenso0010 %>%
  filter(tipopop == "Urbana")

```

Salvando:

```{R}
saveRDS(popcenso0010urb, "popcenso0010urb.RDS")
saveRDS(popcenso0010tot, "popcenso0010tot.RDS")

```





Selecionando apenas variaveis que nos interessa em cada df para montar um painel com os dados das populacoes dos censos 2000, 2010 e 2022, com as estimativas das populaçoes de 2001 a 2024. E tambem a tabela de populacao estimada 

```{r}

#Para populaçoes totais:

#2000 e 2010:

popcen0010tot <- popcenso0010tot %>%
  select(
    ano,
    municipio, 
    codigoibge,
    pop, 
    tipopop
  )

# Para 2022:

popcen22tot <- poptotal22 %>%
  select(
    ano,
    municipio, 
    codigoibge,
    pop,
    tipopop
  )


#Para populacoes urbanas:

popurb0010 <- popcenso0010urb %>%
  select(
    ano,
    municipio, 
    codigoibge,
    pop,
    tipopop
  )

#Para 2022:

popurbcen22 <- popurb22 %>%
  select(
    ano,
    municipio, 
    codigoibge,
    pop,
    tipopop
  )


#Populacao estimada

popest <- popestima %>%
  select(
    codigoibge,
    municipio,
    ano,
    pop 
  )

#Deixando apenas municipios em popest
# Remover observações onde codigoibge é 1 ou municipio é "Brasil"
popest <- popest %>%
  filter(!( (codigoibge == 1 & municipio == "Brasil") |
            (codigoibge == 31 & municipio == "Minas Gerais") ))

```` 


salvando novos df:

```{r}
saveRDS(popcen0010tot, "popcen0010tot.RDS")
saveRDS(popcen22tot, "popcen22tot.RDS")
saveRDS(popurb0010, "popurb0010.RDS")
saveRDS(popurbcen22, "popurbcen22.RDS")
saveRDS(popest, "popest.RDS")
```




Criando df para populacao total (censos 2000, 2010 e 2022) e estimativas dos outros anos. Como tambem para populacao urbana (censos 2000, 2010 e 2022) e estimativas dos outros anos. Teremos dois df


```{r}

#Para populaçao total

#Populacao estimada
popestempi <- popest %>%
  mutate(tipo_pop = "Estimativa") %>%
  select(codigoibge, municipio, ano, tipo_pop, pop)
# Censos 2000 e 2010
popcen0010totempi <- popcen0010tot %>%
  mutate(tipo_pop = "Censo_POP_Total") %>%
  select(codigoibge, municipio, ano, tipo_pop, pop)
#Censo 2022
popcen22totempi <- popcen22tot %>%
  mutate(tipo_pop = "Censo_POP_Total") %>%
  select(codigoibge, municipio, ano, tipo_pop, pop)

# Juntando os DataFrames empilhados
dfpoptotal <- bind_rows(popestempi, popcen0010totempi, popcen22totempi)


#Para populaçao Urbana

#Populacao estimada
popestempi <- popest %>%
  mutate(tipo_pop = "Estimativa") %>%
  select(codigoibge, municipio, ano, tipo_pop, pop)
# Censos 2000 e 2010
popcen0010urbempi <- popurb0010 %>%
  mutate(tipo_pop = "Censo_POP_URB") %>%
  select(codigoibge, municipio, ano, tipo_pop, pop)
#Censo 2022
popcen22urbempi <- popurbcen22 %>%
  mutate(tipo_pop = "Censo_POP_URB") %>%
  select(codigoibge, municipio, ano, tipo_pop, pop)

# Juntando os DataFrames empilhados
dfpopurb <- bind_rows(popestempi, popcen0010urbempi, popcen22urbempi)

```


Salvando o df de populacao total e populacao urbana

```{r}
saveRDS(dfpoptotal, "dfpoptotal.RDS")
saveRDS(dfpopurb, "dfpopurb.RDS")
```
