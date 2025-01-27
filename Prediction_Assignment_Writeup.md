---
title: "Prediction Assignment Writeup"
author: "Gaurav"
date: "August 8, 2017"
---


### <br><b> Executive Summary</b></br>
For this assignment I analyzed the provided data to determine what activity an individual perform. To do this I made use of caret and randomForest which allowed me to generate correct answers for each of the 20 test data cases that provided in this assignment. I made use of a seed value for consistent results. 

```{r}
setwd("~/Project/GitHub/datasciencecoursera/Machine_Learning")
options(warn=-1)
suppressWarnings(library(Hmisc))
suppressWarnings(library(caret))
suppressWarnings(library(randomForest))
suppressWarnings(library(foreach))
suppressWarnings(library(doParallel))
set.seed(1024)
```

First, I downloaded the raw data from the URL provided by COURSERA.

```{r}
if (!file.exists("pml-training.csv")) {
    download.file("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", 
                  destfile = "pml-training.csv")
}
if (!file.exists("pml-testing.csv")) {
    download.file("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", 
                  destfile = "pml-testing.csv")
}
```

Then loaded both data from the provided training and test data. Some values contained a "#DIV/0!" that one I replaced with an NA value.

```{r}
training_data <- read.csv("pml-training.csv", na.strings=c("#DIV/0!") )
evaluation_data <- read.csv("pml-testing.csv", na.strings=c("#DIV/0!") )
```

I have casted all data in 8 columns as numeric values.

```{r}
for(i in c(8:ncol(training_data)-1)) {training_data[,i] = as.numeric(as.character(training_data[,i]))}

for(i in c(8:ncol(evaluation_data)-1)) {evaluation_data[,i] = as.numeric(as.character(evaluation_data[,i]))}
```

Some columns were mostly blank. These did not contribute well to the prediction. I chose a feature set that only included complete columns. We also remove user name, timestamps and windows.

Determine and display out feature set.

```{r}
feature_set <- colnames(training_data[colSums(is.na(training_data)) == 0])[-(1:7)]
model_data <- training_data[feature_set]
feature_set
```

We can split our dataset in 2 models data: training and testing.

```{r}
idx <- createDataPartition(y=model_data$classe, p=0.75, list=FALSE )
training <- model_data[idx,]
testing <- model_data[-idx,]
```

Using parallel processing to build the model, we build 5 random forests with 150 trees each.  I found several examples of how to perform parallel processing with random forests in R, this provided a great speedup.

```{r}
registerDoParallel()
x <- training[-ncol(training)]
y <- training$classe

rf <- foreach(ntree=rep(150, 6), .combine=randomForest::combine, .packages='randomForest') %dopar% {
randomForest(x, y, ntree=ntree) 
}
```

####<br><b>Conclusions and Test Data Submit</b>

As can be seen from the confusion matrix this model is very accurate. This test data was around 99% accurate and all of test cases are nearly to be correct.

Prepare the submission. (using COURSERA provided code)

```{r}
pml_write_files = function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("problem_id_",i,".txt")
    write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
  }
}


x <- evaluation_data
x <- x[feature_set[feature_set!='classe']]
answers <- predict(rf, newdata=x)

answers

pml_write_files(answers)
```