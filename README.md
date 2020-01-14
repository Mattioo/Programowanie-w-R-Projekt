## ➤ Spis treści

![1](https://user-images.githubusercontent.com/9076417/72381127-1d165600-3717-11ea-983f-91fa400106db.png)
* [➤ Instalacja wtyczek](#-instalacja)
* [➤ Uzupełnienie brakujących danych](#-removeNA)

## ➤ Instalacja wtyczek
[![-----------------------------------------------------](https://user-images.githubusercontent.com/9076417/72381127-1d165600-3717-11ea-983f-91fa400106db.png)](#instalacja)

```r
install.packages("rmarkdown")
```

## ➤ Uzupełnienie brakujących danych
[![-----------------------------------------------------](https://user-images.githubusercontent.com/9076417/72381127-1d165600-3717-11ea-983f-91fa400106db.png)](#removeNA)

### Zastąpienie brakujących danych dot. płci słuchotek (NA) najczęstszą wartością atrybutu
```r
get_most_common <- function(x){
  return(names(sort(table(x), decreasing = T, na.last = T)[1]))
}

abalone[is.na(abalone[,1]), 1] <- get_most_common(abalone$Sex)
```

### Zastąpienie brakujących danych numerycznych (NA) wartością średniej danego atrybutu
```r
for(i in 2:ncol(abalone)){
  abalone[is.na(abalone[,i]), i] <- mean(abalone[,i], na.rm = TRUE)
}
```
