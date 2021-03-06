---
title: "Exploring The BRFSS Data"
output: 
  html_document: 
    fig_height: 4
    highlight: pygments
    theme: spacelab
    keep_md: true
---

## Setup

### Load packages


```r
library(ggplot2)
library(dplyr)
library(reshape2)
library(gridExtra)
library(scales)
```

### Load data


```r
load("brfss2013.RData")
```



* * *

## Part 1: Data
The Behavioral Risk Factor Surveillance System (BRFSS) is a collaborative project between all of the states in the United States (US) and participating US territories and the Centers for Disease Control and Prevention (CDC). BRFSS is an ongoing surveillance system designed to measure behavioral risk factors for the non-institutionalized adult population (18 years of age and older) residing in the US. 

The BRFSS objective is to collect uniform, state-specific data on preventive health practices and risk behaviors that are linked to chronic diseases, injuries, and preventable infectious diseases that affect the adult population. Factors assessed by the BRFSS in 2013 include tobacco use, HIV/AIDS knowledge and prevention, exercise, immunization, health status, healthy days - health-related quality of life, health care access, inadequate sleep, hypertension awareness, cholesterol awareness, chronic health conditions, alcohol consumption, fruits and vegetables consumption, arthritis burden, and seatbelt use. 

BRFSS conducts both landline telephone- and cellular telephone-based surveys. In conducting the BRFSS landline telephone survey, interviewers collect data from a randomly selected adult in a household. In conducting the cellular telephone version of the BRFSS questionnaire, interviewers collect data from an adult who participates by using a cellular telephone and resides in a private residence or college housing.

Since the data was collected by survey, there are strict limits to how the data can be interpreted.  Relationships between data points can be suggested, but not causality.  Furthermore, the data is meant to serve as a sample of the larger US population.  Collecting surveys via phone creates some strong biases in this area.  Landline ownership was in rapid decline by 2013, particularly among younger, wealthier and urban demographics.  The data does include data via cell phone, but it is not proportional to the larger population, 26.7% of respondents vs. 41% nationally (CDC:  https://www.cdc.gov/nchs/data/nhis/earlyrelease/wireless201407.pdf ).

Furthermore, a common problem with surveys in general is that not everyone is willing to take them.  This is compounded by the fact that the survey is limited to collection via phone.  This could have been mitigated by expanding collection to include things like mail-in and online surveys, as well as face-to-face interviews.

Finally, this is a large survey of data points that would be very sensitive to many respondents.  Providing accurate answers on things like alcohol consumption, income, and feelings on race (among many such questions) may be difficult for many respondents.

For these reasons, the data represents an excellent starting point for creating and refining questions about national health issues, but those questions would have to be answered through other, better designed research methods.



* * *

## Part 2: Research questions

**Research quesion 1:**
**_Is there a relationship between being uninsured and other risky behavior?_**

The Affordable Care Act (ACA), colloquially known as "Obamacare", sought to reduce the number of uninsured and reduce individual healthcare costs in the United States.  In a national social healthcare scheme, all citizens would be automatically covered.  The ACA instead sought to retain the marketplace of private insurance as the primary source of coverage.  To increase the universality of coverage, the ACA leveraged subsidies for low-income consumers and penalties for opting out of coverage (to maintain viability of the system for insurance companies).  The theory behind the ACA is that by making insurance more affordable, and having it be too expensive to not have insurance, people would be more inclined to purchase coverage.

But are their other factors driving decision-making around purchasing insurance besides cost?  This question examines one of these possible factors.



**Research quesion 2:**
**_Do people who have experienced emotional distress because of how they have been treated due to their race get more or less sleep then people who have not reported such distress and how is this further impacted by their reported race?_**

Historically, a common characteristic of diverse, multi-racial societies has been deeply engrained, unresolved racial tension.  The United States is no different.  In addition to the natural tensions that can arise between people who are different from one another, the US has a long history of systemic and institutional disempowerment, oppression and violence committed by the majority against minority populations.  It was founded on the genocide of the native population with an economy powered by slave labor.  Currently, xenophobia and racism is driving immigration policy more than economic theory or public safety concepts rooted in data.

There is also a pattern of the United States moving towards more "evolved" policies.  But these moves rarely are accompanied with an effort to address the underlying beliefs that drove the original policy.  Thus, the wounds are at best left unhealed, at worst deepened.  Slavery was abolished, but Jim Crow laws were allowed to flourish for the better part of a century.

So the natural tensions between racial subsections of the US are exacerbated by a dark history and unhealed wounds.  

It is from these assumptions that I wanted to explore if these tensions impact the health of individuals.



**Research quesion 3:**
**_Are people who have experienced emotional distress because of how they have been treated due to their race more or less likely to report high blood pressure than people who have not reported such distress and how is this further impacted by their reported race?_** 

This question is a follow up to question two.  The methodology is similar, but this time, I evaluate the datasets against whether or not respondents reported being diagnosed with high blood pressure.


* * *

## Part 3: Exploratory data analysis


**Research quesion 1:**
**_Is there a relationship between being uninsured and other risky behavior?_**

To explore this question, I created a subset of the data.  First, I pulled people who reported being without coverage in the past 12 months.  Then, to reduce the impact of the cost of coverage, I eliminated any making less than $50,000 per year from the dataset.  This new subset was the "test" dataset and the full BRFSS dataset is the "control".


```r
#Extract without coverage at some point in past 12 months
Q1Data<-subset(brfss2013[(brfss2013$nocov121=="Yes" & (!is.na(brfss2013$nocov121))),])
#Isolate only variables needed
Q1Data<-Q1Data[,c(50,69,95,123 )]
#Retain people making 50k or more to limit finances as confounding variable to not having insurance
Q1Data<-subset(Q1Data[((Q1Data$income2=="Less than $75,000" |
                          Q1Data$income2=="$75,000 or more") & 
                         (!is.na(Q1Data$income2))),])
```

Next, I looked at smoking patterns among the respondents.  The science is well-established and conclusive that smoking is very harmful to human health.  Furthermore, the messaging around this is extremely comprehensive.  Children are taught in school about the dangers of smoking and advertising to that effect are found in almost every media form imaginable.  Smoking is dangerous and it is difficult for someone in the United States to not know that.

The corollary to this is that if one does smoke, it can be interpreted as risk-taking behavior.  The danger is established and known.  Choosing that danger is to choose a known risk.

The BRFSS dataset includes the variable smokday2 which evaluates the frequency of days currently smoking. I created a plot that compares the smoking frequency between the test dataset and the full BRFSS dataset.


```r
##Create summary dataset that describes smoking frequency among no coverage/high income respondents
Q1Smoke<-Q1Data %>% 
  group_by(smokday2) %>%
  summarise(count=n()) %>%
  mutate(freq = count / sum(count))
##Create summary dataset that describes smoking frequency among all BRFSS respondents
Full.Smoke<-brfss2013 %>% 
  group_by(smokday2) %>%
  summarise(count=n()) %>%
  mutate(freq = count / sum(count))
##Combine smoking freq datasets, renameing columns and eliminating NA's
compare.smoke<-bind_cols(Q1Smoke[,c(1,3)], Full.Smoke[,3])
colnames(compare.smoke)<-c("frequency", "No Coverage/High Income", "All Respondents")
compare.smoke<-compare.smoke[c(1:3),]
compare.smoke2 <- melt(compare.smoke, id.vars='frequency')
##store plot for combined dataset
plotsmoke<-ggplot(compare.smoke2, aes(x=frequency, y=value, fill=variable)) +
  geom_bar(stat='identity', position='dodge')+
  labs(x="Smoking Frequency", y="Percentage of Respondenets", 
       title="Smoking Among BRFSS Respondents") +
  theme(axis.text.x = element_text(angle=90)) +
  scale_fill_discrete(name="Respondent Type")
```

I also wanted to look at seatbelt usage.  Like smoking, not using a seatbelt is a well-established dangerous choice with very comprehensive messaging.  It should also serve as an indicator of risk-taking behavior.  It is interesting to note that while one choosing to smoke is risky, not to use a use a seatbelt is the risky choice.  One is an action, the other an inaction.

The BRFSS dataset includes the variable seatbelt which evaluates the frequency of seatbelt use.  I created a plot that compares the seatbelt usage between the test dataset and the full BRFSS dataset.

```r
##Create summary dataset that describes seatbelt usage among no coverage/high income respondents
Q1Seatbelts<-Q1Data %>% 
  group_by(seatbelt) %>%
  summarise(count=n()) %>%
  mutate(freq = count / sum(count))
##Create summary dataset that describes smoking frequency among all BRFSS respondents
Full.Seatbelts<-brfss2013 %>% 
  group_by(seatbelt) %>%
  summarise(count=n()) %>%
  mutate(freq = count / sum(count))
##Combine smoking freq datasets, renameing columns and eliminating NA's
compare.belts<-bind_cols(Q1Seatbelts[,c(1,3)], Full.Seatbelts[,3])
colnames(compare.belts)<-c("Usage", "No Coverage/High Income", "All Respondents")
compare.belts<-compare.belts[c(1:5),]
compare.belts2 <- melt(compare.belts, id.vars='Usage')
##store plot for combined dataset
plotbelts<-ggplot(compare.belts2, aes(x=Usage, y=value, fill=variable)) +
  geom_bar(stat='identity', position='dodge')+
  labs(x="Seat Belt Usage", y="Percentage of Respondenets", 
       title="Seat Belt Usage Among BRFSS Respondents") +
  theme(axis.text.x = element_text(angle=90)) +
  scale_fill_discrete(name="Respondent Type")
```
And here are the resulting plots:

```r
grid.arrange(plotsmoke, plotbelts, ncol=2)
```

![](intro_data_prob_project_files/figure-html/unnamed-chunk-4-1.png)<!-- -->
There is a clear pattern of people who are uninsured (and being able to afford it) being more inclined towards risky behavior.  This group of respondents was more inclined to smoke, and with greater frequency, than the general population.  They were also less likely to use a seatbelt.

The separation between smoking and seatbelt patterns between to the two datasets (test group vs. full BRFSS) is pretty narrow.  The pattern suggests a relationship between risk-taking behavior and decision-making towards purchasing insurance coverage, but it is difficult to gauge the strength of that pattern (at least with the tools thus far presented in coursework of which this project is a part).  

The results are intriguing, though, and suggest that this is a worthy topic for further research, perhaps with more targeted experimental design to better establish causality.




**Research quesion 2:**
**_Do people who have experienced emotional distress because of how they have been treated due to their race get more or less sleep then people who have not reported such distress and how is this further impacted by their reported race?_**

I began by creating two subsets of data based on the variable rremtsm2.  This variable tracked responses to the question "Within the past 30 days, have you felt emotionally upset, for example angry, sad, or frustrated, as a result of how you were treated based on your race?  One dataset held respondents who said yes, the other held the no's.


```r
#Eliminate unneeded variables
Q2Data<-brfss2013[,c(27, 183, 222)]

#Creating dataset (Q2Race.Emotion) of people emotionally upset by treatment due to race over past 30 days 
#and seperate into imputed race
Q2Race.Emotion<-Q2Data[Q2Data$rremtsm2 =="Yes" & !is.na(Q2Data$rremtsm2),]

#extract same data for respondents not upset by racial treatment
Q2Not.Upset<-Q2Data[Q2Data$rremtsm2 =="No" & !is.na(Q2Data$rremtsm2),]
```
Next, I created a summary table and plotted the results to compare the two data sets, separated by reported race.


```r
#create summary table for upset respondents
Race.Sleep<-Q2Race.Emotion[,c(1,3)]
sleep.summary<-Race.Sleep[!is.na(Race.Sleep$sleptim1),]
sleep.summary<-sleep.summary %>%
  group_by(X_imprace) %>% summarise_each(funs(mean, sd))
```

```
## `summarise_each()` is deprecated.
## Use `summarise_all()`, `summarise_at()` or `summarise_if()` instead.
## To map `funs` over all variables, use `summarise_all()`
```

```r
sleep.summary$group<-"test"
#create summary table for not upset respondents
NotUpset.Sleep<-Q2Not.Upset[,c(1,3)]
notupset.summary<-NotUpset.Sleep[!is.na(NotUpset.Sleep$sleptim1) &
                                   !is.na(NotUpset.Sleep$X_imprace),]
notupset.summary<-notupset.summary %>%
  group_by(X_imprace) %>% summarise_each(funs(mean, sd))
```

```
## `summarise_each()` is deprecated.
## Use `summarise_all()`, `summarise_at()` or `summarise_if()` instead.
## To map `funs` over all variables, use `summarise_all()`
```

```r
notupset.summary$group<-"control"
#merge tables
sum.table<-full_join(sleep.summary, notupset.summary)
```

```
## Joining, by = c("X_imprace", "mean", "sd", "group")
```

```r
#plot summary table
ggplot(sum.table, aes(x=X_imprace, y=mean, fill=group))+  
  geom_col(position="dodge")+
  coord_cartesian(ylim=c(5.5, 7.5))+
  scale_x_discrete(labels=c("White","Black", "Asian", "Native", "Hispanic", "Other"))+
  labs(x="Race", y="Hours of Sleep(Mean)", 
       title="Hours of Sleep by Race", subtitle="comparing respondents by reaction to treatment due to race") +
  scale_fill_discrete(name="Emotionally Upset By Treatment", labels=c("No", "Yes"))
```

![](intro_data_prob_project_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
There is a clear distinction between the reported sleep between the two groups.  On average (among the racial categories), respondents who report emotional distress due to treatment based on their race got 32 minutes less sleep than those who reported not experiencing such stress. Native Americans show the biggest difference with 78.3 fewer minutes of sleep, Hispanics the least difference of only 6.2 fewer minutes.  Surprisingly, Whites showed the third highest impact, with a difference of 28.1 minutes less sleep.

These results are intriguing and suggest that there is a relationship between racial tension on a national/cultural level and individual health.

Beyond that, it is difficult to describe that relationship with much detail or certainty.  First, the sample group (the two datasets) are relatively small.   Only 3,942 respondents answered yes or no to this BRFSS question.  Of those, only 179 answered yes.

Many people are very sensitive to this issue and this may have contributed to the low response rate.  If this is the case, it is possible that those who have experience distress around this issue may be less comfortable answering this question than the rest of the population.  Since 2013 (the year this data was collected), the national conversation around race has gotten more prominent and the voices in that conversation have become more diverse.  It is possible that these results would be different for in an updated survey executed in the current climate.

At the same time, there has been a trend of Caucasians, particular middle-aged men, expressing feelings of racial disadvantage.  Recent polling suggests this is a result of general economic distress, rather than specific racial oppression.  Further research could explore the relationship between feelings of racial tension and economic uncertainty.

Finally, causality would be difficult to identify without very careful targeting of a new experimental model.  Are people getting less sleep because of stress due to how they are treated?  Is the loss of sleep due to a general feeling of stress and that this feeling increases sensitivity to feelings of racial distress?  Clearly, careful consideration would have to be applied to avoid these and other confounding factors.



**Research quesion 3:**
**_Are people who have experienced emotional distress because of how they have been treated due to their race more or less likely to report high blood pressure than people who have not reported such distress and how is this further impacted by their reported race?_**

As with question two, I began by subsetting the data into two databases:  one holding yes responses to rremtsm2, the other holding no's. 


```r
#Eliminate unneeded variables
Q3Data<-brfss2013[,c(28, 183, 222)]

#Creating dataset (Q3Race.Emotion) of people emotionally upset by treatment due to race over past 30 days 
#and seperate into imputed race
Q3Race.Emotion<-Q3Data[Q3Data$rremtsm2 =="Yes" & !is.na(Q3Data$rremtsm2),]

#extract same data for respondents not upset by racial treatment
Q3Not.Upset<-Q3Data[Q3Data$rremtsm2 =="No" & !is.na(Q3Data$rremtsm2),]
```
I then created two summary tables that grouped the datasets by race and presented counts and percentages for responses to the bphigh4 variable.   This variable tracks adults who have been told they have high blood pressure by a doctor, nurse, or other health professional.  

```r
#create summary table for upset respondents
Q3.test<-Q3Race.Emotion %>% 
  group_by(X_imprace, bphigh4) %>%
  tally() %>%
  mutate(freq = n / sum(n))
#drop pregnancy induced hbp and borderline
Q3.test<-Q3.test[c(1,3:5,7:14),]

#create summary table for respondents not upset
Q3.control<-Q3Not.Upset %>% 
  group_by(X_imprace, bphigh4) %>%
  tally() %>%
  mutate(freq = n / sum(n))
#drop pregnancy induced hbp and borderline
Q3.control<-Q3.control[c(1,3,6,8:11,13,14,16,19,20),]

#add label variable for use after merge
Q3.test$group<-"test"
Q3.control$group<-"control"

#merge tables
Q3.summary<-full_join(Q3.test, Q3.control)
```

```
## Joining, by = c("X_imprace", "bphigh4", "n", "freq", "group")
```

I then plotted the data.

```r
#create subsetted plots by race
plot.white<-ggplot()+
  geom_bar(data=Q3.summary[(Q3.summary$X_imprace=="White, Non-Hispanic"),], 
           stat="identity", position="dodge", aes(x=bphigh4, y=freq, fill=group))+
  scale_fill_discrete(name="Emotionally Upset By Treatment", labels=c("No", "Yes"))+
  labs(x="High Blood Pressure", y="Percentage of Respondents", 
       title="White, Non-Hispanic") +
  scale_y_continuous(labels = scales::percent)

plot.black<-ggplot()+
  geom_bar(data=Q3.summary[(Q3.summary$X_imprace=="Black, Non-Hispanic"),], 
           stat="identity", position="dodge", aes(x=bphigh4, y=freq, fill=group))+
  scale_fill_discrete(name="Emotionally Upset By Treatment", labels=c("No", "Yes"))+
  labs(x="High Blood Pressure", y="Percentage of Respondents", 
       title="Black, Non-Hispanic") +
  scale_y_continuous(labels = scales::percent)

plot.asian<-ggplot()+
  geom_bar(data=Q3.summary[(Q3.summary$X_imprace=="Asian, Non-Hispanic"),], 
           stat="identity", position="dodge", aes(x=bphigh4, y=freq, fill=group))+
  scale_fill_discrete(name="Emotionally Upset By Treatment", labels=c("No", "Yes"))+
  labs(x="High Blood Pressure", y="Percentage of Respondents", 
       title="Asian, Non-Hispanic") +
  scale_y_continuous(labels = scales::percent)

plot.native<-ggplot()+
  geom_bar(data=Q3.summary[(Q3.summary$X_imprace=="American Indian/Alaskan Native, Non-Hispanic"),], 
           stat="identity", position="dodge", aes(x=bphigh4, y=freq, fill=group))+
  scale_fill_discrete(name="Emotionally Upset By Treatment", labels=c("No", "Yes"))+
  labs(x="High Blood Pressure", y="Percentage of Respondents", 
       title="American Indian/Alaskan Native, Non-Hispanic") +
  scale_y_continuous(labels = scales::percent)

plot.hispanic<-ggplot()+
  geom_bar(data=Q3.summary[(Q3.summary$X_imprace=="Hispanic"),], 
           stat="identity", position="dodge", aes(x=bphigh4, y=freq, fill=group))+
  scale_fill_discrete(name="Emotionally Upset By Treatment", labels=c("No", "Yes"))+
  labs(x="High Blood Pressure", y="Percentage of Respondents", 
       title="Hispanic") +
  scale_y_continuous(labels = scales::percent)

plot.other<-ggplot()+
  geom_bar(data=Q3.summary[(Q3.summary$X_imprace=="Other race, Non-Hispanic"),], 
           stat="identity", position="dodge", aes(x=bphigh4, y=freq, fill=group))+
  scale_fill_discrete(name="Emotionally Upset By Treatment", labels=c("No", "Yes"))+
  labs(x="High Blood Pressure", y="Percentage of Respondents", 
       title="Other race, Non-Hispanic") +
  scale_y_continuous(labels = scales::percent)

##Create plots
grid.arrange(plot.white, plot.black, plot.asian, plot.native, plot.hispanic, plot.other, nrow=3, ncol=2)
```

![](intro_data_prob_project_files/figure-html/unnamed-chunk-9-1.png)<!-- -->
The results for this question are much less conclusive than the other two. There is no noticeable pattern.  The possible exception is for the "American Indian/Alaskan Native, Non-Hispanic" and "Other race, Non-Hispanic" results.  Both show higher reports of high blood pressure among respondents who have also reported emotional distress resulting from their treatment due to their race.

The problem is that the sample sizes for these two groups are so small (16 and 13, respectively), it is difficult to assert the presence of a significant pattern.

Though further research with more targeted design may provide some interesting results, the results from this study do not suggest any kind of pattern. Certainly nothing as compelling as what was found for questions one and two.
