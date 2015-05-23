# Classifying Quality of Barbell Exercises from Sensor Data

## Executive Summary
In this analysis, I take sensor data collected during barbell exercises and build a Random Forest model to classify the quality of the exercise. After cleaning the data to strip non-predictor variables, I partition it into a train set (80% of data) and a validation set (20% of data). I build a Random Forest model on the training set and estimate the out of sample error on the validation set, finding an estimate of 0.57%. Finally, I predict the exercise quality on a separate test set of 20 samples.

## Analysis

### Import data

```r
data <- read.csv("pml-training.csv")
testing <- read.csv("pml-testing.csv")
```

### Load Libraries

```
## Loading required package: lattice
## Loading required package: ggplot2
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

### Clean Data

```r
originalData <- data
# Remove rows with newwindow == 'yes'
data <- data[data$new_window != "yes",]

# Remove columns with NA's
data <- data[, colSums(is.na(data)) == 0]

# Remove empty columns
data <- data[,colSums(data == "") == 0]

# Remove Timestamps and other non-predictors
data <- subset(data, select=-c(X, user_name, raw_timestamp_part_1, raw_timestamp_part_2, cvtd_timestamp, new_window, num_window))
```

### Split Data into Training and Validation set

```r
set.seed(2526)
inTrain <- createDataPartition(y=data$classe, p=0.8, list=FALSE)
training <- data[inTrain,]
validation <- data[-inTrain,]
```

### Train Random Forest

```r
set.seed(443)
modFit.rf <- randomForest(classe ~ ., data=training, importance=TRUE)
```

### Predict on Validation dataset

```r
valPred.rf <- predict(modFit.rf, validation)
```

### Check accuracy of predictions on validation set

```r
table(valPred.rf, validation$classe)
```

```
##           
## valPred.rf    A    B    C    D    E
##          A 1093    5    0    0    0
##          B    1  738    8    0    0
##          C    0    0  662    4    2
##          D    0    0    0  624    1
##          E    0    0    0    1  702
```

```r
correct <- valPred.rf == validation$classe
```

The accuracy of the model on the validation dataset is 0.9942723, giving an estimated out of sample error rate of 0.0057277


### Predict on Testing dataset

```r
testPred.rf <- predict(modFit.rf, testing)
```


## Appendix


### Write predictions to files

```r
pml_write_files = function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("problem_id_",i,".txt")
    write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
  }
}
pml_write_files(testPred.rf)
```

### Citation
Data provided by: 

Velloso, E.; Bulling, A.; Gellersen, H.; Ugulino, W.; Fuks, H. Qualitative Activity Recognition of Weight Lifting Exercises. Proceedings of 4th International Conference in Cooperation with SIGCHI (Augmented Human '13) . Stuttgart, Germany: ACM SIGCHI, 2013.
