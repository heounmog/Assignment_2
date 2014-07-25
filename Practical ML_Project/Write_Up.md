Fitting classification model for Human Activity Recoginition dataset
==================================================================

First, I load the csv file, load necessary library, and set seed for reproducibility.

```r
data <- read.csv("pml-training.csv")
set.seed(233)
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.1.1
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

Second, according to the source page(http://groupware.les.inf.puc-rio.br/har), the key variables for deciding activity class might be those who have x, y, or z cordinate. I decide to select those variables plus the target variable of classe, and make data subset.


```r
col_of_subset <- c(grep("_x$", names(data)), grep("_y$", names(data)), grep("_z$", names(data)), which(names(data) == "classe"))
data <- data[ , col_of_subset]
```

Third, I split the pml-training data into training and test set.


```r
inTrain <- createDataPartition(data$classe, p=0.7, list=F)
training <- data[inTrain, ]
testing <- data[-inTrain, ]
```

Fourth, in terms of model selection, I choose to try rpart tree and random forest model because the target variable is factor variable with 5 level.

The rpart model shows the accuracy of about 50% for test set, which is pretty low.


```r
model_rpart <- train(classe ~., method="rpart", data=training)
```

```
## Loading required package: rpart
```

```
## Warning: package 'e1071' was built under R version 3.1.1
```

```r
vali_rpart <- confusionMatrix(testing$classe, predict(model_rpart, testing))
vali_rpart$overall[1]
```

```
## Accuracy 
##   0.5018
```

The random forest model shows the accuracy of 98.9%, which means that it is fairly good model.

According to the cross validation, the out of sample error is expected to be less than 1.44% with 95% confidence, because the lower bound of 95% CI for accuracy is 0.9856. So the maximum out of sample error would be 1-0.9856 = 1.44%


```r
## model_rf <- train(classe ~., method="rf", data=training)
## vali_rf <- confusionMatrix(testing$classe, predict(model_rf, testing))
## vali_rf$overall[1]
```

Due to the long execution time(more than half an hour), I attach the result directly.

Overall Statistics
                                          
               Accuracy : 0.9886          
                 95% CI : (0.9856, 0.9912)
    No Information Rate : 0.2851          
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.9856          

Next, I load the testing set file of 20 cases, and predict the classe of them with random forest model above.


```r
test_case <- read.csv("pml-testing.csv")
##predict(model_rf, test_case)
```

The predicted results are:

  B ............(omitted)........... B



