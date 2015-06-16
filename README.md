# Global information

This is the Github repository for the tiny data project. In this directory you will find the following files:

README.md (this file) explains the basics and what you can read in each file.
CodeBook.md with the detailed data description, the file per file details and the list of all transformations.

run_analysis.R is the file with all R scripts. The goal of this script is to:

1. Merge the training and the test sets to create one data set.
2. Extract only the measurements on the mean and standard deviation for each measurement. 
3. Use descriptive activity names to name the activities in the data set
4. Appropriately label the data set with descriptive variable names. 
5. creates a second, independent tidy data set with the average of each variable for each activity and each subject.


------------------------------------------

The scripts are easy to follow:

 Select the relevant libraries

 Read the files from the hard disk

TrainData <- read.table("X_train.txt", header=FALSE)

TrainLabels <- read.table("Y_train.txt", header=FALSE)

TrainSubject <- read.table("subject_train.txt", header=FALSE)

TestData <- read.table("X_test.txt", header=FALSE)

TestLabels <- read.table("Y_test.txt", header=FALSE)

TestSubject <- read.table("subject_test.txt", header=FALSE)

# read the Activity Names table 

ActivityLabels <- read.table("activity_labels.txt", header=FALSE)



#################################################################################

######### Merge the datasets (question 1)

#### Begin with the training group (with 7352 obs)

train <- cbind(TrainSubject, TrainLabels)

#### Give an explicit name to the columns

names(train)<- c("Subject", "Activity")

#### Merge the two columns with the data in TrainData

train <- cbind(train, TrainData)

#### Follow with the Test group (with 2947 obs) 

#### Merging all data from the test sets of data

test <- cbind(TestSubject, TestLabels)

#### Give an explicit name to the columns

names(test)<- c("Subject", "Activity")

#### Merge the two columns with the data in TestData

test <- cbind(test, TestData)

#### Merge training testing data 

global <- rbind(train,test)

#### global has 10299 observations and 563 variables


#################################################################################

######### Add relevant names to know where the mean and Std are (question 4)

#### read the Features file 

Features <- read.table("features.txt", header=FALSE)

Features2<-as.vector(Features[,2])

#### create a single row with the Subject, Activity and Feature

toto <- c("Subject", "Activity",Features2)

#### add column names to the global file

colnames(global) <- toto



#################################################################################

######### Selecting only columns with mean and std (question 2)


#### detecting where it is written mean and std

goodvariables <- with(Features, (grepl("mean", V2) | grepl("std", V2 ))  & !grepl("meanFreq", V2 ))

good2<-c(TRUE,TRUE,goodvariables)

table(good2)

#### subset the global file to get only variables with mean and std PLUS subject and activity

global2<-global[,good2]

#### subset the global file to get only variables with mean and std 

global3<-global2[,3:68]


#################################################################################

######### Use descriptive activity names to name the activities (question 3)


#### rename  column names to Activity

colnames(ActivityLabels)<- c("Activity","ACTION")



#### add descriptive activity names as a new column called ACTION

globalrec<- merge(ActivityLabels, global2, x.by='label', y.by='label', all.y = T)


#################################################################################

######### Create a tidy data set with the average per subject and per activity


#### Change the shape into 4 colunms to calculate the mean

globalrec2<-globalrec[2:69]

globalmelt <- melt(globalrec2, id.vars = c("ACTION","Subject"))

##Calculate the tidy file with the mean by Activityn Subject and Variable

TidyData <- ddply(globalmelt, .(ACTION,Subject,variable), summarize, measurement = mean(value))


## write out the tidy data file to a text file

write.table(TidyData,file="tidyavgs.txt", row.name=FALSE)