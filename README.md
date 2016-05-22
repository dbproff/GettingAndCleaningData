
project code_book : : [code_book.md](code_book.md)

Project source code is in the file : [run_analysis.R](run_analysis.R)

## The run_analysis.R script is composed of 5 major phases with the following functionality:

###### Phase 0 is designed to load required libraries and download data for the project.  

###### Phase 1 is designed to merge the training and the test sets to create one data set. 
* reads the training and its corresponding subject and activity label files 
	and merge it into one tidy data set named (train_data).
* reads the testing and its corresponding subject and activity label files 
	and merge it into one tidy data set named (test_data).
	
###### Phase 2		
	
* appropriately label the data set (train_data) and (test_data) with descriptive variable names 
	taken from the features.txt and activity_labels.txt
* merges the training and testing tidy data sets into one and name it as (mergedData).


###### Phase 3 is designed to extract only the measurements on the mean and standard deviation for each measurement in (mergedData). 
* use pattern matching to extract mean and std data from (mergedData) and keep subjectid and activityid

###### Phase 4 is designed to assign a descriptive activity name for every activity in the filteredDataSet through the following phases:
* reads the activity_labels.txt file to create a lookup table (activityLabels).
* assign the appropriate activity label based on the values in the lookup table.
* join mergedData with activity_labels on ActivityId
* tidy up  column names
	* remove "()"
	* replace dashes with undescores 
	* capitalize Mean and Std
* Write the dataset of activities/subjects/features to a file   "GettingCleaningData_Project.TXT"

###### Phase 5 From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
* set SubjectId, ActivityId, Activity as a key.
* reshape dataset from a short and wide format to a tall and narrow format using melt()
* create a data set with the average of each featureCode for each Activity and each SubjectId using cast()
* write the final tidy dataset of activities/subjects/features Means to a file  "GettingCleaningData_Project_Mean.TXT"
