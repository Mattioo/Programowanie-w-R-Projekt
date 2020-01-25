---
output:
  word_document: default
  html_document: default
---
----
title: "Projekt - Zbiór różnych parametrów słuchotek (uchowców) – ślimaków morskich"
author: "Mateusz Korolow"
date: "`r format(Sys.time(), '%d %B, %Y')`"
----

# Spis treści

1. [Wstęp](#1)
2. [Wykorzystane biblioteki](#2)
3. [Wczytanie danych](#3)
4. [Podstawowe statystyki](#4)
5. [Uzupełnienie brakujących danych](#5)
6. [Szczegółowa analiza wartości atrybutów](#6)
7. [Korelacja pomiędzy atrybutami](#7)
8. [Regresor](#8)
9. [Analiza ważności atrybutów](#9)
10. [Klasyfikator](#10)

Wstęp
------
<a name="1"></a>

Poniższy raport ma na celu podjęcie analizy danych opisujących cechy anatomiczne słuchotek.
Znaleziony zbiór (http://archive.ics.uci.edu/ml/datasets/Abalone) jest kompletny, dlatego na jego postawie wygenerowano nowy plik usuwając z niego część danych (~5%). W projekcie wykorzystano zmodyfikowany plik danch dostępny w projekcie na platformie GitHub.Brakujące wartości numeryczne zostały uzupełnione średnimi wartościami dla konkretnych atrybutów, natomiast brakujące informacje o płci uzupełniono wartością najczęściej występującą.

Wykorzystane biblioteki
------
<a name="2"></a>

```{r setSeed, warning=FALSE, message=FALSE, echo=FALSE, include=TRUE}
set.seed(26)
```

```{r libraryLists, warning=FALSE, message=FALSE}
library(knitr)
library(tibble)
library(tidyverse)
library(caret)
library(corrplot)
```

Wczytanie danych
------
<a name="3"></a>

Wczytanie pliku *.csv z danymi i wyznaczenie typu danych

```{r import, cache=FALSE, include=TRUE}
initial <- read.table("abalone_missing.csv", nrows = 4177, sep = ';', comment.char = "", na.strings = '?', header = TRUE)
classes <- sapply(initial, class)
data <- read.table("abalone_missing.csv", sep = ';', comment.char = "", na.strings = '?', header = TRUE, colClasses = classes)
```

Podstawowe statystyki
------
<a name="4"></a>

Wczytano `r nrow(data)` wierszy, które opisują `r ncol(data)` cech.

```{r basicStats, cache=TRUE, include=TRUE}
kable(summary(data))
```

Uzupełnienie brakujących danych
------
<a name="5"></a>

Utworzenie funkcji zajmującej się zastępowaniem wartości NA we wskazanej kolumnie wartością najczęściej w niej występującą.
Wywołanie zdefiniowanej metody dla kolumny pierwszej (płeć słuchotki).

```{r removeNASex, cache=FALSE, include=TRUE}
get_most_common <- function(x){
  return(names(sort(table(x), decreasing = T, na.last = T)[1]))
}
data[is.na(data[,1]), 1] <- get_most_common(data$Sex)
```

Zastąpienie wartości NA (iteracyjnie) dla kolejnych atrybutów ich wartościami uśrednionymi.

```{r removeNANumeric, cache=FALSE, include=TRUE}
for(i in 2:ncol(data)){
  data[is.na(data[,i]), i] <- mean(data[,i], na.rm = TRUE)
}
```

Szczegółowa analiza wartości atrybutów
------
<a name="6"></a>

```{r attr_calculate, echo=TRUE, cache=FALSE}
for(i in 2:ncol(data)){
  plot(c(1:nrow(data)), c(data[,i]) ,type = 'p', xlab = '', ylab = names(data[i]))
}
```

Korelacja pomiędzy atrybutami
------
<a name="7"></a>

```{r cor_calculate, echo=TRUE, cache=TRUE}
correlation_data  <- cor(data[,2:9], use = "pairwise.complete.obs")
corrplot(correlation_data, method = "color", tl.cex = 0.4, order = "FPC", tl.col="black")
```

Możemy zauważyć mocną korelację pomiędzy  długością słuchotki, a wagą jej skorupy. Im dłuższa jest, tym jej skorupa ma większą wagę.

```{r cor_length_shell_calculate, echo=TRUE, cache=FALSE}
plot(data$Length, data$Shell.weight ,type = 'p', xlab = 'Length', ylab = 'Shell weight')
```

Zauważalna jest również zależność pomiędzy wagą muszli, a ilością pierścieni słuchotki. Prawdopodobnie wynika to z wmiarów skorupy
(cięższa muszla jest większa, zatem jest w stanie pomieścić większą ilość pierścieni). Widać również, że liczba pierścieni słuchotki jest ograniczona, ponieważ tendencje wzrostowe wykresu zaczynają maleć wraz z wagą muszli.

```{r cor_shell_rings_calculate, echo=TRUE, cache=FALSE}
plot(data$Shell.weight, data$Rings ,type = 'p', xlab = 'Shell weight', ylab = 'Rings')
```

Regresor
------
<a name="8"></a>

Dla modelu regresji wybieram jedynie najistotniejsze atrybuty: płeć, długość, wysokość, ciężar, ciężar muszli, ilość pierścieni.
Cały zbiór danych dzielę na część testową i treningową w proporcji 1/4 do 3/4 wykorzystując walidację krzyżową. Przewidywanie będzie
dotyczyć ilości pierścini słuchotek.

```{r regresor, warning=FALSE}
rdf <- data %>% select(1, 2, 4, 5, 8, 9)
forTraining <- createDataPartition(y = rdf$Length, p = .75, list = FALSE)

set.seed(123)

trainingSet <- rdf[forTraining, ]
testingSet <- rdf[-forTraining, ]

ctrl <- trainControl(method = "repeatedcv", number = 2,repeats = 3)
fitLm <- train(Length ~ ., data = trainingSet, method = "lm", metric = "RMSE", trControl = ctrl)

lmPredict<-predict(fitLm, newdata=testingSet)
postResample(lmPredict,testingSet$Length)
```

Analiza ważności atrybutów
------
<a name="9"></a>

Analiza ważności atrybutów wykazała, że do szacowania długości słuchotki najlepszy okazuje się parametr informując o ilości pierścieni.

```{r analize, warning=FALSE}
fitLm %>% summary()
```

Klasyfikator
------
<a name="10"></a>

Próba utworzenia modelu klasyfikującego płeć słuchotek. Wykorzystanie algorytmu klasyfikacji random forest i walidacji krzyżowej uzyskując ~ 52% miary accuracy.

```{r classifier, warning=FALSE}
cdf <- data %>% select(1, 2, 4, 5, 8, 9)
forTrainingClassifier <- createDataPartition(y = cdf$Sex, p = .75, list = FALSE)

set.seed(123)

trainingClassifierSet <- rdf[forTrainingClassifier, ]
testingClassifierSet <- rdf[-forTrainingClassifier, ]

ctrl <- trainControl(method = "repeatedcv", number = 2,repeats = 3)

clsf <- train(Sex ~ ., trainingClassifierSet, method = 'rf', TuneLength = 5, trainControl = ctrl)

print(clsf$results)
```

Rezultaty

```{r analizeClassifier, warning=FALSE}
predictedData <- predict(clsf, testingClassifierSet)
table(predictedData, testingClassifierSet$Sex)
```
