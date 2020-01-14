## ➤ Table of Contents

![1](https://user-images.githubusercontent.com/9076417/72381127-1d165600-3717-11ea-983f-91fa400106db.png)
* [➤ Installation](#-installation)

# Programowanie-w-R-Projekt
Określenie głównych przyczyn stopniowego karłowacenia śledzi oceanicznych wyławianych w Europie

### Instalacja niezbędnych paczek
```
install.packages("rmarkdown")
```
### Wczytanie zbioru danych sledzie.csv w programie RStudio
![2](https://user-images.githubusercontent.com/9076417/72377818-7dee6000-3710-11ea-8cdb-eb3a508c3628.png)

![3](https://user-images.githubusercontent.com/9076417/72378442-aa56ac00-3711-11ea-97b6-a37dc20dbc92.png)

![4](https://user-images.githubusercontent.com/9076417/72378505-d1ad7900-3711-11ea-8ab2-4ca2be46f142.png)

### Zastąpienie brakujących danych dot. płci słuchotek (NA) najczęstszą wartością atrybutu

```r
get_most_common <- function(x){
  return(names(sort(table(x), decreasing = T, na.last = T)[1]))
}

abalone[is.na(abalone[,1]), 1] <- get_most_common(abalone$Sex)
```

[![-----------------------------------------------------](https://user-images.githubusercontent.com/9076417/72381127-1d165600-3717-11ea-983f-91fa400106db.png)](#installation)

### Zastąpienie brakujących danych numerycznych (NA) wartością średniej danego atrybutu

```r
for(i in 2:ncol(abalone)){
  abalone[is.na(abalone[,i]), i] <- mean(abalone[,i], na.rm = TRUE)
}
```

![5](https://user-images.githubusercontent.com/9076417/72380564-fa377200-3715-11ea-8707-e3c191d81b8f.png)
