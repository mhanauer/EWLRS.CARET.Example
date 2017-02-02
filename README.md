# EWLRS.CARET.Example
# EWLRS CARET Project
data = read.csv("EWLRS.Data-9.csv", header = TRUE)
library(caret)
library(mlbench)
set.seed(12345)
inTrain = createDataPartition(y = data$DOS, p = .75, list = FALSE)
# This is creating a training data set and is assigning the outcomes 
# as y and using the classification DOS.  It is using 75% of the data to create
# the training data set.  You want a fair amount of data to produce a good fit,
# remember you do not want to overfit the data.
training = data[inTrain,]; head(training); length(training$ID)
testing = data[-inTrain,]; head(testing); length(testing$ID) 
# Creating the training and testing data sets.  
# The testing data set is made up the values that were not selected into the training set.

fitControl <- trainControl(
  method = "repeatedcv",
  number = 10,
  repeats = 10)
# We are setting up a ten fold cross validation method.  
# Cross validiation takes the training data and splits it into k data sets where the user sets k.
# Here we have set k to ten.  The repeatedcv function that we are using means repeat the process of the splitting the training data into k (i.e. 10) partitions
# The fitControl function doesn't actually do anything.  It will be used in the next section where we run regressions with the training sets set up by the
# the fitControl function.
set.seed(825)
gbmFit1 <- train(DOS ~ ., data = training, 
                 method = "gbm", 
                 trControl = fitControl,
                 ## This last option is actually one
                 ## for gbm() that passes through
                 verbose = FALSE,)

# Here we have run the train algorithm set up above with the regressing all of the 
# covariates on DOS with the various training data set and then calculating the R^2 and RMSE.
# The data equal the training data we set up earlier and the trControl equals the fitControl function, which tells R
# to conduct ten cross fold valdiation 10 tens.
# gbm is a Stochastic Gradient Boosting that is used for regression and classification such classifying students as drop out or not.
# The main point of the function above to produce a model that can provide a probability of a student dropping out based upon the included
# covariates.  The function above fits the model to the training set specified by the trControl function.




# There are four pieces of output.  Essentially, R is telling us the criteria it used to decide on the best model.  
# First is the interaction depth. 
predict(gbmFit1, newdata = head(testing))
# This gives the probabilities that the students will be dropout.  It just 
# gives the first few values though.

drop.out.prob = predict(gbmFit1, newdata = testing)
drop.out.prob
# It might
