---
title: "Classifying and Predicting Barbell Lifts from Accelerometer Data"
author: "Phil Silberman"
date: "February 18, 2015"
output: html_document
---
Begin by downloading the data and loading it into R.

```r
testURL <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
trainURL <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
download.file(url = testURL, destfile = "test.csv", method = "curl")
download.file(url = trainURL, destfile = "train.csv", method = "curl")
dat <- read.csv("train.csv", na.strings=c("NA","", "#DIV/0!"))
testing <- read.csv("test.csv", na.strings=c("NA","", "#DIV/0!"))
```
We can remove the irrelevant columns that include, among other things, the name of the subjects and timestamps. 

```r
library(caret); library(randomForest)
dat <- dat[,7:160]
```
I've decided to use Random Forest as my model with 500 trees. In order to do this, we need to remove NA values, else randomForest will not run properly.

```r
gooddata <- apply(!is.na(dat),2,sum) > 19621
dat <- dat[,gooddata]
dim(dat)
```

```
## [1] 19622    54
```
We can split our training set into smaller training and validation sets and create our model.

```r
set.seed(1345)
inTrain <- createDataPartition(y=dat$classe,p=0.7,list=FALSE)
training <- dat[inTrain,]
test <- dat[-inTrain,]
dim(training)
```

```
## [1] 13737    54
```

```r
rf_model <- randomForest(classe~., data=training, ntree=500)
print(rf_model)
```

```
## 
## Call:
##  randomForest(formula = classe ~ ., data = training, ntree = 500) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 7
## 
##         OOB estimate of  error rate: 0.27%
## Confusion matrix:
##      A    B    C    D    E class.error
## A 3906    0    0    0    0 0.000000000
## B    6 2651    1    0    0 0.002633559
## C    0   11 2385    0    0 0.004590985
## D    0    0   14 2237    1 0.006660746
## E    0    0    0    4 2521 0.001584158
```
We can quickly check the relative accuracy of our model by comparing with our in-sample validation set. Then we can estimate the out of sample error.

```r
predictions <- predict(rf_model,test)
table(predictions,test$classe)
```

```
##            
## predictions    A    B    C    D    E
##           A 1674    0    0    0    0
##           B    0 1137    6    0    0
##           C    0    2 1020    2    0
##           D    0    0    0  962    1
##           E    0    0    0    0 1081
```

```r
oose.accuracy <- sum(predictions == test$classe)/length(predictions)
oose.accuracy
```

```
## [1] 0.9981308
```

```r
(1 - oose.accuracy)*100
```

```
## [1] 0.1869159
```
Hence, my estimate for out of sample error is 0.18%. 
