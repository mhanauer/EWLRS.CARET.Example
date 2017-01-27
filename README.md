# EWLRS.CARET.Example
# EWLRS CARET Project
data = read.csv("EWLRS.Data-9.csv", header = TRUE)
library(caret)
library(mlbench)
set.seed(12345)
inTrain = createDataPartition(y = data$DOS, p = .75, list = FALSE)
# This is creating a training data se and is assigning the outcomes 
# as y and using the classification DOS.  It is using 75% of the data to create
# the training data set.  You want a fair amount of data to produce a good fit,
# remember you do not want to overfit the data.
training = data[inTrain,]; head(training); length(training$ID)
testing = data[-inTrain,]; head(testing); length(testing$ID) 
# Creating the training and testing data sets.  Not sure how the second is working
# It is creating a testing set that is made up of the rest of the data
###### Now we are going to turn the parameters #########################
fitControl <- trainControl(## 10-fold CV
  method = "repeatedcv",
  number = 10,
  ## repeated ten times
  repeats = 10)
## We are setting up a ten fold cross validation method 
set.seed(825)
gbmFit1 <- train(DOS ~ ., data = training, 
                 method = "gbm", 
                 trControl = fitControl,
                 ## This last option is actually one
                 ## for gbm() that passes through
                 verbose = FALSE)
gbmFit1

# Here we have run the train algorithm set up above with the regressing all of the 
# covariates on DOS with the various training data set and then calculating the R^2 and RMSE


set.seed(825)
gbmFit3 <- train(DOS ~ ., data = training, 
                 method = "gbm", 
                 trControl = fitControl, 
                 verbose = FALSE, 
                 tuneGrid = gbmGrid,
                 ## Specify which metric to optimize
                 metric = "ROC")
gbmFit3
# Figure out how to get the ROC and make sure you are trying thing
# I think want to do regression to predict which class they will be, but 
# maybe classification?  You have true scores those and you want a probability.

predict(gbmFit1, newdata = head(testing))
# This gives the probabilities that the students will be dropout.  It just 
# gives the first few values though.

drop.out.prob = predict(gbmFit1, newdata = testing)
drop.out.prob
### What's next
# Make sure you are using the right algorithm for prediction
# Get the ROC curves to work
# Use this website to continue your work: http://topepo.github.io/caret/model-training-and-tuning.html#choosing-the-final-model
