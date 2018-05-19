# working_with_axes
codice per personalizzare il formato delle assi nei grafici 

---
title: "working_with_axes"
author: "Elia Vangi"
date: "02 maggio 2018"
output:
 github_document:
    toc: TRUE
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE,warning = F,fig.align = "center",fig.width = 5,fig.height = 3)
```

Alcune funzioni per agire sulle assi:
## I dati

Lavoreremo col database `Iris`

```{r}
library(ggplot2)
library(tidyverse)
library(gridExtra)
h <-  ggplot()+
  geom_density(data=iris,aes(x=Petal.Length,fill=Species))+theme_bw()
h

```

## Invertire gli assi

```{r}
h+coord_flip()
```

# Variabili discrete

## Cambiare manualmente gli ordini di variabili discrete

```{r}
g <- ggplot(iris,aes(Species,Petal.Length,color=Species))+geom_boxplot()+theme_bw()
g
g+scale_x_discrete(limits=c("versicolor","virginica","setosa"))
```

## Invertire l'ordine di variaboli discrete

```{r}
#ottenere i nomi della variabile "Species"
lev <- levels(iris$Species)
lev
#invertire l'ordine
rlev <- rev(lev)
rlev
g+scale_x_discrete(limits=rlev)
```

## Impostare i nomi delle variabili discrete

```{r fig.height=5}
#i nomi solitamente vengono presi direttamente dal dataframe dei dati, ma possono essereimpostati manualmente
a <- g+scale_x_discrete(breaks=c("setosa","versicolor","virginica"),
                   labels=c("set","vers","virg"))
#nascondere i nomi
b <- g+theme(axis.ticks = element_blank(),axis.text = element_blank())+theme_dark()
#nascondere i nomi e la relativa griglia
c <- g+scale_x_discrete(breaks=NULL)+theme_dark()
grid.arrange(a,b,c,nrow=3)
```

# Variabili continue

## Cambiare i limiti di una variabile continua

```{r}
#espandere i limiti di y
a <- g+expand_limits(y=0)+theme(legend.position = "none")
b <- g+expand_limits(y=c(0,8))+theme(legend.position = "none")
#si puo impostare anche con la funzione "ylim(0,8)"
c <- g+ylim(0,8)+theme(legend.position = "none")
#la funzione "scale_y_continuos" è analoga a "ylim" ma sovrascrive le impostazioni di quest'ultimo, quindi in presenza di tutte e due le funzioni prevale la prima
d <- g+ylim(0,8)+scale_y_continuous(limits = c(0,7))+theme(legend.position = "none")
grid.arrange(a,b,c,d,ncol=2,nrow=2)
```
con i metodi illustrati sopra, i valori di y che non ricadono nel range impostato vengono ignorati.
`coord_cartesian` risolve questo problema perchè imposta le dimensione di tutta l'area di vista del grafico

```{r}
 g+ylim(2,7)#qui i dati vengono eliminati, infatti spariscono anche dalla legenda
 g+coord_cartesian(ylim = c(2,7))#qui i dati vengono conservati. la funzionepraticamente fa uno  "zoom" sull'area impostata
 g+coord_cartesian(ylim = c(2,7),xlim = c(2,3))#nota che anche se l'asse x è una variabile discreta e una stringa di testo la funzione mostra comunque la parte di grafico richiesta
```

## Specificare l'intervallo di una variabile continua

```{r}
#assegnare ad un range di valori x-y un intervallo scelto
g+scale_y_continuous(breaks = seq(0,7,0.5))# con 0-7 il range sulle y e 0.5 l'intervallo
```

## Invertire i valori degli assi

```{r}
g+scale_y_reverse()#solo per variabili continue
```

## impostare e nascondere i valori sulle assi
```{r}
species_group <- group_by(iris,Species)
species_group <- summarise(species_group,mean_petal.length=mean(Petal.Length),median_petal.length=median(Petal.Length))
```

```{r}
species_group
```

```{r}
#impostare l'intervallo sulle y
g+scale_y_continuous(breaks = seq(1,10,1/4))+geom_hline(data=species_group,aes(yintercept=mean_petal.length,color=Species),size=0.3)
#impostare manulamente l'intervallo
g+scale_y_continuous(breaks =species_group$median_petal.length)#impostare i valoriin base ad un'espressione
g+scale_y_continuous(breaks =c(1.5,2.5,5,5.75,6))#impostare manualemente i valori con un vettore
```

## Trasformazioni delle assi (log,sqrt,ecc.)

Creiamo i dati che ci serviranno

```{r}
set.seed(201)
n <- 100
dat <- data.frame(
  x=(1:n+rnorm(n,sd=5))/20,
  y=2*2^((1:n+rnorm(n,sd = 5))/20)
  )
a <- ggplot(dat,aes(x,y))+geom_point()+theme_bw()
a
```

scaliamo l'asse y a log2

```{r}
library(scales)
b <- a+scale_y_continuous(trans = log2_trans())#scala mantenendo i valori sull'asse y ad intervalli lineari
c<- a+coord_trans(y="log2")
grid.arrange(b,c,nrow=2)
```

## Fissare il rapporto tra gli assi

```{r}
set.seed(202)
dat <- data.frame(x=runif(40,0,10),y=runif(40,0,50))
a <- ggplot(dat,aes(x,y))+geom_point()+theme_bw()
a
a+coord_fixed()#rapporto tra gli assi di defoult=1
a+coord_fixed(ratio = 1/3)#rapporto di 1/3 cioè un'unita sull'asse y è 3 volte più piccola di quella sull'asse x
```


## Settare le intestazioni delle assi

```{r}
#eliminare l'intestazione dalle assi
a+theme(axis.title.x = element_blank(),axis.title.y = element_blank())
#impostare il titolo degli assi
a+ylab("ipsilon")+xlab("ics")
#equivalente a :
a+scale_x_continuous(name = "")+scale_y_continuous(name = "ipsilon")
#impostare il font e il colore del titolo
a+theme(axis.title.x = element_text(face = "bold",colour = 2,size = 15))
#impostare il font, il colore e l'angolo dei valori sulle assi
a+theme(axis.text.x = element_text(angle = 90,size = 10,colour = 2))
```


