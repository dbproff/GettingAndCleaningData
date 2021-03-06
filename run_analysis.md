run\_analysis
================

Including Code
--------------

### run\_analysis.R that does the following:

-   Merges the training and the test sets to create one data set.
-   Extracts only the measurements on the mean and standard deviation for each measurement.
-   Uses descriptive activity names to name the activities in the data set
-   Appropriately labels the data set with descriptive variable names.
-   From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

0 load required libraries and download data for the project.
------------------------------------------------------------

``` r
    rm(list = ls(all.names = TRUE))
    wd <<- "C:/Users/user/Documents/R/3.Getting And Cleansing Data/week4/final"
    gwd <- getwd()
    
    if ( gwd != wd ) {setwd (wd)}
    
    if("data.table" %in% rownames(installed.packages()) == FALSE) {install.packages("data.table")}
    require(data.table)
```

    ## Loading required package: data.table

``` r
    if("plyr" %in% rownames(installed.packages()) == FALSE) {install.packages("plyr")}
    require(plyr) 
```

    ## Loading required package: plyr

``` r
    if("reshape" %in% rownames(installed.packages()) == FALSE) {install.packages("reshape")}
    require(reshape)
```

    ## Loading required package: reshape

    ## 
    ## Attaching package: 'reshape'

    ## The following objects are masked from 'package:plyr':
    ## 
    ##     rename, round_any

    ## The following object is masked from 'package:data.table':
    ## 
    ##     melt

``` r
    if("downloader" %in% rownames(installed.packages()) == FALSE) {install.packages("downloader")}
    require(downloader)
```

    ## Loading required package: downloader

``` r
    ###  download data archive and unzip 
    fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
    dataFile = "./getdata-projectfiles-UCI HAR Dataset.zip"
    if(file.exists(dataFile)) {
        message("File is already downloaded")
    } else {
        download(fileUrl, dataFile, mode = "wb")
        unzip(dataFile)
    }
```

    ## File is already downloaded

1 Merges the training and the test sets to create one data set.
---------------------------------------------------------------

    ### read train files

``` r
        x_train <- read.table("./UCI HAR Dataset/train/X_train.txt")
        y_train <- read.table("./UCI HAR Dataset/train/y_train.txt")
        subject_train <- read.table("./UCI HAR Dataset/train/subject_train.txt")
    ### read test files

        x_test<- read.table("./UCI HAR Dataset/test/X_test.txt")
        y_test <- read.table("./UCI HAR Dataset/test/y_test.txt") 
        subject_test <- read.table("./UCI HAR Dataset/test/subject_test.txt")

            ### TRAINING DATA STRUCTURE SUMMARY  
                # summary(x_train)
                # summary(y_train)
                # summary(subject_train)

            ### TEST DATA STRUCTURE SUMMARY  
                # summary(x_test)
                # summary(y_test)
                # summary(subject_test)
```

Uses descriptive activity names to name the activities in the data set
----------------------------------------------------------------------

    ### read features file 

``` r
    features <- read.table("./UCI HAR Dataset/features.txt")
    

    ### Appropriately labels the data set with descriptive variable names.
        names(x_train)<-features[,2]
        names(x_test)<-features[,2]

        names(y_train)<-c("ActivityId")
        names(y_test)<-c("ActivityId")

        names(subject_train)<-c("SubjectId")
        names(subject_test)<-c("SubjectId")



    ### column bind subject, activity and features data:
        train_data <- cbind(subject_train, y_train, x_train)
        test_data  <- cbind(subject_test, y_test, x_test)

    ### row bind train and test data:
        mergedData <- rbind(train_data, test_data)
```

3 Extracts only the measurements on the mean and standard deviation for each measurement.
-----------------------------------------------------------------------------------------

    ### use pattern matching to extract mean and std data and keep subjectid and activityid

``` r
                mergedData <- mergedData[,grepl("SubjectId|ActivityId|mean\\(\\)|std\\(\\)", names(mergedData))]
```

Uses descriptive activity names to name the activities in the mergedData set
----------------------------------------------------------------------------

``` r
                    ### join mergedData with activity_labels on ActivityId
        activity_labels <- read.table("./UCI HAR Dataset/activity_labels.txt")
        names(activity_labels) <- c("ActivityId", "Activity")
        mergedData <- join( mergedData,activity_labels, by="ActivityId",  match="first")     
    ### tidy column names
    ### remove "()"
        names(mergedData) <- gsub("\\(\\)", "", names(mergedData)) 
    ### replace dashes with undescores 
        names(mergedData) <- gsub("-", "_", names(mergedData))  
    ### capitalize Mean and Std
        names(mergedData) <- gsub("mean", "Mean", names(mergedData)) 
        names(mergedData) <- gsub("std", "Std", names(mergedData)) 
    
    ###  move activity column in the beginning  &  remove activityId index column 
        col_idx <- grep("^Activity$", names(mergedData))
        tidyData <- mergedData[, c(col_idx, (1:ncol(mergedData))[-col_idx])]
        rm(mergedData)
        tidyData <- tidyData[,-2]
        str(tidyData)
    
    ### Write the dataset of activities/subjects/features to a file   
    tidyDataFile <- "GettingCleaningData_Project.TXT"
    write.table(tidyData, tidyDataFile, row.names=FALSE)
```

5 From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
------------------------------------------------------------------------------------------------------------------------------------------------

``` r
    ### set SubjectId, ActivityId, Activity as a key.
         tidyData <- as.data.table(tidyData)
        # tidyData <- tbl_df(tidyData)
         setkey(tidyData, SubjectId, Activity)

    ### reshape dataset from a short and wide format to a tall and narrow format.

        tidyData <- data.table(melt(tidyData, key(tidyData), variable.name="featureCode"))

## Create a data set with the average of each featureCode for each Activity and each SubjectId.
    setkey(tidyData, SubjectId, Activity,featureCode)
  
    ### Using cast to move featureCode into columns 
    tidyDataSummary <- cast(tidyData, SubjectId+Activity ~ featureCode,mean)
    tidyDataSummary <- as.data.table(tidyDataSummary)
    rm(tidyData)
    str(tidyDataSummary)
    ### Write the final tidy dataset of activities/subjects/features Means to a file   
    tidyDataFile <- "GettingCleaningData_Project_Mean.TXT"
    write.table(tidyDataSummary, tidyDataFile, row.names=FALSE)
```
