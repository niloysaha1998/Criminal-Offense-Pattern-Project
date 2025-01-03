# Criminal-Offense-Pattern-Project
---

## Title

### ***'Criminal Offense Trends: A Comprehensive Analysis of Demographic Variables in Offense Patterns'***

```{r}
library(dplyr)
library(tidyverse)
```

## Background

This research is driven by the need to understand potential systemic biases and inequities, offering data-driven insights that can inform policy reforms, promote fairness, and support more equitable practices within the justice system. The outcomes of this study contribute to the broader discourse on social justice, emphasizing the importance of evidence-based approaches in addressing disparities and fostering a transparent, equitable society. This analysis investigates the relationship between offense patterns, race, and age demographics to identify significant patterns and disparities within the criminal justice system.

## Dataset Description

```{r}
#Understanding the dataset


library(readr)
Crime_Data <- read_csv("C:/Users/niloy/OneDrive/Desktop/Group Files/Group Files/Crime Data.csv")
str(Crime_Data)
dim(Crime_Data)

```

**Source:**

The dataset is provided by the **Texas Department of Criminal Justice (TDCJ)**, titled *"High-Value Dataset, July 2024." (link :* <https://data.texas.gov/dataset/High-Value-Dataset-July-2024/qz8r-du54/about_data%5D>)

-   Rights: As the dataset is publicly available through governmental portal, there is no restrictions in sharing and usage.

-   Limitations: The dataset only covers offenses for a specific period. It also pertains to only offenders under the jurisdiction of Texas Department of Justice. Certain racial groups or offense types may have small sample sizes, impacting the robustness of statistical analysis.The categorization of offenses might be broad or subjective, which could influence the interpretation of results.

**Size:**

Number of Rows: 134579

Number of Columns: 20

**key Features:**

The dataset contains the following key attributes:

-   Race: Categorical variable indicating the racial group of the offender (e.g., White, Black, Hispanic).

-   Age: Numeric variable representing the age of the offender.

-   TDCJ Offense: Detailed description of the offense committed by the offender.

-   Parole Decision: Outcome of parole hearings (e.g., Approved, Denied).

-   Gender: Categorical variable representing the gender of the offender.

Other Demographic Details: Additional information such as sentence date, County, and sentence length.

#### **Purpose of the Dataset:**

The dataset is intended for analyzing trends and disparities in the criminal justice system based on demograhic and offense patterns.

## Analysis

### ***Part 1 : Data Cleaning***

```{r}

#Understanding the dataset


Crime_Data <- read_csv("C:/Users/niloy/OneDrive/Desktop/Group Files/Group Files/Crime Data.csv")
str(Crime_Data)
dim(Crime_Data)


#checking for need of cleaning
colSums(is.na(Crime_Data)|Crime_Data == "")



#cleaning the data

#removing NA values
cleaned_crime <- Crime_Data %>%
  na.omit()
#removing blank values
cleaned_crime <- cleaned_crime[apply(cleaned_crime != "" , 1 , all ), ]




#Checking the number of blanks and NAs after cleaning to confirm
dim(cleaned_crime)
colSums(is.na(cleaned_crime)|cleaned_crime == "")





#updating column names
names(cleaned_crime) <- tolower(names(cleaned_crime))
names(cleaned_crime)




View(cleaned_crime)

```

**Comment:** In this first step, we have initiated the cleaning procedure over the dataset as it has many rows with NA and blank values which can be observed through the code run.After cleaning the dataset, we have called it "cleaned crime" and checked if the dataset is fully cleaned of blank rows and NA values. As all columns are cleaned of these values, We updated the column names also to analyse effectively. The "final_crime_Date" is the final dataset after cleaning.

### ***Part 2: Data Preparation***

```{r}


#Selection of relevant columns

final_crime_data <- cleaned_crime %>%
  select(1,3,5,6,7,14,17,18,19, 20)
colnames((final_crime_data))




#Unique offenses to list for categorisation

unique_offenses <-  unique(final_crime_data$`tdcj offense`)





#Function for categorizing offenses

categorise_function <- function(offense) {
  if (grepl("MURDER|ASSAULT|ROBBERY|HOMICIDE|KIDNAPPING", offense, ignore.case = TRUE)) {
    return("violent crimes")
  } else if(grepl("BURGLARY|THEFT|ARSON",offense, ignore.case = TRUE)) {
  return("property crimes")
  }  else if (grepl("DRUG|SUBSTANCE",offense, ignore.case = TRUE)){
  return("drug offenses")
  }   else if(grepl("SEXUAL|INDECENCY", offense, ignore.case = TRUE)) {
    return("sexual crimes")
  }   else if (grepl("WEAPON|FIREARM", offense, ignore.case = TRUE)) {
    return("Weapon Crimes")
  }   else {
    return("other crimes")
  }
}



#Creating the new column as "Offense Category" by function

final_crime_data <- final_crime_data %>%
  mutate(Offense_Category = sapply(`tdcj offense`, categorise_function))




#Count NA and blank values after new column added

colSums(is.na(final_crime_data)|final_crime_data == "")

head(final_crime_data)

```

**Comment:** After cleaning the dataset, we went on to prepare the dataset for analysis. we have added a new column named "Offense catgory" to categorise similar offenses and make our analysis more understandable on a general perspective. And we have checked again for blank rows and rows with 'NA' values in the new column. Finally, We can now work on this cleaned and prepared dataset for analysis and interpretation.

### *Part 3: Descriptive Analysis*

```{r}



#converting columns into factors for summary 
final_crime_data$race <- as.factor(final_crime_data$race)
final_crime_data$Offense_Category <- as.factor(final_crime_data$Offense_Category)



#Summary of crime related statistic of convicts by Race
count_of_offenses <- final_crime_data %>%
  group_by( race , Offense_Category) %>%
  summarise(count = n(), .groups = "drop")
print(count_of_offenses)




#Summary of age related stistics for convicts
convict_age_summary <- final_crime_data %>%
group_by(race, Offense_Category)%>%
summarise(
mean_age = mean(age, na.rm =TRUE),
median_age = median(age, na.rm=TRUE),
min_age = min(age, na.rm = TRUE),
max_age = max(age, na.rm=TRUE),
range_age = max_age - min_age
)
print(convict_age_summary)






```

**Comment:** As we want to explore the relationship among race, age and offense categories. We first made the categorical variables as factor for easier detection of values. Then we analysed to find the number of offenses of one type by each race. It shows offense categories as 'other crimes' in high number as it contains offenses from many scenarios that can not be mentioned alone. Also we have analysed the datset to find out the summary statistics of age based on race and offense pattern. It shows that people committing any type of crime from any race belong to mostly 40-50 age group.

### *Part 4: Visualisation*

```{r}

#Bar Plot- Visualisation of the relationship between race and offence type

ggplot(count_of_offenses, aes(x = race, y = count, fill = Offense_Category)) +
  geom_col(position = "dodge") +  
  labs(
    title = "Type of Offenses by Race",
    x = "Race",
    y = "Count of Offenses",
    fill = "Offense Category"
  ) +
  theme_minimal()  + facet_wrap(~ Offense_Category, scales = "free_y")           






```

**Comment:** From the graph, we can see that Blacks, Hispanics and Whites dominate the chart data in terms of offense types. For example, drug offenses are mostly carried by the Whites and Blacks but the sexual crimes are committed by Hispanics in high number. In terms of violent crimes, Blacks are the most reported convicts much higher than both Whites and Hispanics.

```{r}

#Heatmap - Visualisation of the relationship among age, race and offense type

ggplot(convict_age_summary, aes(x = race, y = Offense_Category, fill = mean_age)) +
  geom_tile(color = "white") +
  scale_fill_gradient(low = "mistyrose", high = "darkred") +
  labs(title = "Heatmap of Mean Age Based on Race and Offense Type", x = "Race", y = "Offense Category", fill = "Mean Age") +
  theme_minimal()



```

**Comment:** From the heatmap, we can see that ages of most convicts regardless of race fall between 40-60 bracket. But offenders committing weapon crimes tend to be younger, while those involved in sexual crimes tend to be older across most racial groups

```{r}
#Boxplot - visualisation of the relationship of mean age with offense patterns and race


ggplot(final_crime_data, aes(x = race, y = age, fill = race)) +
  geom_boxplot() +
  labs(title = "Distribution of Mean Age by Offense Category and Race", x = "Offense Category", y = "Mean Age", fill = "Race") +
  theme_minimal() + facet_wrap(~Offense_Category)


```

**Comment:** From the boxplot, we can see that the average age range revolve around 40-60 for any convicts for any offense type. But there are certain distinct differences as well. For example, the average age of Hispanic convicts in drug offenses is lower than black or white convicts. On the other hand, the average age of black convicts is lower than any other races in violent crimes.

### *Part 5: Hypothesis Testing*

Null Hypothesis : The mean age does not differ across race groups

Alternate Hypothesis: The mean age differs significantly across race groups

```{r}

#Hypothesis Testing 

#ANOVA_test
anova_result <- aov(age ~ race, data = final_crime_data)
anova_summary <- summary(anova_result)

#checking p value
p_value <- anova_summary[[1]][1,"Pr(>F)"] 
p_value

#Hypothesis Results
if (p_value < 0.05) {
  cat("There is a significant difference in mean age across races.\n")
} else {
  cat("There is no significant difference in mean age across races.\n")
}


#Detecting the races between which age difers
Pairwise_race <- TukeyHSD(anova_result)
print(Pairwise_race)
plot(Pairwise_race,las = 1, col = "blue")



```

**Comment:** As we can see, the hypothesis is that there is no significant difference in mean age across race groups. But after doing the hypothesis testing, we can see that the p value is less than .05, meaning that there is significant difference in mean age across race groups. To analyse, the race groups with most differences in mean age, we have also conducted Tukey's HSD test. Comparisons where the **confidence interval does not cross 0** indicate a **significant difference** in mean ages between the two race groups. For example, Blacks and Asians convicts have signficant difference in mean age.

## **Conclusion/Insights**

After a multiphased analysis combining exploratory data analysis, hypothesis testing and visualisation, we have found important insights in the relationship among offense types and offender demographics. The mean age of offenders are different across different race groups. For example, hispanic convicts tend to be younger than black or white convicts. Also, the type of offenses that are committed in most numbers vary among race groups. For example, drug offenses are committed by mostly whites and blacks while hispanics commit sexual crimes more than any other group. In addition, the black convicts tend to commit violent crimes more than any other and they tend to be younger in age as well. Another highlight is that, Offenders committing weapon crimes tend to be younger, while those involved in sexual crimes tend to be older across most racial groups. The findings suggest a need for further investigation into potential systemic biases within the justice system. Addressing these disparities could inform criminal justice policies and support efforts to promote fairness and equity.
