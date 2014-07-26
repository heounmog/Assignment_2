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
confusionMatrix(testing$classe, predict(model_rpart, testing))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1286   49  161  172    6
##          B  254  395  269  221    0
##          C  312   44  602   68    0
##          D  254  179  177  354    0
##          E  113  215  229  209  316
## 
## Overall Statistics
##                                         
##                Accuracy : 0.502         
##                  95% CI : (0.489, 0.515)
##     No Information Rate : 0.377         
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.363         
##  Mcnemar's Test P-Value : <2e-16        
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.580   0.4478    0.419   0.3457   0.9814
## Specificity             0.894   0.8513    0.905   0.8745   0.8623
## Pos Pred Value          0.768   0.3468    0.587   0.3672   0.2921
## Neg Pred Value          0.778   0.8974    0.828   0.8638   0.9988
## Prevalence              0.377   0.1499    0.244   0.1740   0.0547
## Detection Rate          0.219   0.0671    0.102   0.0602   0.0537
## Detection Prevalence    0.284   0.1935    0.174   0.1638   0.1839
## Balanced Accuracy       0.737   0.6496    0.662   0.6101   0.9218
```


Whereas, the random forest model shows the accuracy of 98.9%, which means that it is fairly good model.

According to the cross validation, the out of sample error is expected to be less than 1.44% with 95% confidence, because the lower bound of 95% CI for accuracy is 0.9856. So the maximum out of sample error would be 1-0.9856 = 1.44%


```r
model_rf <- train(classe ~., method="rf", data=training)
confusionMatrix(testing$classe, predict(model_rf, testing))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1669    2    0    3    0
##          B   19 1105   13    2    0
##          C    0    9 1017    0    0
##          D    0    0   35  929    0
##          E    0    0    3    0 1079
## 
## Overall Statistics
##                                         
##                Accuracy : 0.985         
##                  95% CI : (0.982, 0.988)
##     No Information Rate : 0.287         
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.982         
##  Mcnemar's Test P-Value : NA            
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.989    0.990    0.952    0.995    1.000
## Specificity             0.999    0.993    0.998    0.993    0.999
## Pos Pred Value          0.997    0.970    0.991    0.964    0.997
## Neg Pred Value          0.995    0.998    0.990    0.999    1.000
## Prevalence              0.287    0.190    0.181    0.159    0.183
## Detection Rate          0.284    0.188    0.173    0.158    0.183
## Detection Prevalence    0.284    0.194    0.174    0.164    0.184
## Balanced Accuracy       0.994    0.992    0.975    0.994    1.000
```

So, I decide to select random forest model finally and use it to predict the class of 20 cases.

Next, I load the testing set file of 20 cases, and predict the classe of them with random forest model above.


```r
test_case <- read.csv("pml-testing.csv")
predict(model_rf, test_case)
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
