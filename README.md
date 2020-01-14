# Programowanie-w-R-Projekt
Określenie głównych przyczyn stopniowego karłowacenia śledzi oceanicznych wyławianych w Europie

### Instalacja niezbędnych paczek
```
install.packages("rmarkdown")
```
### Wczytanie zbioru danych sledzie.csv w programie RStudio
![1](https://user-images.githubusercontent.com/9076417/72377818-7dee6000-3710-11ea-8cdb-eb3a508c3628.png)

![2](https://user-images.githubusercontent.com/9076417/72378442-aa56ac00-3711-11ea-97b6-a37dc20dbc92.png)

![3](https://user-images.githubusercontent.com/9076417/72378505-d1ad7900-3711-11ea-8ab2-4ca2be46f142.png)

### Zastąpienie brakujących danych (NA) wartością średniej atrybutu

```r
for(i in 2:ncol(abalone)){
  abalone[is.na(abalone[,i]), i] <- mean(abalone[,i], na.rm = TRUE)
}
```
