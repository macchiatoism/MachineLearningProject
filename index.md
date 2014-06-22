Coursera Practical Machine Learning Project Report
========================================================

## Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: [http://groupware.les.inf.puc-rio.br/har](http://groupware.les.inf.puc-rio.br/har) (see the section on the Weight Lifting Exercise Dataset). 

## Datasets

>- source : [http://groupware.les.inf.puc-rio.br/har]( http://groupware.les.inf.puc-rio.br/har)
>- The training data for this project are available here: [https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv)
>- The test data are available here: [https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv)

## Goal
>- The goal of this project is to predict the manner in which they did the exercise ("classe" variable in the training set or the other variables to predict). 
>- should create a report describing how you built your model, how you used cross validation, what you think the expected out of sample error is, and why you made the choices you did. 
>- You will also use your prediction model to predict 20 different test cases. 
>- should also apply your machine learning algorithm to the 20 test cases available in the test data above
...The packages we use here include caret, MASS, plyr, knitr and e1071. 

```r
# setwd("~/R WS/MachineLearningProject/MachineLearningProject") # set current workspace, edit here if any differences
if (!require("bst",character.only = TRUE)){
  install.packages("bst",dep=TRUE)}
```

```
## Loading required package: bst
## Loading required package: rpart
## Loading required package: gbm
## Loading required package: survival
## Loading required package: splines
## Loading required package: lattice
## Loading required package: parallel
## Loaded gbm 2.1
```

```r
if (!require("caret",character.only = TRUE)){
  install.packages("caret",dep=TRUE)}
```

```
## Loading required package: caret
## Loading required package: ggplot2
## 
## Attaching package: 'caret'
## 
## Das folgende Objekt ist maskiert from 'package:survival':
## 
##     cluster
```

```r
if (!require("e1071",character.only = TRUE)){
  install.packages("e1071",dep=TRUE)}
```

```
## Loading required package: e1071
```

```r
if (!require("plyr",character.only = TRUE)){
  install.packages("plyr",dep=TRUE)}
```

```
## Loading required package: plyr
```

```r
if (!require("MASS",character.only = TRUE)){
  install.packages("MASS",dep=TRUE)}
```

```
## Loading required package: MASS
```

```r
if (!require("knitr",character.only = TRUE)){
  install.packages("knitr",dep=TRUE)}
```
We start with data training process.
Cross-validation is a model validation technique used to evaluate how the results of a statistical analysis can be generalized to a set of independent data. Notice that in a prediction problem it is normal to have a set of data you can develop your model with (training data) and another set, treated like unknown data, you can test your model with (testing data).

```r
trainedData <- read.csv("pml-training.csv") # import training data
NAs <- apply(trainedData,2,function(x) {sum(is.na(x))}) 
validData <- trainedData[,which(NAs == 0)]
trainedIndex <- createDataPartition(y = validData$classe, p=0.8,list=FALSE)
trainData <- validData[trainedIndex,]
testData <- validData[-trainedIndex,]
removeIndex <- grep("timestamp|X|user_name|new_window",names(trainData))
trainData <- trainData[,-removeIndex]
```
...In this project, you can use an important set of data (with an high number of cases) and you will know its classification by the proposed categories (A,B,C,D,E). To do a cross-validation, you need to divide the data in two groups: on one side you will have the training data and on the other side you will have the testing data. So,you  can understand how good your model is.
...To split the data I used the caret function "createDataPartition", and the datas has been  splitted in various size to test more than one option, in particular I used training data=80% of the total data. 

```r
dataModel <- train(trainData$classe ~  gyros_arm_x + gyros_arm_y + gyros_arm_z+
                     accel_arm_x + accel_arm_y + accel_arm_z +
                     gyros_dumbbell_x + gyros_dumbbell_y + gyros_dumbbell_z + 
                     accel_dumbbell_x + accel_dumbbell_y + accel_dumbbell_z +
                     gyros_forearm_x + gyros_forearm_y + gyros_forearm_z + 
                     accel_forearm_x + accel_forearm_y + accel_forearm_z,data = trainData,preProcess=c("center","scale"),
                   maximize=TRUE,method = 'lda', verbose=TRUE)
```
...For training stage it has been used caret R package, in particular train function. This function allows different configurations and several training and preprocessing  methods, for example:
-  Bayesian generalized linear model - bayesglm.
-	CART - rpart.
-	Linear discriminant analysis - lda. 
-	Naive bayes - nb.
-	Random forest - rf.  
-	k-nearest neighbors - knn

...We use Linear discriminant analysis (method='lda'), see [http://stat.ethz.ch/R-manual/R-patched/library/MASS/html/lda.html](http://stat.ethz.ch/R-manual/R-patched/library/MASS/html/lda.html) for more details. 

```r
NAs <- apply(testData,1,function(x) {sum(is.na(x))})
validTest <- testData[which(NAs == 0),]
print(confusionMatrix(predict(dataModel,validTest),validTest$classe))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction   A   B   C   D   E
##          A 726 154 325 113 228
##          B  42 330  26  79 181
##          C  84  73 185  61  91
##          D 190  83  87 351 128
##          E  74 119  61  39  93
## 
## Overall Statistics
##                                         
##                Accuracy : 0.43          
##                  95% CI : (0.414, 0.445)
##     No Information Rate : 0.284         
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.269         
##  Mcnemar's Test P-Value : <2e-16        
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.651   0.4348   0.2705   0.5459   0.1290
## Specificity             0.708   0.8963   0.9046   0.8512   0.9085
## Pos Pred Value          0.470   0.5015   0.3745   0.4184   0.2409
## Neg Pred Value          0.836   0.8686   0.8545   0.9053   0.8224
## Prevalence              0.284   0.1935   0.1744   0.1639   0.1838
## Detection Rate          0.185   0.0841   0.0472   0.0895   0.0237
## Detection Prevalence    0.394   0.1677   0.1259   0.2139   0.0984
## Balanced Accuracy       0.679   0.6656   0.5875   0.6985   0.5187
```
...With the 20 sets of testing data and "predict" Caret-function we can make an idea of the accuracy of the model constructed with the training data.

```r
test_clase <- read.csv("pml-testing.csv")
prediction <- predict(dataModel,test_clase)
```
Below is the results of predictions of the 20 tests datasets.

```r
results<-as.data.frame(cbind(1:20,as.character(prediction)))
names(results)<-(c("Test No.","Clase"))
results
```

```
##    Test No. Clase
## 1         1     D
## 2         2     A
## 3         3     A
## 4         4     A
## 5         5     A
## 6         6     A
## 7         7     D
## 8         8     D
## 9         9     A
## 10       10     A
## 11       11     A
## 12       12     A
## 13       13     B
## 14       14     A
## 15       15     A
## 16       16     B
## 17       17     A
## 18       18     E
## 19       19     B
## 20       20     B
```
