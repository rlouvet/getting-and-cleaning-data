Getting and Cleaning Data Course Project
=========================

This file describes the process used to obtain the tidy data that can be found in the file "UCI HAR Dataset.txt"

## Libraries used

Here is a list of libraries used for the project:
* library(data.table)
* library(reshape2)

## Reading files from working directory
First we read features and activity names
<pre><code>features = read.table(".\\UCI HAR Dataset\\features.txt")
activities = read.table(".\\UCI HAR Dataset\\activity_labels.txt")
names(activities) = c("Id", "Activity")</pre></code>

Then prepare the list of data files to be read for each folder (training or test) and loop on these files
<pre><code>seq = list(c(".\\UCI HAR Dataset\\train",list.files(path = ".\\UCI HAR Dataset\\train", pattern = "*.txt")),
           c(".\\UCI HAR Dataset\\test",list.files(path = ".\\UCI HAR Dataset\\test", pattern = "*.txt")))
for (files_list in seq){ #looping first on training files then on test files
  object_names = gsub(".txt", "", files_list[2:length(files_list)]) # Creating a list of vector names based on file names
  inputfolder = files_list[1]
  for (i in 2:length(files_list)){ # looping on each file of the selected folder
    f1 = read.table(paste(inputfolder, "\\",files_list[i], sep=""))
    assign(object_names[i-1], f1) # renaming with corresponding vector name
  }
}
remove("f1","files_list","object_names") # deleting temporary declaration</pre></code>

## Building the tidy data set
We merge the training and the test sets to create one data set
<pre><code>total_subject = rbind(subject_train,subject_test)$V1
total_X = rbind(X_train,X_test)
colnames(total_X) = features$V2
total_Y = rbind(y_train,y_test)$V1</pre></code>

We retrieve activity name string 
<pre><code>Y_activity = activities[match(total_Y,activities$Id),"Activity"]
selected_variables = grep("mean\\(\\)|std", features$V2, value = TRUE)</pre></code>
Then we create the tidy data set with "melt" and "dcast" functions
<pre><code>DT = data.table(Activity = Y_activity, Subject = total_subject, X = total_X[,selected_variables])
melted_data = melt(DT,id=c("Activity","Subject"))
tidy_data = dcast(melted_data, Activity + Subject ~ selected_variables,fun.aggregate=mean)</pre></code>

Finally we rename column names to match with best pratices
<pre><code>colnames(tidy_data) = gsub("\\(\\)", "", colnames(tidy_data))
colnames(tidy_data) = gsub("-", ".", colnames(tidy_data))</pre></code>
