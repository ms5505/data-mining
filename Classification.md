# Data Mining Project - Classification of Indonesian Women's Contraceptive Method
Melinda Song  
12/21/2017  




# Introduction

The dataset I'm using in this project is of contraceptive method choice of Indonesian women. The data was collected as a part of the 1987 National Indonesia Contraceptive Prevalence Survey. There are two variables, one of which is the contraceptive method used, which contains three classes - "no-use", "long-term", and "short-term". This is the response variable that I will attempt to classify using various machine learning methods for classification. 

The remaining variables are the following: 
1. Wife's age (numerical)  
2. Wife's education (categorical) 1=low, 2, 3, 4=high   
3. Husband's education (categorical) 1=low, 2, 3, 4=high   
4. Number of children ever born (numerical)   
5. Wife's religion (binary) 0=Non-Islam, 1=Islam  
6. Wife's now working? (binary) 0=Yes, 1=No  
7. Husband's occupation (categorical) 1, 2, 3, 4  
8. Standard-of-living index (categorical) 1=low, 2, 3, 4=high  
9. Media exposure (binary) 0=Good, 1=Not good  

I obtained this dataset from the UCI Machine Learning Repository. 

# Importing Data 


```r
colnames <- c("wife age", "wife edu", "husband edu", "num chd", "wife religion", "wife work", "husband occu", "std live", "media expo", "cmu")
cmc <- read.table("/Users/melindasong/Downloads/cmc.data", sep = ",", col.names = colnames)
cmc <- data.frame(cmc)
```


```r
cols <- c("wife.edu", "husband.edu", "wife.religion", "wife.work", "husband.occu", "std.live", "media.expo", "cmu")
cmc[cols] <- lapply(cmc[cols], factor)
```


```r
str(cmc)
```

```
## 'data.frame':	1473 obs. of  10 variables:
##  $ wife.age     : int  24 45 43 42 36 19 38 21 27 45 ...
##  $ wife.edu     : Factor w/ 4 levels "1","2","3","4": 2 1 2 3 3 4 2 3 2 1 ...
##  $ husband.edu  : Factor w/ 4 levels "1","2","3","4": 3 3 3 2 3 4 3 3 3 1 ...
##  $ num.chd      : int  3 10 7 9 8 0 6 1 3 8 ...
##  $ wife.religion: Factor w/ 2 levels "0","1": 2 2 2 2 2 2 2 2 2 2 ...
##  $ wife.work    : Factor w/ 2 levels "0","1": 2 2 2 2 2 2 2 1 2 2 ...
##  $ husband.occu : Factor w/ 4 levels "1","2","3","4": 2 3 3 3 3 3 3 3 3 2 ...
##  $ std.live     : Factor w/ 4 levels "1","2","3","4": 3 4 4 3 2 3 2 2 4 2 ...
##  $ media.expo   : Factor w/ 2 levels "0","1": 1 1 1 1 1 1 1 1 1 2 ...
##  $ cmu          : Factor w/ 3 levels "1","2","3": 1 1 1 1 1 1 1 1 1 1 ...
```

The `str()` output verifies that the variables take on the correct forms. 


# Exploratory Data Analysis

In this section, I constructed a few graphs in order to get an understanding of some of the patterns present in the dataset. 



```r
library(ggplot2)
cmu_prop <- cmc %>%
  count(cmu) %>%
  mutate(prop = prop.table(n))

cmu_prop <- as.data.frame(cmu_prop)
cmu_prop$cmu <- factor(cmu_prop$cmu, labels = c("no use", "long-term", "short-term"))

ggplot(data = cmu_prop, aes(x = cmu, y = prop, fill = cmu)) + 
  geom_bar(stat = "identity", position = "dodge") + 
  scale_x_discrete(labels = c("No-use", "Long-term", "Short-term")) +
  xlab("Contraceptive Methods") + 
  scale_y_continuous(labels = percent) +
  labs(y = "Proportions", title = "Contraceptive Use in Indonedia", legend = NULL)
```

![](DM_project_-_ms5505_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

According to the bar graph of the proportion of women who use each of the 3 contraceptive methods, over 40% of Indonesia women surveyed do not use contraceptives at all. This makes sense as Indonesia is a predominately Muslim country. About 35% use short-term contraceptives, and about 22% use long-term. 



```r
cmu_religion <- cmc %>% 
  group_by(wife.religion) %>%
    count(cmu)

cmu_religion$wife.religion <- factor(cmu_religion$wife.religion, labels = c("Non-Muslim", "Muslim"))

ggplot(cmu_religion, aes(x = cmu, y = n, fill = wife.religion)) + geom_bar(stat = "identity") + 
  labs(y = "Count", title = "Contraceptive Use and Wife's Religion", legend = NULL)
```

![](DM_project_-_ms5505_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

As hypothesized above, this graph confirms that most of the survey respondents are Muslims. 



```r
cmu_wedu <- cmc %>%
  group_by(cmu) %>%
  count(wife.edu)

cmu_wedu$wife.edu<- factor(cmu_wedu$wife.edu, labels = c("low", "mid-low", "mid-high", "high"))

ggplot(cmu_wedu, aes(x = wife.edu, y = n, fill = cmu)) + 
  geom_bar(stat = "identity") + 
  labs(x = "Wife's education level", y = "Count", title = "Contraceptive Use and Wife's Education", legend = NULL)
```

![](DM_project_-_ms5505_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


```r
cmu_hedu <- cmc %>%
  group_by(cmu) %>%
  count(husband.edu)

cmu_hedu$husband.edu<- factor(cmu_hedu$husband.edu, labels = c("low", "mid-low", "mid-high", "high"))

ggplot(cmu_hedu, aes(x = husband.edu, y = n, fill = cmu)) + 
  geom_bar(stat = "identity") + 
  labs(x = "Husband's education level", y = "Count", title = "Contraceptive Use and Husband's Education", legend = NULL)
```

![](DM_project_-_ms5505_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

At the lowest education level for husband and wife, most of the respondents do not use contraception. As education level rises, the proportion of respondents who do use some form of contraception goes up as well. At the highest education level, there is a fairly even split among the three contraceptive methods. 



```r
cmu_SOL <- cmc %>%
  group_by(cmu) %>%
  count(std.live)

cmu_SOL$std.live<- factor(cmu_SOL$std.live, labels = c("low", "mid-low", "mid-high", "high"))

ggplot(cmu_SOL, aes(x = std.live, y = n, fill = cmu)) + 
  geom_bar(stat = "identity") + 
  labs(x = "Standard-of-living index", y = "Count", title = "Contraceptive Use and Standard-of-Living", legend = NULL)
```

![](DM_project_-_ms5505_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

Similar to the correlation with education level, the higher the standard-of-living, the more people use contraception.  
 
 

```r
cmu_media <- cmc %>%
  group_by(cmu) %>%
  count(media.expo)

cmu_media$media.expo <- factor(cmu_media$media.expo, labels = c("Good", "Not-good"))

ggplot(cmu_media, aes(x = media.expo, y = n, fill = cmu)) + 
  geom_bar(stat = "identity") + 
  labs(x = "Opinion on media exposure", y = "Count", title = "Contraceptive Use and Opinion on Media Exposure", legend = NULL)
```

![](DM_project_-_ms5505_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

The majority of respondents think that media exposure is good. For this group, about 60% of the respondents use contraception. For the respondents who think that media exposure is not good, most people do not use contraception.



```r
cmu_numchd <- cmc %>%
  group_by(cmu) %>%
  count(num.chd)

ggplot(cmu_numchd, aes(x = num.chd, y = n, fill = cmu)) + 
  geom_bar(stat = "identity") + 
  labs(x = "Number of children ever born", y = "Count", title = "Contraceptive Use and Number of Children", legend = NULL)
```

![](DM_project_-_ms5505_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

For the respondents who do not have kids, it's a bit surprising to see that the vast majority of them do not use contraception. Contraceptive use rises with more kids but the relationship appears to be nonlinear. 


These graphs demonstrate that the data is clearly not uniformly distributed. Some subgroups have far more respondents than others and some relationships are nonlinear. These attributes can make accurate classification difficult. 


# Classification Models

Splitting the dataset into training and testing sets. 


```r
set.seed(24159121)
library(caret)
```

```
## Loading required package: lattice
```

```
## Warning in as.POSIXlt.POSIXct(Sys.time()): unknown timezone 'zone/tz/2018c.
## 1.0/zoneinfo/America/New_York'
```

```r
in_train <- createDataPartition(cmc$cmu, p = 3 / 4, list = FALSE)
training <- cmc[in_train, ]
testing <- cmc[-in_train, ]

testing <- na.omit(testing)
```

Since there are more than two response classes, logistic regression won't work here. I tried the two methods that incorporates Bayes' Theorem. 

### Linear Discriminant Analysis

```r
library(MASS)
```

```
## 
## Attaching package: 'MASS'
```

```
## The following object is masked from 'package:dplyr':
## 
##     select
```

```r
cmc.lda <- lda(cmu ~ . , data = training)
tbl_lda <- table(testing$cmu, predict(cmc.lda, newdata = testing)$class)
sum(diag(tbl_lda)) / sum(tbl_lda)
```

```
## [1] 0.5422343
```

### Quadratic Discriminant Analysis 


```r
cmc.qda <- qda(cmu ~ . , data = training)
tbl_qda <- table(testing$cmu, predict(cmc.qda, newdata = testing)$class)
sum(diag(tbl_qda)) / sum(tbl_qda)
```

```
## [1] 0.4495913
```

LDA performed better than QDA in this case. This could be due to the fact that the training set is fairly small so being able to reduce variance is crucial.   

Next I tried some tree-based models:  

### Regression Tree

```r
library(rpart)
cmc.rpart <- rpart(cmu ~ . , data = training, method = "class")
mean(testing$cmu == predict(cmc.rpart, newdata = testing, type = "class"))
```

```
## [1] 0.5722071
```

### Random Forest

```r
library(randomForest)
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```
## The following object is masked from 'package:dplyr':
## 
##     combine
```

```r
cmc.rf <- randomForest(cmu ~ . , data = training)
mean(testing$cmu == predict(cmc.rf, newdata = testing, type = "class"))
```

```
## [1] 0.5504087
```

### Bagging


```r
cmc.bagging <- randomForest(cmu ~ . , data = training, mtry = 
                          ncol(training) - 1)
mean(testing$cmu == predict(cmc.bagging, newdata = testing, type = "class"))
```

```
## [1] 0.5149864
```

### Boosting

```r
library(gbm)
```

```
## Loading required package: survival
```

```
## 
## Attaching package: 'survival'
```

```
## The following object is masked from 'package:caret':
## 
##     cluster
```

```
## Loading required package: splines
```

```
## Loading required package: parallel
```

```
## Loaded gbm 2.1.3
```

```r
cmc.boost <- gbm(cmu ~ ., data = training, distribution = "multinomial", shrinkage = 0.001, n.cores = parallel::detectCores()) 

cmc.boostPred <- predict(cmc.boost, newdata = testing, type = "response", n.trees = 100)

cmc.boostPred1 <- apply(cmc.boostPred, 1, which.max)

mean(testing$cmu == cmc.boostPred1)
```

```
## [1] 0.5258856
```

Out of the tree-based models, the regression tree performed the best with 57.2% accuracy.


Below are some other models I tried: 

### Nearest-Neighbor Classification


```r
library(knncat)
cmc.knncat <- knncat(training, testing, classcol = 10)
mean(testing$cmu == predict(cmc.knncat, training, newdata = testing, 
                            train.classcol = 10, 
                            newdata.classcol = 10))
```

```
## [1] 0.4741144
```


### Multinomial log-linear model via neural networks:


```r
library(nnet)
cmc.multinom <- multinom(cmu ~ . , data = training)
```

```
## # weights:  57 (36 variable)
## initial  value 1215.065191 
## iter  10 value 1072.519679
## iter  20 value 1032.755729
## iter  30 value 1030.205818
## iter  40 value 1030.136220
## iter  40 value 1030.136219
## iter  40 value 1030.136219
## final  value 1030.136219 
## converged
```

```r
mean(testing$cmu == predict(cmc.multinom, newdata = testing, type = "class"))
```

```
## [1] 0.5313351
```

### Neural network model:


```r
library(nnet)
cmc.nnet <- nnet(cmu ~ . , data = training, 
                  size = 5, decay = 0.5)
```

```
## # weights:  108
## initial  value 1331.681707 
## iter  10 value 1180.743347
## iter  20 value 1099.438867
## iter  30 value 1051.840524
## iter  40 value 1015.955371
## iter  50 value 1003.200948
## iter  60 value 994.748437
## iter  70 value 988.920729
## iter  80 value 986.847367
## iter  90 value 985.231767
## iter 100 value 983.714545
## final  value 983.714545 
## stopped after 100 iterations
```

```r
mean(testing$cmu == predict(cmc.nnet, newdata = testing, type = "class"))
```

```
## [1] 0.5940054
```


### Support vector machines: 


```r
library(e1071)
cmc.svm <- svm(cmu ~ . , data = training, kernel = "radial") 
mean(testing$cmu == predict(cmc.svm, newdata = testing, type = "response"))        
```

```
## [1] 0.5858311
```

### Naive Bayes: 


```r
cmc.naive <- naiveBayes(cmu ~ ., data = training)
mean(testing$cmu == predict(cmc.naive, newdata = testing))
```

```
## [1] 0.520436
```

The neural network and support vector machine models delivered the highest accuracy rates obtained so far. 


# Conclusion 


```r
cmu_prop
```

```
##          cmu   n      prop
## 1     no use 629 0.4270197
## 2  long-term 333 0.2260692
## 3 short-term 511 0.3469111
```

The "no-use" class has the highest proportion at 42.7%, so this can serve as the baseline. We can see that all of the models performed better than the baseline, but QDA with an accuracy rate of about 45% is barely better. 

Most of the models achieved an accuracy rate between 50-60%. A few models performed poorly with accuracy rates below 50%. This seems to be a challenging dataset to classify. The multi-class response variable means that it's much more difficult to classify than with a binary response variable. Categorical variables inherently contain less detail than numerical variables. Given that this dataset has abundant categorical and binary variables, and only two out of nine numeric predictor variables, that could have contributed to universal low performance among the classification models. Other factors such as the uneven distribution of data from the subgroups and nonlinear patterns in the data likely made classification more challenging as well. 

Further improvements in the models is possible by carefully tuning the parameters in the models. For this project, I mostly relied on the default settings. If I had more time, I would look into model diagnostics and set the parameters that are best fit for this data set. 

Lastly, randomness in the partitioning of training and testing datasets likely also had an impact on which models performed better than others. Therfore, the performances of models presented here should not be interpreted as conclusive by any means. 


