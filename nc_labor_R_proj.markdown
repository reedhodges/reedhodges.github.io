---
layout: page
title: Projects
permalink: /R-project/
---

# Using R to investigate employment trends in North Carolina, 1990-2022

In this portfolio project, we're going to look into employment trends in the state of North Carolina from 1990 to 2022.  Motivated by the dynamic nature of the job market and the critical role employment plays in economic health, this analysis seeks to identify factors influencing employment levels, wage disparities, and seasonal variations across different industries and ownership types. The dataset, sourced from the [state's quarterly and annual employment records](https://d4.nccommerce.com/), encompasses a wide range of variables.  Here are some that we will be looking at more closely:

- **Year**/**Quarter**: indicates the year, or quarter within the year, the datapoint is measured
- **Industry**: the industry sector according to the North American Industry Classification System (NAICS)
- **Ownership**: breaks down employment by private sector vs. local, state, and federal government 
- **Average employment**
- **Average weekly wages**

Through a series of statistical tests we'll explore the underlying trends and differences within the data. 

## Exploratory plot

It is natural to assume that employment in a given industry can change over time.  The rise of the World Wide Web at the turn of the millenium led to a huge increase in activity in the Information sector.  We can see that change, and compare it to an industry like Manufacturing over the same period, using a simple scatter plot.

The R script to produce the plot using the `ggplot2` library is:

```R
library(ggplot2)
library(dplyr)

data <- read.csv("filepath/data_annual.csv")

data$Year <- as.numeric(as.character(data$Year))

process_industry_data <- function(df, industry_name) {
  df %>%
    filter(Industry == industry_name) %>%
    group_by(Year) %>%
    summarise(Average_Annual_Employment = mean(`Average.Employment`, na.rm = TRUE)) %>%
    mutate(Percent_of_Max = (Average_Annual_Employment / max(Average_Annual_Employment, na.rm = TRUE)),
           Industry = industry_name)
}

information_data <- process_industry_data(data, 'Information')
manufacturing_data <- process_industry_data(data, 'Manufacturing')

combined_data <- bind_rows(information_data, manufacturing_data)

ggplot(combined_data, aes(x = Year, y = Percent_of_Max, color = Industry)) +
  geom_line() + 
  geom_point() + 
  theme_minimal() + 
  scale_y_continuous(labels = scales::percent_format()) +
  labs(title = "Average Annual Employment as Percentage of Industry Maximum",
       subtitle = "Comparing Information vs. Manufacturing Industries in North Carolina",
       x = "Year",
       y = "% of Industry's Maximum Employment")
```

Here we have a function that takes the annual data and calculates the average employment for a given industry over the course of a year, and makes a column of data for the percentage of the maximum employment over the time period.  We then plot the data for the two industries.

![trend-over-time](images/R-trend-over-time.png)

You can clearly see the boom in information jobs over the course of the 1990s, while manufacturing jobs have fallen to about 50% of their 1990 levels.

## Average weekly wages for different ownership types

One might expect that, for example, private sector jobs pay more than local government jobs.  We can perform a statistical test to check this.

### The ANOVA Test: A Primer

To examine the differences in wages among these groups, the Analysis of Variance (ANOVA) test is employed. ANOVA is a statistical method used to compare the means of three or more independent groups to determine if at least one group mean is statistically different from the others. It is particularly well-suited for this analysis for several reasons:

- **Multiple Groups**: ANOVA is designed to handle comparisons across more than two groups, making it ideal for our investigation into the four types of ownership.
- **Homogeneity of Variances**: ANOVA assumes that all groups have similar variances, an assumption that fits well with the structured nature of employment sectors where wage policies might introduce some level of uniformity.
- **Normality**: The test assumes that the data within each group are normally distributed, which is a reasonable assumption for large datasets like employment wages, according to the Central Limit Theorem.

In the context of this study, ANOVA is particularly appropriate for a few key reasons:

- **Comparative Analysis**: We are interested in comparing the average wages across multiple ownership categories, not just examining the relationship between two variables or comparing two groups.
- **Identifying Variance**: Beyond identifying differences, ANOVA helps in understanding how much variance in wages can be attributed to the type of ownership, offering insights into the impact of ownership on wage structures.
- **Foundation for Further Analysis**: Results from ANOVA can lead to more detailed post-hoc analyses, allowing us to pinpoint exactly which ownership types differ from each other, further enriching our understanding.

Our hypotheses for this test are as follows.

- **Null Hypothesis (H0)**: There is no significant difference in the average weekly wages among the different ownership types (Federal Government, State Government, Local Government, and Private). This hypothesis posits that any observed differences in average wages across these groups are due to random chance rather than a systematic effect of ownership type.

- **Alternative Hypothesis (H1)**: There is at least one significant difference in the average weekly wages between the ownership types. This hypothesis suggests that the type of ownership does indeed have an impact on wage levels, and the observed differences in wages are not merely the result of random variation.


### Results of the ANOVA test

It is simple to perform the ANOVA test in R:

```R
library(ggplot2)
library(dplyr)

data <- read.csv("/Users/reedhodges/Documents/GitHub/nc_labor_data/qcew_data/data_quarterly.csv")

data$Ownership <- as.factor(data$Ownership)
data$Average.Weekly.Wage <- as.numeric(data$Average.Weekly.Wage)

data_clean <- filter(data, !is.na(Average.Weekly.Wage))

# ANOVA to test for differences in average weekly wages across different ownership types
anova_result <- aov(Average.Weekly.Wage ~ Ownership, data = data_clean)
summary(anova_result)
```

The results of the test are as follows.

| Source    | Df   | Sum Sq     | Mean Sq  | F value | Pr(>F)     |
|-----------|------|------------|----------|---------|------------|
| Ownership | 3    | 6.073e+07  | 20244888 | 119.6   | <2e-16     |
| Residuals | 7074 | 1.198e+09  | 169282   |         |            |


What do these values mean?

- **Df (Degrees of Freedom)**: 
  - **Ownership**: 3. This represents the number of levels in the ownership variable minus one. Since there are four types of ownership, the degrees of freedom are three, indicating the number of comparisons that can be made.
  - **Residuals**: 7074. This number represents the degrees of freedom for the error term, calculated as the total number of observations minus the number of levels in the ownership variable.

- **Sum Sq (Sum of Squares)**: 
  - **Ownership**: 6.073e+07. This value indicates the total variation in average weekly wages attributable to differences between the ownership types.
  - **Residuals**: 1.198e+09. This is the total variation in average weekly wages that is not explained by the ownership types, essentially the error or residual variation.

- **Mean Sq (Mean Square)**: 
  - **Ownership**: 20244888. The mean square for ownership is the sum of squares for ownership divided by its degrees of freedom, measuring the average variation in wages between the different ownership types.
  - **Residuals**: 169282. The mean square for residuals is the sum of squares for residuals divided by its degrees of freedom, representing the average variation in wages within the ownership groups.

- **F value**: 119.6. This statistic measures the ratio of the mean square for ownership to the mean square for residuals. An F value of 119.6 significantly exceeds 1, indicating that the variation between group means is substantially greater than the variation within the groups, suggesting that ownership type has a strong effect on wage levels.

- **Pr(>F)**: <2e-16. This p-value is the probability of observing an F value as extreme as, or more extreme than, what was observed if the null hypothesis were true (no difference in means across ownership types). A p-value less than 2e-16 provides strong evidence against the null hypothesis, indicating a statistically significant difference in average weekly wages across the different ownership types.

The ANOVA test results strongly suggest that the type of ownership is a significant factor in determining wage levels, with a very low p-value indicating that the observed differences in mean wages among ownership types are highly unlikely to have occurred by chance. The significant F value reinforces this conclusion, highlighting that the variance between the groups is much larger than the variance within the groups, thus affirming the impact of ownership type on wage disparities.  We reject the null hypothesis.

Given these results, further post-hoc analysis, such as Tukey's HSD test, is warranted to identify specifically which ownership types differ in terms of average weekly wages, providing detailed insights into the nature of these disparities.

### Tukey's Honest Significant Difference (HSD) Test

Following the ANOVA test, which indicated significant differences in average weekly wages across ownership types, the Tukey's Honest Significant Difference (HSD) test was conducted. This post-hoc analysis is crucial for interpreting ANOVA results more granularly by comparing all possible pairs of groups to identify exactly where the significant differences lie.

Tukey's HSD test is a widely used method for performing multiple pairwise comparisons between group means after a one-way ANOVA has found a significant F-statistic. This test controls the family-wise error rate, thereby maintaining the probability of making one or more Type I errors at a desired level, typically 0.05. The key features of Tukey's HSD test include:

- **Pairwise Comparisons**: It compares the means of every group with every other group.
- **Error Rate Control**: Unlike conducting multiple t-tests, Tukey's HSD maintains the overall error rate, making it more reliable for comparing multiple groups.
- **Confidence Intervals**: For each pairwise comparison, it provides a confidence interval for the difference in means, offering a range of plausible values for the true difference.

After identifying that ownership type influences wage disparities, it becomes essential to pinpoint which types of ownership differ from each other. Tukey's HSD is the appropriate next step because:

- It allows for a comprehensive comparison across all ownership categories.
- It ensures that the risk of false discoveries is controlled when making multiple comparisons.
- It provides actionable insights that can directly inform policy adjustments, wage standardization efforts, or further research into the causes of these disparities.

The results from Tukey's HSD test will typically include:

- **Difference in Means**: For each pair of groups, the difference in their means along with the direction of the difference.
- **Confidence Interval**: A 95% confidence interval for the difference in means. If this interval does not include zero, the difference is considered statistically significant.
- **Adjusted P-value**: The p-value adjusted for the number of comparisons being made. A low p-value (<0.05) indicates a significant difference between the pair of groups.

Interpreting these results allows us to understand not just if but where significant wage disparities exist between different types of ownership.  The R script to perform the test is:

```R
if (summary(anova_result)[[1]][["Pr(>F)"]][1] < 0.05) {
  tukey_result <- TukeyHSD(anova_result)
  print(tukey_result)
}
```

It has the following output.

| Comparison                                 | Difference | Lower Bound | Upper Bound | Adjusted P-value |
|--------------------------------------------|------------|-------------|-------------|------------------|
| Local Government - Federal Government      | -259.70534 | -297.43360  | -221.97707  | 0e+00            |
| Private - Federal Government               | -87.20839  | -122.10058  | -52.31619   | 0e+00            |
| State Government - Federal Government      | -174.65524 | -214.33931  | -134.97117  | 0e+00            |
| Private - Local Government                 | 172.49695  | 139.42449   | 205.56941   | 0e+00            |
| State Government - Local Government        | 85.05010   | 46.95617    | 123.14403   | 1e-07            |
| State Government - Private                 | -87.44685  | -122.73412  | -52.15958   | 0e+00            |

Here are the key takeaways:

- **Difference**: Represents the difference in mean wages between the two groups being compared. A negative value indicates that the first group has a lower average wage than the second group.
- **Lower Bound and Upper Bound**: These columns provide the 95% confidence interval for the difference in means. If this interval does not include zero, the difference is considered statistically significant, suggesting that the wage disparity between the groups is not due to random chance.
- **Adjusted P-value**: Reflects the significance of the difference in means after adjusting for the multiple comparisons being made. A value close to 0 indicates a very significant difference between the groups.

The results indicate significant wage disparities between various types of ownership. Specifically, Federal Government employees tend to have higher wages compared to Local Government, Private, and State Government sectors. Additionally, the Private sector shows significantly higher wages than the Local Government, underscoring the influence of ownership type on wage levels.

### Plot

We can generate a box-and-whisker plot for the wages across the ownership types.

```R
ggplot(data_clean, aes(x = Ownership, y = Average.Weekly.Wage, fill = Ownership)) +
  geom_boxplot() +
  theme_minimal() +
  labs(title = "Comparison of Average Weekly Wages by Ownership Type",
       x = "Ownership Type",
       y = "Average Weekly Wage ($)") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

![ownership-plot](images/R-weekly-wages-ownership.png)

The dots indicate outliers.