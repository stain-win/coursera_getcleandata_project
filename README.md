Guide to creating tidy dataset from "Human Activity Recognition Using Smartphones Dataset" provided by Davide Anguita, Alessandro Ghio, Luca Oneto, Xavier Parra and Jorge L. Reyes-Ortiz. Human Activity Recognition on Smartphones using a Multiclass Hardware-Friendly Support Vector Machine. International Workshop of Ambient Assisted Living (IWAAL 2012). Vitoria-Gasteiz, Spain. Dec 2012

###Prerequisites
You must be able to run r code on your computer and you must be connected to internet since script downloads dataset from web storage ( https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip )

###Creating tidy dataset
running this script will produce tidy dataset written as .csv file - average_measurements_dataset.csv in the same location as script

#####run_analysis.r
```{r}
#define packages used
list.of.packages <- c("dplyr", "reshape2")
new.packages <- list.of.packages[!(list.of.packages %in% installed.packages()[,"Package"])]
if(length(new.packages)) install.packages(new.packages)

#load libraries
library(dplyr)
library(reshape2)

tempFile <- tempfile()
sourceUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(sourceUrl,tempFile,method="curl")
unzip(tempFile, exdir = "./data/")

#clearing all data before we start working with it
rm(list=ls())

#start importing data
activity_labels <- read.table("data/UCI HAR Dataset/activity_labels.txt", quote="\"")
features <- read.table("data/UCI HAR Dataset/features.txt", quote="\"")

test_subject <- read.table("data/UCI HAR Dataset/test/subject_test.txt", quote="\"")
test_x <- read.table("data/UCI HAR Dataset/test/X_test.txt", quote="\"")
test_y <- read.table("data/UCI HAR Dataset/test/y_test.txt", quote="\"")

#merge test data into ona dataframe
test_df <- do.call(cbind, list( test_subject, test_y, test_x))
#nice names to variables
names(test_df)[1:2] <- c('subjectId', 'activityId')

train_subject <- read.table("data/UCI HAR Dataset/train/subject_train.txt", quote="\"")
train_x <- read.table("data/UCI HAR Dataset/train/X_train.txt", quote="\"")
train_y <- read.table("data/UCI HAR Dataset/train/y_train.txt", quote="\"")

#merge train data into ona dataframe
train_df <- do.call(cbind, list(train_subject, train_y, train_x))
#nice names to variables
names(train_df)[1:2] <- c('subjectId', 'activityId')

#mergin everything into one dataframe
main_df <- rbind(test_df, train_df)

#fixing variable names so dplyr doesn't return errors
names(main_df)[3:ncol(main_df)] <- make.names(features$V2, unique=TRUE, allow_ = TRUE)

#making new dataframe with only mean and standard deviation measurements
measurements_df <- select(main_df, matches("subjectId|activityId|std|mean"))
#nice activity names instead of activity ids
measurements_df$activityId <- activity_labels[match(measurements_df$activityId, activity_labels$V1), 'V2']
names(measurements_df)[2] <- c('activity')

#new dataframe with average value for each activity and subject
measurementsAverageValues_df <- group_by(measurements_df,subjectId, activity)
measurementsAverageValues_df <- 
  measurementsAverageValues_df %>% 
  summarise_each(funs(mean))

#tidy data
measurementsAverageValues_df <- melt(measurementsAverageValues_df, id=c('subjectId','activity'))
names(measurementsAverageValues_df)[3:4] <- c('feature', 'average')

#output to file
write.table(measurementsAverageValues_df, 'average_measurements_dataset.csv', row.name=FALSE)
```

###What this script actually does

Downloads dataset containing accelerometer and gyroscope 3-axial raw signals tAcc-XYZ and tGyro-XYZ data.

Merges the training and the test datasets into one and giving meaningful names to variables and activities.

Extracts only the measurements on the mean and standard deviation for each measurement into new dataframe.
 
Creates another tidy data set, long, with the average of each variable for each activity and each subject and exports it to a .csv file.