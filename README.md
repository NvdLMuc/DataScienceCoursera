#"Course project Getting and Cleaning Data"

The course project, written in run_analysis.R using dplyr package,  was subdivided into five subtasks:

##Task 1: Merging data sets
	*The training and test data sets (X_train,X_test) were loaded together with the corresponding Subjectlist and Activitylist related to each record in the data set
	*Each set was merged with the corresponding Subject list and Activity list
	*Finally, both extended data sets were merged
	
##Task 2: Selecting mean and standard deviation features
	*The features describing the mean and standard deviation were selected by analysing the featurenames and selecting those containing "mean()" and "std()"
	*The complete dataset was reduced using this selection. Additionally, the columns with the ID description (ParticipantID and ActivityID) were incorporated
	
##Task 3: Altering ActivityID in Descriptive Test
	*The AcitivityIDs were subsitiuted by the description, which was read from the activity_labels.txt-file.  This was achieved by merging  both tables and removing the activityID column

##Task 4: Descriptive labeling
	*The column-names were cleaned: "-" was replaced by "_" and "()" by ""
	
##Task5: Grouping Dataset and evaluate means
	*The dataset was grouped by participant and activity
	*The individual mean of all features in a group war calculated using the dplyr function summarise_each()
	*The resulting table was written to a file named "ReducedDataset_Mean.txt"
	
-----
#Source code run_analysis.R:

	
	# Task data science course "Getting and Cleaning Data", Coursera
	library(dplyr)
	
		### TASK 1 : Merging Training and Test dataset
		# Training set (Datafiles are assumed to exist otherwise use function if(!file.exists...))
		# load data sets & add corresponding column names
		feat  <- read.table("features.txt",stringsAsFactors = FALSE)
		Train_x <- read.table("./train/X_train.txt")
		colnames(Train_x) <- feat$V2
		Train_y <- read.table("./train/y_train.txt")
		colnames(Train_y) <- c("ActivityID")
		Train_sub <- read.table("./train/subject_train.txt")
		colnames(Train_sub) <- c("ParticipantID")
		SETID <- rep("TrainingSet",1,NROW(Train_x))
	
	#Combine data and remove large data sets which are not needed anymore
		Training_Set <- cbind(SetID=SETID,Train_sub,Train_y,Train_x)
		rm("Train_sub","Train_x","Train_y","feat","SETID") 
	
		# Test set (Datafiles are assumed to exist otherwise use function if(!file.exists...))
	# load data sets & add corresponding column names
		feat  <- read.table("features.txt",stringsAsFactors = FALSE)
		Test_x <- read.table("./test/X_test.txt")
		colnames(Test_x) <- feat$V2
		Test_y <- read.table("./test/y_test.txt")
		colnames(Test_y) <- c("ActivityID")
		Test_sub <- read.table("./test/subject_test.txt")
		colnames(Test_sub) <- c("ParticipantID")
		SETID <- rep("TestSet",1,NROW(Test_x))
	
	#Merge data and remove large data sets which are not needed anymore
		Test_Set <- cbind(SetID=SETID,Test_sub,Test_y,Test_x)
		rm("Test_sub","Test_x","Test_y","feat","SETID") 
	
	# Merge Training and Test set
		Dataset <- rbind(Test_Set,Training_Set)
		rm("Test_Set","Training_Set")
	
	### TASK 2: Retrieve Mean and Standard deviation data
	## Select mean and standard deviation data
		ColD <- colnames(Dataset)
		ColSelection <- which(grepl("mean\\(\\)",ColD)|grepl("std\\(\\)",ColD)|grepl("ID$",ColD))
		#ColSelection <- c(1:3,ColSelection)
		DataSet_MeanStd <- Dataset[,ColSelection]
		rm("ColD","ColSelection")
	
	### TASK 3: Rename Activity ID into descriptive names
		act_liv <- read.table("activity_labels.txt",stringsAsFactors = FALSE)
		colnames(act_liv) <- c("ActivityID", "ActivityName")
		Dataset_red <- select(merge(DataSet_MeanStd,act_liv,by="ActivityID"),-ActivityID)
	
	### TASK 4: Labeling
	##  Removes parathenses and hyphens in labels
		ColD <- colnames(Dataset_red)
		ColD1 <- sapply(ColD,function (x) gsub("-","_",x))
		ColD2 <- sapply(ColD1,function(x) gsub("\\(\\)","",x))
		colnames(Dataset_red) <- ColD2
		rm("ColD","ColD1","ColD2")
	
	### TASK 5: Mean of variable grouped by Subject(ParticipantID) and ActivityName
		Dataset_red_bySubject_Activity <- group_by(select(Dataset_red,-SetID),ParticipantID,ActivityName)
		Dataset_mean <- summarise_each(Dataset_red_bySubject_Activity,funs(mean))
		write.table(Dataset_mean,"ReducedDataset_Mean.txt", row.names=FALSE)
		rm("DataSet_MeanStd", "Dataset_red","Dataset_red_bySubject_Activity")
