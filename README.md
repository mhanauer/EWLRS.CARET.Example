---
title: "Using CARET for Predictions"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
This is an example of how to apply the CARET machine learning package in R to classify individuals or objects based upon covariates.  I use the iris data set as an example that takes characteristics of three flower types using four covariates that describe the flowers (five total variable).  More information about the data set can be found here: http://stat.ethz.ch/R-manual/R-devel/library/datasets/html/iris.html

This example is based upon the example provided by the creators of the CARET package who demonstrated a very similar process with a different data set.  The example can be found here: http://topepo.github.io/caret/model-training-and-tuning.html

The goal of this example is to develop a model that will accurately predict the classification of each flower based upon the following characteristics, sepal length, sepal width, petal length, and petal width.  However, the process that I am about to explain can be used for other more meaningful data.  For example, variables such as attendance, GPA, and in and out of school suspensions can be used to obtain the probability that a student will drop out of school.  


Below we read in the data containing five variables.

Next we need to load and library two packages caret and mlbench.

Then we set the seed, so that others can reproduce the results shown in this example.
```{r }
data("iris")
iris$Species = as.factor(iris$Species)
head(iris)
library(caret)
library(mlbench)
set.seed(12345)

```
Next we need to partition the training sets from the testing sets.  The createDataPartition in CARET does this by taking a stratified random sample of .75 of the data for training.

We then create both the training and testing data sets which will be used to develop and evaluate the model.
```{r}
inTrain = createDataPartition(y = iris$Species, p = .75, list = FALSE)
training = iris[inTrain,]
testing = iris[-inTrain,] 
```
Here we are creating the cross validation method that will be used by CARET to create the training sets. Cross validation means to randomly split the data into k (in our case ten) data testing data sets and the repeated part just means to repeat this process k times (in our case ten as well).
```{r}
fitControl <- trainControl(
  method = "repeatedcv",
  number = 10,
  repeats = 10)
```
Now we are ready to the develop model.  We use the train function in CARET to regress the dependent variable Species onto all of the other covariates.  Instead of explicitly naming all of the covariates, in the CARET package the "." is used, which means include all of the other variables in the data set.  

Next the method or type of regression is selected.  Here we are using the gbm or Stochastic Gradient Boosting that is used for regression and classification.  More information about the gbm package can be found here: https://cran.r-project.org/web/packages/gbm/gbm.pdf

The trControl is used to assign the validation method created above.  It says run a gbm model with a ten cross validation method and repeat that process ten times.  Finally, the verbose command just hides the calculations CARET computes from the user.
```{r}
set.seed(12345)
gbmFit1 <- train(Species ~ ., data = training, 
                 method = "gbm", 
                 trControl = fitControl,
                 verbose = FALSE)

```
Let's now inspect the results.  The most important piece of information is the accuracy, because that is what CARET uses to choose the final model.  It is the overall agreement rate between the cross validation methods.  The Kappa is another statistical method used for assessing models with categorical variables such as ours. 

CARET chose the first model with an interaction depth of 1, number of trees at 50, an accuracy of 97% and a Kappa of 95%.
```{r}
gbmFit1
```
Finally, we can use the training model to predict both classifications and probabilities for the test data set.  

The first line of codes uses the built in predict function with the training model (gbmFit1) to predict values using the testing data set, which is the 25% of the data set that we set aside at the beginning of this example.  We include the code "head" for your convenience so that R does not display the entire data set.  If "head" were removed R would display all of the predictions.  

The first piece of code includes the argument type = "prob", which tells R to display the probabilities that a flower is classified as setosa, versicolor, or virginica.  As we can see, there is a 99% probability that the first flow in the data set is a setosa.

Again, as stated at the beginning of this example, other more meaningful data sets can be substituted for the iris data set using the steps provided above.
```{r}
predict(gbmFit1, newdata = head(testing), type = "prob")

```

