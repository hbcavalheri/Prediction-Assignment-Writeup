---
title: "Prediction Assignment Writeup"
output:
  pdf_document: default
  
---

Briefly, using data gathered by tracking activity devices, such as accelerometers on the belt, forearm, arm, and dumbell of 6 participants, the goal of this project is to predict how well these devices determine what type of exercise users did. 

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Loading data
In this section, I downloaded the two data sets: training and testing. 

```{r}
download.file(url = "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv",destfile = "./pml-training.csv", method = "curl")
training<-read.csv("./pml-training.csv", na.strings=c("NA","#DIV/0!",""))
download.file(url = "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv",destfile = "./pml-testing.csv", method = "curl")
testing<-read.csv("./pml-testing.csv", na.strings=c("NA","#DIV/0!",""))
```

## Data Processing
The training data set has many columns containing only NAs. So, I removed this columns and, then, excluded the columns that are time-series or are non numeric. Next, I did the same for the testing data set.

```{r}
remove.na<-sapply(training,function(x)any(is.na(x)|x == ""))
training.new<-training[,!remove.na]
training.new<-training.new[,-c(1:7)]

remove.na1<-sapply(testing,function(x)any(is.na(x)|x == ""))
testing.new<-testing[,!remove.na1]
testing.new<-testing.new[,-c(1:7)]
```

## Creating cross validation data set
Here, I split the "training.new" data set into 60% training and 40% probing test.

```{r}
library(caret)
set.seed(12345) 
partition<-createDataPartition(training.new$classe,p=0.6,list=FALSE)
training2<-training.new[partition,]
training2$classe<-as.factor(training2$classe)
probe<-training.new[-partition,]
probe$classe<-as.factor(probe$classe)
```

## Training the model
Here, I am using random forest from "caret" package.
Accuracy is 98% and out of bag error is less than 1%.
```{r}
set<-trainControl(method="cv", 5)
mod.fit<-train(classe~., data=training2,method="rf",trControl=set,
               ntree=250)
mod.fit
varImp(mod.fit)
mod.fit$finalModel
```

## Cross validation
Accuracy is 99%.
```{r}
pred<-predict(mod.fit,probe)
confusionMatrix(pred,probe[,"classe"])
```

## Predicting with testing data set

```{r}
pred.f<-predict(mod.fit,testing.new)
pred.f
```
