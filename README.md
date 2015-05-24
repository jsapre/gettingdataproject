# This README explains the steps of analysis for the GettingAndCleaningData course project

The instruction for steps of the scripit was as follows
 1. Merges the training and the test sets to create one data set.
 2. Extracts only the measurements on the mean and standard deviation for each measurement. 
 3. Uses descriptive activity names to name the activities in the data set
 4. Appropriately labels the data set with descriptive variable names. 
 5. From the data set in step 4, creates a second, independent tidy data set with 

The slightly different actual steps are described below.

## R packages used
* plyr

#### Assumption as per instructions: 
* current working directory is "UCI HAR Dataset" and the data is available in it

#### First feature and activity labels are read
```
feature_labels <- read.table("features.txt", header = FALSE, stringsAsFactors = FALSE)
activity_labels <- read.table("activity_labels.txt", header = FALSE, stringsAsFactors = FALSE)
```
#### Then  test data is read  subject, y the activities and x all the observations of variables 
```
test_subject <- read.table("test/subject_test.txt", header = FALSE)
test_y <- read.table("test/y_test.txt", header = FALSE)
test_x <- read.table("test/x_test.txt", header = FALSE)
```
#### Subsetting is done to get only the columns corresponding to mean() and standard deviation std() variables
```
test_x_reduced <- test_x[,feature_labels[grep("mean\\(\\)|std\\(\\)", feature_labels$V2),1]]
```
#### Test data is combined from the subject, activity and variables reduced data
```
test_data <- cbind(test_subject, test_y, test_x_reduced)
```
####  Give better labels to variables : use "subject", "activity" and feature_labels to begin with
```
colnames(test_data) <- c("subject", "activity", feature_labels[grep("mean\\(\\)|std\\(\\)", feature_labels$V2),2])
```
# The steps to read test data are repeated to read train data
A function might have been written to avoid the repetition of code but the code is just steps
and it results in variables with specific names so function is not deemed very necessary.

#### Read train data
```
train_subject <- read.table("train/subject_train.txt", header = FALSE)
train_y <- read.table("train/y_train.txt", header = FALSE)
train_x <- read.table("train/x_train.txt", header = FALSE)
```
#### Subset only the columns corresponding to mean() and standard deviation std() variables
```
train_x_reduced <- train_x[,feature_labels[grep("mean\\(\\)|std\\(\\)", feature_labels$V2),1]]
```
#### Combine the subject, activity and variables data
```
train_data <- cbind(train_subject, train_y, train_x_reduced)
```
####  Give better labels to variables : use "subject", "activity" and feature_labels to begin with
```
colnames(train_data) <- c("subject", "activity", feature_labels[grep("mean\\(\\)|std\\(\\)", feature_labels$V2),2])
```
# Further tidying is done as follows

#### The test and train data is combined
```
combined_data <- rbind(test_data, train_data)
```
#### Activity numbers are replaced with descriptions
```
combined_data$activity <- activity_labels[combined_data$activity, 2]
```
#### Group by subject & activity and  mean of variables by group is taken
```
tidy_data <- aggregate(. ~subject + activity, combined_data, mean)
```

#### Variable names are made more descriptive by expanding the labels
   gsub() is used to replace patterns in the variable names to    make it more meaningful
```
cn <- colnames(tidy_data)
cn <- gsub(pattern = "^t", replacement =  "timeDomain", cn)
cn <- gsub(pattern = "^f", replacement =  "frequencyDomain", cn)
cn <- gsub(pattern = "Gyro", replacement =  "Gyroscope", cn)
cn <- gsub(pattern = "Acc", replacement =  "Accelerometer", cn)
cn <- gsub(pattern = "$", replacement =  "-groupMean", cn)
cn <- gsub(pattern = "subject-groupMean", replacement =  "subject", cn)
cn <- gsub(pattern = "activity-groupMean", replacement =  "activity", cn)
colnames(tidy_data) <- cn
```
#### The tidy data is written to a text file 
```
write.table(tidy_data, file = "jsapre_tidy_data.txt", row.names =  FALSE)
```