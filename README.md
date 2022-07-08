# School_District_Analysis
An analysis of school districts performance in math and reading using Python and Pandas
## Overview of the school district analysis:
This program centers around the PyCitySchools_Challenge_testing.ipynb file. the PyCitySchools.ipynb file acts as the source code from which this has been refactored. PyCitySchools was originally a program designed to create a detailed summary of how various districts and schools within those districts were performing in their math and reading scores utilizing a csv file of student scores. After analyzing the districts and schools, it became apparent that more categories to analyze would help detetmine the reasoning behind the results of the analysis. As such, the amount spent per student by a school, school size, and school type were all analyzed as well.

Where PyCitySchools_Challenge_testing.ipynb differs is that the ninth grade reading and math scores for a specific school needed to be removed. Under suspicion of academic dishonesty, the Thomas High School 9th grade scores needed removal to ensure that the results for the school district remained accurate. As such, the code has been refactored to remove these test scores.

To begin, the program imports the pandas library under the alias "pd" so that it can make use of pandas dataframes. Next, it reads the schools_complete.csv and students_complete.csv files and stores each line in the school_data_df and student_data_df dataframes respectively. As the student_data_df contained prefixes in the names for each student, these needed to be removed as they were for PyCitySchools.ipynb.

`import pandas as pd`

`school_data_to_load = "Resources/schools_complete.csv"`

`student_data_to_load = "Resources/students_complete.csv"`

`school_data_df = pd.read_csv(school_data_to_load)`

`student_data_df = pd.read_csv(student_data_to_load)`

`prefixes_suffixes = ["Dr. ", "Mr. ","Ms. ", "Mrs. ", "Miss ", " MD", " DDS", " DVM", " PhD"]`

`for word in prefixes_suffixes:`

`    student_data_df["student_name"] = student_data_df["student_name"].str.replace(word,"")`

This was the resulting student_data_df dataframe:

![student_data_df](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/student_data_df.png?raw=true)

Next, numpy was imported as np to give access to the non-numerical value NaN. Then, to replace the Thomas High School 9th grade test scores with NaN, the pandas loc method was used and repurposed for each test:

`student_data_df.loc[(student_data_df['grade'] == "9th") & (student_data_df['school_name'] == "Thomas High School"), 'reading_score'] = np.NAN`

This was the resulting change to the student_data_df:

![Thomas_9th_NaN](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/Thomas_9th_NaN.png?raw=true)

### District Summary
Next, with the necessary changes made to students_data_df, the students_data_df and the school_data_df were ready to be combined into the school_data_complete_df using the merge() method:

`school_data_complete_df = pd.merge(student_data_df, school_data_df, how="left", on=["school_name", "school_name"])`

The resulting dataframe was such:

![schools_data_complete_df](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/school_data_complete_df.png?raw=true)

Next, the total number of schools, total number of students, and total budget per school needed to be calculated. To accomplish this, the school_count was calculated as the length of the list of unique school names. For the student_count, it was the count of each student name. And for the total_budget, it was calculated as the sum of the budget of every school in the school_data_df:

`school_count = len(school_data_complete_df["school_name"].unique())`

`student_count = school_data_complete_df["Student ID"].count()`

`total_budget = school_data_df["budget"].sum()`

After this, the average reading and math scores were calculated for the clean data using the mean() method:

`average_reading_score = school_data_complete_df["reading_score"].mean()`

`average_math_score = school_data_complete_df["math_score"].mean()`

Next, to update the total student counts to account for the removed Thomas High 9th graders, the total students and students to remove had to be tallied. Following this, the students to remove were subtracted from the original count of students:

`thomas_9th = school_data_complete_df.loc[(school_data_complete_df["grade"] == "9th") & (school_data_complete_df["school_name"] == "Thomas High School"), "Student ID"].count()`

`student_count = school_data_complete_df["Student ID"].count()`

`student_count = student_count - thomas_9th`

As 70% or higher is considered a passing score for the math test, reading test, or both, this was used as the basis for tallying the students names who met this criteria:

`passing_math_count = school_data_complete_df[(school_data_complete_df["math_score"] >= 70)].count()["student_name"]`

`passing_reading_count = school_data_complete_df[(school_data_complete_df["reading_score"] >= 70)].count()["student_name"]`

`passing_math_reading = school_data_complete_df[(school_data_complete_df["math_score"] >= 70)`

`                                               & (school_data_complete_df["reading_score"] >= 70)]`

`overall_passing_math_reading_count = passing_math_reading["student_name"].count()`

The total percentage of students passing math and english were then calculated by dividing the passing student count by the total student count and multiplying by 100:

`passing_math_percentage = 100 * passing_math_count / student_count`

`passing_reading_percentage = 100 * passing_reading_count / student_count`

`overall_passing_percentage = 100 * overall_passing_math_reading_count / student_count`

With these values all calculated and adjusted for the removed Thomas High 9th grade students, the district_summary_df was ready to be created:

`district_summary_df = pd.DataFrame(`

`          [{"Total Schools": school_count, `

`          "Total Students": student_count, `

`          "Total Budget": total_budget,`

`          "Average Math Score": average_math_score, `

`          "Average Reading Score": average_reading_score,`

`          "% Passing Math": passing_math_percentage,`

`         "% Passing Reading": passing_reading_percentage,`

`        "% Overall Passing": overall_passing_percentage}])`

However, as the dollar and float values were in near unreadable formats, these values needed to be reformatted as such to include dolar values and decimal rounding:

`district_summary_df["Total Students"] = district_summary_df["Total Students"].map("{:,}".format)`

`district_summary_df["Total Budget"] = district_summary_df["Total Budget"].map("${:,.2f}".format)`

The result was the following dataframe:

![district_summary_df](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/district_df.png?raw=true)

### School Summary
To be able to summarize results by school, lists of each value in the district_summary_df needed to be created and grouped by "school_name":

`per_school_math = school_data_complete_df.groupby(["school_name"]).mean()["math_score"]`

Next, in the same manor that the district_summary_df was created, the per_school_summary_df was created:

![per_school_summary_df](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/per_school_summary_df.png?raw=true)

However, this chart has one very notable flaw, it does not account for the 9th graders at Thomas High School that need to be removed. As such, the % passing is abnormally low. The next steps would be used to account for this discrepancy.

Firstly, the Thomas High School 10th-12th graders were stored in a new dataframe that would be used to replace the Thomas High School test grade stats:

`thomas_non9th_df = pd.DataFrame(school_data_complete_df.loc[(school_data_complete_df['school_name'] == "Thomas High School")`

`                                            & (school_data_complete_df['grade'] != "9th")])`

Using this dataframe, the number of Student ID's were counted to get the # of test scored being recorded. Next, using the thomas_non9th_df, the students who passed math, english, and both were recorded in their own respective dataframes:

`thomas_overall_passing_df = pd.DataFrame(thomas_non9th_df.loc[(thomas_non9th_df['math_score'] >= 70) & (thomas_non9th_df['reading_score'] >= 70)])`

![thomas_overall_passing_df](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/thomas_overall_passing_df.png?raw=true)

Next, the percentages of passing Thomas Students were calculated:

`thomas_passing_math_percentage = 100 * thomas_passing_math_df['Student ID'].count() / thomas_non9th_count`

`thomas_passing_reading_percentage = 100 * thomas_passing_reading_df['Student ID'].count() / thomas_non9th_count`

`thomas_overall_passing_percentage = 100 * thomas_overall_passing_df['Student ID'].count() / thomas_non9th_count`

These percentages were then used to replace the respective Thomas High School values in the per_school_summary_df:

`per_school_summary_df.loc[["Thomas High School"], "% Passing Math"] = thomas_passing_math_percentage`

The result was an updated dataframe that accounts for these adjusted values:

![per_school_summaru_df_thomas_updated](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/per_school_summary_df_thomas_update.png?raw=true)

### High and Low Performing Schools
To display the highest and lowest performing schools overall, these schools were sorted by % Overall Passing:

`top_schools = per_school_summary_df.sort_values(["% Overall Passing"], ascending=False)`

`top_schools.head()`

![top_5_schools](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/top_5_schools.png?raw=true)

`bottom_schools = per_school_summary_df.sort_values(["% Overall Passing"], ascending=True)`

`bottom_schools.head()`

![bottom_5_schools](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/bottom_5_schools.png?raw=true)

### Math and Reading Scores by Grade
To display the analysis of the performance of students by grade, a series of scores by grade was needed to be created. Then, using this series, the average math and reading scores per grade could be calculated. Finally, these scores would be compiled and formatted into their own math_scores_by_grade_df and reading_scoresby_grade_df dataframes:

`ninth_graders = school_data_complete_df[(school_data_complete_df["grade"] == "9th")]`

`ninth_grade_math_scores = ninth_graders.groupby(["school_name"]).mean()["math_score"]`

`math_scores_by_grade = pd.DataFrame({`

`               "9th": ninth_grade_math_scores,`

`               "10th": tenth_grade_math_scores,`

`               "11th": eleventh_grade_math_scores,`

`               "12th": twelfth_grade_math_scores})`

`math_scores_by_grade["9th"] = math_scores_by_grade["9th"].map("{:.1f}".format)`

`math_scores_by_grade["10th"] = math_scores_by_grade["10th"].map("{:.1f}".format)`

`math_scores_by_grade["11th"] = math_scores_by_grade["11th"].map("{:.1f}".format)`

`math_scores_by_grade["12th"] = math_scores_by_grade["12th"].map("{:.1f}".format)`

`math_scores_by_grade.index.name = None`

Math:
![math_scores_by_grade_df](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/math_scores_by_grade_formatted.png?raw=true)

Reading:
![reading_scores_by_grade_df](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/reading_scores_by_grade_formatted.png?raw=true)

### Scores by School Spending
To analyze the scores by school spending, a set of bins needed to be created so that the schools could be evenly divided into 4 sections. With these sections created, the per_school_capita was grouped by each of the sections and counted to see how many was in each bin. Additionaly, another column was added to the per_school_summary_df that described which bin each school fit into:

`spending_bins = [0, 585, 630, 645, 675]`

`per_school_capita.groupby(pd.cut(per_school_capita, spending_bins)).count()`

`group_names = ["<$586", "$586-630", "$631-645", "$646-675"]`

`per_school_summary_df["Spending Ranges (Per Student)"] = pd.cut(per_school_capita, spending_bins, labels=group_names)`

This was the resulting dataframe:

![per_school_summary_df_spending_ranges](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/per_school_summary_df_spending_ranges.png?raw=true)

With each school in its appropriate bin, it was then possible to group the math scores, reading scores, and percentages passing per bin and find the averages for each. This data was all collected and then placed into a dataframe:

`spending_math_scores = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Math Score"]`

`spending_reading_scores = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Reading Score"]`

`spending_passing_math = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Math"]`

`spending_passing_reading = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Reading"]`

`overall_passing_spending = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Overall Passing"]`

`spending_summary_df = pd.DataFrame({`

`          "Average Math Score" : spending_math_scores,`

`          "Average Reading Score": spending_reading_scores,`

`          "% Passing Math": spending_passing_math,`

`          "% Passing Reading": spending_passing_reading,`

`          "% Overall Passing": overall_passing_spending})`

Spending Summary:
![spending_summary_df_formatted](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/spending_summary_df_formatted.png?raw=true)

### Scores by School Size
In a similar fashion to the spending summary, the scores by school size was also created by creating bins, assigning them to schools, and calculating the averages within those bins. The result was the followning dataframe:

![size_summary_df](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/size_summary_df_formatted.png?raw=true)

### Scores by School Type
By utilizing the groupby() method to group schools by school type, this code also calculates the average statistics for each school type and puts them into the following dataframe:

![type_summary_df](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/type_summary_df_formatted.png?raw=true)

## Results
- How is the district summary affected?
  
  Original:
  ![district_summary_original](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/district_df_original.png?raw=true)
  
  Updated:
  ![district_summary_new](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/district_df.png?raw=true)
  
  As seen in the pictures above, the updated percentages for math, reading, and both are all slightly lower than in the original. This would suggest that the 9th grade class at Thomas High School was skewing the data to the right.

- How is the school summary affected?
  
  For almost all schools in the school summary, the changes to these calculations does not affect them. As their own scores and students have not been changed at all, they do not change in the school summary. However, for Thomas High School, this change does lower their average scores.

- How does replacing the ninth graders’ math and reading scores affect Thomas High School’s performance relative to the other schools?
  
  Original:
  ![top_5_original](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/top_5_schools_original.png?raw=true)
  
  Updated:
  ![top_5_updated](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/top_5_schools.png?raw=true)

  When compared to other schools, it appears as if Thomas High School does not change in its rankings. However, its percentages of passing scores have lowered slightly to give Cabrera High School more of a lead. 
  
- How does replacing the ninth-grade scores affect the following:
  
  - Math and reading scores by grade
  
  Math and Reading Scores by Grade remain almost completely intact when compared to the original. The only difference is that for Thomas High School on each of these dataframes, the 9th grade column is changed to the value NaN
  
  - Scores by school spending
  
  Original:
  ![scores_by_school_spending_original](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/spending_summary_df_formatted_original.png?raw=true)

  Updated:
  ![scores_by_school_spending_updated](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/spending_summary_df_formatted.png?raw=true)
  
  Because of the size of the bins and the rounding to single decimals and whole numbers, it appears that the scores by school spending has not changed at all. However, in reality, the scores for the $631-645 bin have dropped such a miniscule amount that it did not affect the rounding of these numbers.
  
  - Scores by school size
  
  Original:
  ![scores_by_size_original](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/size_summary_df_formatted_original.png?raw=true)
  
  Updated:
  ![scores_by_size_updated](![image](https://user-images.githubusercontent.com/80123282/177893426-dc52aa58-4977-4617-bcb0-c309352ccb66.png)
  
  In a similar fashion to the Scores by School Spending, the Scores by School Size remain at their previous numbers purely due to rounding. In reality, the medium school bin has dropped such a miniscule amount that it cannot affect the rounded numbers.
  
  - Scores by school type
  
  Original:
  ![scores_by_type_original](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/type_summary_df_formatted_original.png?raw=true)
  
  Updated:
  ![scores_by_type_updated](https://github.com/waciciarelli/School_District_Analysis/blob/main/Resources/type_summary_df_formatted.png?raw=true)
  
  And finally, for the same reason as the 2 previous analyses, the Scores by School Type does not appear to change despite the numbers for charter schools being slightly lower than before.
  
## Summary
There are notable changes to the analyses that can be noticed by the removal of the Thomas High School 9th Grade results. Most notably is that scores and pasing percentages in all analyses that had previously included these results have dropped. In the district analysis, scores and percentages accross the board have been lowered by around 0.1%. In the top 5 schools, the gap between Thomas High School and Cabrera High School has only grown. In the school spending, school size, and school type analyses, the scores and percentages have dropped by a negligable amount. This would sugges that theses results were skewing the averages upward. Another change that may not be so easily observed is that there is a lower population size for this analysis. This means that the other scores all have more sway in the results of this analysis.
