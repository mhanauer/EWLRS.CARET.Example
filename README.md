---
title: "Using CARET to Predict Student Dropout Rates"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

This is an example of how to apply the CARET machine learning package in R to the identification of students at risk of dropping out of school.  I use a fake data set to demonstrate how typical indicators of risk (academic performance, in and out of school suspensions, and absences) can predict the probability of a student dropping out. 

This example is based upon the example provided by the creators of the CARET package who demonstrated a very similar process. Instead of predicting student’s risk of dropping out of school, the creators used the famous iris data set to classify flowers based upon their characteristics.  The example can be found here: http://topepo.github.io/caret/model-training-and-tuning.html

Below we read in data containing six variables.  First is the student ID, which is just a unique identifier for each student.  G represents the student's grade on a scale of 1-4 over the past school year.  A represents the number of times the student was absent during the school year.  ISS and OSS represent in and out of school suspensions over the past school year.  And finally, the dependent variable or the variable of interest is DOS.  DOS represents whether the student's drop out status.

After loading the data, DOS will likely need to be converted to a factor, so that CARET treats this variable as a factor with two levels and provides the appropriate output later on.

Next the we need to load and library two packages caret and mlbench.

Then I set the seed, so that others can reproduce the results shown in this example.



```{r }
data = read.csv("EWLRS.Data.csv", header = TRUE); head(data)
data$DOS = as.factor(data$DOS); head(data$DOS)
library(caret)
library(mlbench)
set.seed(12345)

```


Next we need to partition the training sets from the testing sets.  The createDataParition in CARET does this by taking a stratified random sample of .75 of the data for training.

We then create both the training and testing data sets which will be used to develop a model and evaluate the model.

```{r}
inTrain = createDataPartition(y = data$DOS, p = .75, list = FALSE)
training = data[inTrain,]; head(training); length(training$ID)
testing = data[-inTrain,]; head(testing); length(testing$ID) 
```

Here we are creating the cross validation method that will be used by CARET to create the training sets. Cross validation means to randomly split the data into k (in our case ten) data testing data sets and the repeated part just means to repeat this process k times (in our case ten).

```{r}
fitControl <- trainControl(
  method = "repeatedcv",
  number = 10,
  repeats = 10)
```

Now we are ready to develop model.  We use the train function in CARET to regress the dependent variable DOS onto all of the other covariates (academic performance, in and out of school suspensions, and absences).  Instead of explicitly naming all of the covariates, in the CARET package the "." is used, which means include all of the other variables in the data set.  

Next the method or type of regression is selected.  Here we are using the gbm or Stochastic Gradient Boosting that is used for regression and classification such classifying students as drop out or not. More information about the gbm package can be found here: https://cran.r-project.org/web/packages/gbm/gbm.pdf

The trControl is used to assign the validation method created above.  It says run a gbm model for the with a ten cross validation method and repeat that process ten times.  Finally, the verbose command just hides the calculations CARET computes from the user.

```{r}
set.seed(12345)
gbmFit1 <- train(DOS ~ ., data = training, 
                 method = "gbm", 
                 trControl = fitControl,
                 verbose = FALSE)

```

Let's now inspect the results.  The most important piece of information is the accuracy.  It is the overall agreement rate between between the cross validation methods.  The Kappa is another statistics method used for assessing models with categorical variables such as ours.     


```{r}
gbmFit1
```

Finally, we can use the training model to predict both classifications and probabilities for test data set.  

The first line of codes uses the built in predict function with the training model (gbmFit1) to predict values using the testing data set, which is the 25% of the data set that we set aside at the beginning of this example.  We include the code "head" for your convenience so that R does not display the entire sample.  If "head" were removed R would display all of the probabilities.  Additionally, the user can assign this function to a variable such predict.DOS and access the predicated values of drop out status that way.

The first piece of code includes the argument type = "prob", which tells R to displays the probabilities that a student is classified as either a 1 or 0 (i.e. 1 means dropout and 0 means not a dropout).  As we can see, there is a 75% that student one will dropout of school, which is helpful for school administrators who want to understand which students to target.

```{r}
predict(gbmFit1, newdata = head(testing), type = "prob")

```

