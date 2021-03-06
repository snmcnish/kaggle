Introduction
============

This is an anlysis of responses from the 2017 Kaggle ML and Data Science
Survey. All data was collected by [Kaggle](https://www.kaggle.com) and
was available to users from August 7 - August 25, 2017. The survey
received over 16,000 responses.

Specifically, this analysis seeks to provide insight to job seekers
looking for jobs in Data Science. It seeks to answer three main
questions:

1.  What do the respondents look like in terms of demographic
    information?
2.  How do those who are employed differ from job seekers in
    recommendations?
3.  How should job seekers set their expectations in a job?

Installing Required Packages
----------------------------

    library(tidyverse)
    library(scales)
    library(RColorBrewer) 

Loading Data
------------

    # Read-in multiple choice data
    MCData <- read.csv('./input/multipleChoiceResponses.csv', stringsAsFactors = TRUE, header = TRUE)

    # Read-in freeform responses
    FFData <- read.csv('./input/freeformResponses.csv', stringsAsFactors = FALSE, header = TRUE)

    # Read-in the actual questions asked
    schema <- read.csv('./input/schema.csv', stringsAsFactors = FALSE, header = TRUE)

    # Read-in the conversion rate table
    conversionRates <- read.csv('./input/conversionRates.csv', header = TRUE)

Exploration of the Respondents
==============================

Let's get a sense of who the survey respondents are, with specialized
attention in comparing job seekers and those employed.

Employment Status
-----------------

First, what is the employment status of all survey respondents?

    MCData %>% 
      group_by(EmploymentStatus) %>% 
      summarise(count = n()) %>% 
      arrange(desc(count)) %>% 
      ggplot(aes(x=EmploymentStatus, y = count)) +
      geom_bar(aes(fill = EmploymentStatus), stat = "identity",color = "black" ) +
      coord_flip()  + 
      labs(x = "", y="\nNumber of Respondents") +
      ggtitle("Employment status of survey respondents") +
      theme( axis.line = element_line(size=.5, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank(), panel.grid.major.x = element_line(colour="gray90", size=1)
            , panel.grid.minor.x = element_line(colour="gray90", size=.5)) +
      theme(plot.title = element_text(size = 10,  face = "bold"), legend.position = "none", 
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) + scale_fill_brewer( palette = "Set3")

<img src="survey_files/figure-markdown_strict/unnamed-chunk-3-1.png" width="100%" />

An overwhelming majority of respondents are employed (10,897
respondents), but those who are seeking work represent a sizeable sample
of the survey (2,110 respondents).

Age Distribution
----------------

Let's take a look at the age distribution of job seekers vs those who
are employed.

    MCData %>% 
      # Remove any rows where the respondent didn't answer the question
      filter(!Age == "") %>% 
      #get only job seekers and employees
      filter(EmploymentStatus ==
               c("Employed full-time","Not employed, but looking for work")) %>% 
    ggplot( aes(x = Age, group = EmploymentStatus, fill = EmploymentStatus)) + 
      geom_histogram(binwidth = 1, position="identity", alpha = 0.5, color = "black") + 
      xlab("Age (years)") + 
      ylab("Number of Respondents")  + 
      ggtitle("Age distribution of survey respondents") +
      theme( axis.line = element_line(size=.6, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank(), panel.grid.major.y = element_line(colour="gray90", size=1)) +
      theme(plot.title = element_text(size = 10,  face = "bold"), legend.position = "bottom",
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) + scale_fill_brewer( palette = "Set3", name = "Employment Status")

<img src="survey_files/figure-markdown_strict/unnamed-chunk-4-1.png" width="100%" />

Those looking for a job follow roughly the same age distribution as
those who are employeed, skewed just a bit towards the younger side.
This shows that job seekers are represented by those entering the job
market for the first time as well as those in a mid-career job change.

Gender
------

How are the genders represented in the survey respondents?

    MCData %>% 
      # Remove any rows where the respondent didn't answer the question
      filter(!GenderSelect == "") %>% 
      #get only job seekers and employees
      filter(EmploymentStatus ==
               c("Employed full-time","Not employed, but looking for work")) %>% 
    ggplot( aes(x = EmploymentStatus, group = GenderSelect, fill = GenderSelect)) + 
      geom_bar(position = "dodge", color = "black") + 
      labs(x = "", y="\nNumber of Respondents") +
      ggtitle("Gender by employment status") +
      theme( axis.line = element_line(size=.5, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank(), panel.grid.major.y = element_line(colour="gray90", size=1)) +
      theme(plot.title = element_text(size = 10,  face = "bold"), legend.position = "bottom", 
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) + scale_fill_brewer( palette = "Set3", name = "Gender")

<img src="survey_files/figure-markdown_strict/unnamed-chunk-5-1.png" width="100%" />
Clearly, males make up most of both those who are employed and job
seekers.

If we dive deeper into the relationships in the groups between males and
females, the table below shows that out of those employed, employed
females represent 18% of the total number of employeed males.
Comparitively, females represent 28% of the total amount of males
looking for work.

This can be interpretted to mean that more females are newly entering
the data science job market, and/or that females are not being hired at
the same rate as males.

    MCData %>% 
      # Remove any rows where the respondent didn't answer the question
      filter(!GenderSelect == "") %>% 
      #look at only male and females since the other groups are so small
      filter(GenderSelect %in% c("Female","Male")) %>% 
      filter(EmploymentStatus ==
               c("Employed full-time","Not employed, but looking for work")) %>% 
      group_by(EmploymentStatus, GenderSelect) %>% 
      summarise(count = n()) %>% 
      mutate(percentage = round(count/sum(count)*100, digits = 1))%>%
      group_by(EmploymentStatus) %>% 
      summarise(PercFemaleofMale = paste0(round(min(percentage)/max(percentage)*100,digits=2),"%"))

    ## # A tibble: 2 x 2
    ##                     EmploymentStatus PercFemaleofMale
    ##                               <fctr>            <chr>
    ## 1                 Employed full-time           17.79%
    ## 2 Not employed, but looking for work           28.37%

Comparing Responses from those Employeed to Responses from Job Seekers
======================================================================

Now that we have a sense of who represents the employed and job seeker
segments, let's turn to see how their responses for common questions
differ in order to gain information that job seekers can utilize. The
notion here is that those employeed have a better sense of what it takes
to get a job than those who are seeking work.

Finding a Job
-------------

Perhaps the statistic that those employed have proven themselves more
knowedgable about than job seekers is the way to get a job. Let's
compare what resources job seekers are using to find a job compared to
how those who are employeed found a job.

First, what resources are job seekers using to find a job?

    MCData %>% 
      filter(JobSearchResource != "") %>% 
      group_by(JobSearchResource) %>% 
      summarise(count = n()) %>% 
      arrange(desc(count)) %>% 
      ggplot(aes(x=JobSearchResource, y = count)) +
      geom_bar(aes(fill = JobSearchResource), stat = "identity",color = "black" ) +
      coord_flip()+
      labs(x = "", y="\nNumber of Respondents") +
      ggtitle("Methods job seekers are using to find a job") +
      theme( axis.line = element_line(size=.5, colour = "black"),legend.position="none",
            panel.border = element_blank(), panel.background = element_blank(), panel.grid.major.x = element_line(colour="gray90", size=1)) +
      theme(plot.title = element_text(size = 10,  face = "bold"),
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) + scale_fill_brewer( palette = "Set3")

<img src="survey_files/figure-markdown_strict/unnamed-chunk-7-1.png" width="100%" />

Unemployed job seekers are mostly using company websites and job boards
to find a job.

Now let's compare that to how employees found their current job.

    MCData %>% 
      filter(EmployerSearchMethod != "") %>% 
      group_by(EmployerSearchMethod) %>% 
      summarise(count = n()) %>% 
      arrange(desc(count)) %>% 
      ggplot(aes(x=EmployerSearchMethod, y = count)) +
      geom_bar(aes(fill = EmployerSearchMethod), stat = "identity",color = "black" ) +
      coord_flip()+
      labs(x = "", y="\nNumber of Respondents") +
      ggtitle("How employees got their job") +
      theme( axis.line = element_line(size=.5, colour = "black"),legend.position="none",
            panel.border = element_blank(), panel.background = element_blank(), panel.grid.major.x = element_line(colour="gray90", size=1)) +
      theme(plot.title = element_text(size = 10,  face = "bold"),
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) + scale_fill_brewer( palette = "Set3")

<img src="survey_files/figure-markdown_strict/unnamed-chunk-8-1.png" width="100%" />

Most employees found their current job through a personal connection - a
friend, family member, or current employee. This shows high evidence of
the value of networking and using personal connections to find jobs.
Those seeking jobs should build thier personal network and use their
connections. Additionally, job seekers should build their online
portfolio as the second most popular way most employees found their job
was that they were reached out to by someone at the company.

Language Recommendation
-----------------------

What programming languages does a respondent who is employeed recommend
to a new data scientist to learn first compared to the programming
languages someone seeking a job recommends?

    MCData %>% 
      #get only job seekers and employees
      filter(EmploymentStatus ==
               c("Employed full-time","Not employed, but looking for work")) %>% 
      filter(LanguageRecommendationSelect != "") %>% 
      group_by(EmploymentStatus, LanguageRecommendationSelect) %>% 
      summarise(count = n()) %>% 
      arrange(desc(count)) %>% 
      ggplot(aes(x =EmploymentStatus , y= count, fill = LanguageRecommendationSelect)) +
      geom_bar(stat = "identity", position = "fill", width=0.5, color = "black") +
      labs(x = "", y="\nPercent Chosen") +
      ggtitle("Laguange recommendation for a new data scientist to learn first") + 
      scale_y_continuous(labels = scales::percent) +
      theme( axis.line = element_line(size=.5, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank(), panel.grid.major.y = element_line(colour="gray90", size=1)) +
      theme(plot.title = element_text(size = 10,  face = "bold"),
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) + scale_fill_brewer( palette = "Set3" , name = "Skill")

<img src="survey_files/figure-markdown_strict/unnamed-chunk-9-1.png" width="100%" />

Clearly, job seekers and those employed mostly recommend Python as the
programming language to learn first followed by R.

Let's remove those two from analysis to pull out more trends obfiscated
by the distribution.

    MCData %>% 
      filter(LanguageRecommendationSelect != "R" ) %>% 
      filter(LanguageRecommendationSelect != "Python" ) %>% 
      filter(LanguageRecommendationSelect != "" ) %>% 
      filter(EmploymentStatus ==
               c("Employed full-time","Not employed, but looking for work")) %>% 
      group_by(EmploymentStatus, LanguageRecommendationSelect) %>% 
      summarise(count = n()) %>% 
      arrange(desc(count)) %>% 
      ggplot(aes(x =EmploymentStatus , y= count, fill = LanguageRecommendationSelect)) +
      geom_bar(stat = "identity", position = "fill", width=0.5, color =1) +
      labs(x = "", y="\nPercent Chosen") +
      ggtitle("Laguange recommendation for a new data scientist to learn first\n(Python and R removed)") +
      scale_y_continuous(labels = scales::percent) +
      theme( axis.line = element_line(size=.5, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank(), panel.grid.major.y = element_line(colour="gray90", size=1)) +
      theme(plot.title = element_text(size = 10,  face = "bold"),
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) + scale_fill_brewer( palette = "Set3" , name = "Skill")

<img src="survey_files/figure-markdown_strict/unnamed-chunk-10-1.png" width="100%" />

Those employed full-time recommend SQL at a higher rate than those
looking for work. Perhaps those looking for a job should consider
learning SQL to add to their skillset. Additionally, job seekers
reccommend learning C/C++/C\# and Matlab at a much higher rate than
those employed. Perhaps job seekers should instead focus their time on
learning a language prefered by those employed instead of one of those
languages.

Worthwhile Skillsets
--------------------

Let's compare the type of skills job seekers believe are important in
getting a data science job vs the data science/analytics tools,
technologies, and languages that those employeed actually use at work.
These were two seperate questions posed in the survey with different
selection options, but meaningful insights can still be extracted.

First, people who are learning data science were asked to select the
importance of each of the below skills or certifications in getting a
data science job.

    # Create list of skills
    skills <- schema %>% 
      # From the list of questions, keep only questions that contain the below phrase
      filter(grepl("How important do you think the below skills or certifications are in getting a data science job?", Question, fixed = TRUE)) %>%
      # Remove any Columns that contain "FreeForm"
      filter(!grepl("FreeForm", Column, fixed = TRUE)) %>% 
      # Split the question text at the hyphen
      mutate(response = strsplit(as.character(Question), " - ")) %>% 
      # Separate the responses onto separate rows
      unnest(response) %>% 
      # Remove rows that contain the first phrase
      filter(!grepl("How important do you think the below skills or certifications are in getting a data science job", response, fixed = TRUE)) %>% 
      # Keep only the question number and the associated barrier
      select(-2)

    # Limit the full dataset to the columns that answer these questions
    skillsNames <- MCData %>% 
      # Keep only columns that start with "JobSkillImportance" and don't contain "FreeForm"
      select(starts_with("JobSkillImportance"), -contains("FreeForm")) %>% 
      select(-contains("Other")) %>% 
      # Restructure Data
      gather(key = response, value = frequency) %>% 
      # Remove any blank entries
      filter(!frequency == "") 

    # Combine the list of possible challenges with the limited dataset
    skillsNamesChar <- left_join(skillsNames, skills, by = c("response" = "Column")) %>% 
      # Group the responses by the question Number and the frequency with which each challenged is dealt with
      group_by(response.y, frequency) %>%
      # Count the number of entries within each group
      summarise(count = n()) %>% 
      # Re-order the factors
      mutate(frequency = factor(frequency, levels = c("Necessary", "Nice to have", "Unnecessary"), ordered = TRUE)) 

    # Plot results
    ggplot(skillsNamesChar, aes(x = response.y, y = count, fill = frequency)) + 
      geom_bar(stat = "identity",colour = "black") + 
      coord_flip()+
      labs(x = "", y="\nNumber Respondents") +
      ggtitle("Importance of skills in getting a data science job") +
      theme( axis.line = element_line(size=.5, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank(), panel.grid.major.x = element_line(colour="gray90", size=1)) +
      theme(plot.title = element_text(size = 10,  face = "bold"),
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) + scale_fill_brewer( palette = "Set3" , name = "Frequency")

<img src="survey_files/figure-markdown_strict/unnamed-chunk-11-1.png" width="100%" />

Job seekers generally believe to not waste your time with Enterprise
tools such as SAS or an online ceritifaction/MOOC. They see the
importance of becoming fluent in Pyhton, SQL, and R, as well as learning
advanced statistics.

Turning to what tools are actually used on the job, employed survey
respondents were asked how often the following data science/analytics
tools, technologies, and languages did they use in the past year. They
were given an option of 54 choices to rank, but we will only look at the
9 tools chosen most frequently for clarity.

    # Create list of tools
    tools <- schema %>% 
      # From the list of questions, keep only questions that contain the below phrase
      filter(grepl("At work, how often did you use the following data science/analytics tools, technologies, and languages this past year?", Question, fixed = TRUE)) %>%
      # Remove any Columns that contain "FreeForm"
      filter(!grepl("FreeForm", Column, fixed = TRUE)) %>% 
      # Split the question text at the hyphen
      mutate(response = strsplit(as.character(Question), " - ")) %>% 
      # Separate the responses onto separate rows
      unnest(response) %>% 
      # Remove rows that contain the first phrase
      filter(!grepl("At work, how often did you use the following data science/analytics tools, technologies, and languages this past year", response, fixed = TRUE)) %>% 
      # Keep only the question number and the associated barrier
      select(-2)

    # Limit the full dataset to the columns that answer these questions
    toolsNames <- MCData %>% 
      # Keep only columns that start with "WorkToolsFrequency" and don't contain "FreeForm"
      select(starts_with("WorkToolsFrequency"), -contains("FreeForm")) %>% 
      select(-contains("Other")) %>% 
      # Restructure Data
      gather(key = response, value = frequency) %>% 
      # Remove any blank entries
      filter(!frequency == "") 

    #keep only top by frequency
    toolsNames <- toolsNames[ toolsNames$response %in%  names(table(toolsNames$response))[table(toolsNames$response) >1500] , ]

    # Combine the list of possible challenges with the limited dataset
    toolsNamesChar <- left_join(toolsNames, tools, by = c("response" = "Column")) %>% 
      # Group the responses by the question Number and the frequency with which each challenged is dealt with
      group_by(response.y, frequency) %>%
      # Count the number of entries within each group
      summarise(count = n()) %>% 
      # Re-order the factors
      mutate(frequency2 = factor(frequency, levels = c("Most of the time", "Often", "Sometimes", "Rarely"), ordered = TRUE)) 

    ggplot(toolsNamesChar, aes(x = response.y, y = count, fill = frequency)) + 
      geom_bar(stat = "identity", colour="black") + 
      coord_flip()+
      labs(x = "", y="\nNumber Respondents") +
      ggtitle("Tools, technologies, and languages used in the past year") +
      theme( axis.line = element_line(size=.5, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank(), panel.grid.major.x = element_line(colour="gray90", size=1)) +
      theme(plot.title = element_text(size = 10,  face = "bold"),
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) + scale_fill_brewer( palette = "Set3" , name = "Frequency")

<img src="survey_files/figure-markdown_strict/unnamed-chunk-12-1.png" width="100%" />

In practice, those employeed use Python, SQL, and R regularly--which
aligns to the skills job seekers believe are important in getting a job.
Jupyter notebooks are also a tool many data scientists use in practice.
Job seekers should consider learning how to use Jupyter notebooks to add
to their resume.

Job Expectations
================

We now turn to the current job landscape so that job seekers can manage
thier expectations in what to look for.

Job Satisfaction
----------------

What is the typical job satisfaction of employees surveyed and how does
it differ across job titles?

To look into this, I normallized the data by job title so that the
number of respondents of each title is taken into account, making
cross-comparison achiveable.

    MCData %>% 
      filter(JobSatisfaction != "") %>% 
      filter(JobSatisfaction != "NA") %>% 
      filter(CurrentJobTitleSelect != "") %>%
      filter(CurrentJobTitleSelect != "I prefer not to share") %>% 
      #order job satisfaction
      mutate(JobSatisfaction = factor(JobSatisfaction, levels = c("1 - Highly Dissatisfied", "2", "3", "4", "5","6","7","8","9","10 - Highly Satisfied"),    ordered = TRUE)) %>% 
      group_by(CurrentJobTitleSelect, JobSatisfaction) %>% 
      summarise(count = n()) %>% 
      #normalize data
      mutate(norm = count/sum(count)) %>% 
      ggplot(aes(x = JobSatisfaction, y= CurrentJobTitleSelect)) +
      geom_tile(aes(fill = norm), color = "black", size = .5)+
      scale_fill_gradient(low = "white", high="lightskyblue3" , 
                          name = "Count of Respondents\n(Normalized)") +
      labs(x="\nSatisfaction Level", y="Job Title\n") +
      ggtitle("Comparing satisfaction across job titles") +
      theme( axis.line = element_line(size=.5, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank()) +
      theme(plot.title = element_text(size = 10,  face = "bold"),
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8, angle = 75, hjust = 1),
            axis.text.y=element_text(colour="black", size = 8)) 

<img src="survey_files/figure-markdown_strict/unnamed-chunk-13-1.png" width="100%" />

Employees generally rate their job satisfaction in the 7-8 range across
all job titles. Those with the "Programmer" job title are more likely to
select a lower job satisfaction compared to other job titles.

Salary
------

Salary is a very important component to factor into job expectations.
Note that for this part of the analysis, only ~30% of survey respondents
answered this question. This isn't surprising since it may be considered
a private question. We can still analyze the responses from those who
elected to answer. For this analysis, we convert all currencies to US
dollars.

How does salary vary across job titles?

    reportedSalary <- MCData %>% 
      # Keep only the columns with the reported salary and currency
      select(c("CompensationAmount", "CompensationCurrency", "CurrentJobTitleSelect")) %>% 
      # Remove any blank responses
      filter(!CompensationCurrency == "") %>% 
      filter(!CompensationAmount == "") %>% 
      filter(!CurrentJobTitleSelect == "")

    # Combine the reported salary data with the conversion rate
    salaryUSD <- left_join(reportedSalary, conversionRates, by = c("CompensationCurrency" = "originCountry")) %>%
      # Convert the reported salary to a character string
      mutate(CompensationAmount = as.character(CompensationAmount),
             # Remove any commas from the salary entry and convert it to a number
             originalSalary = as.numeric(gsub(",", "", CompensationAmount)),
             # Convert the exchange rate to a number
             exchangeRate = as.numeric(as.character(exchangeRate)),
        # Calculate the salary in USD
        usSalary = originalSalary * exchangeRate,
        # Convert the calculated salary to a number and round to 2 decimal places
        usSalary = as.numeric(format(round(usSalary, 2), nsmall = 2, scientific = FALSE))) %>% 
      arrange(desc(usSalary))

    # For graphing purposes, remove responses over $400,000 (since there are very few above that limit) 
    salaryUSDPlot <- salaryUSD %>% 
      # Remove any salaries (in USD) that are above $400,000 or less than or equal to 0
      filter(usSalary < 400000,
             usSalary > 0)

    # Reorder the data by median of salary by job title for boxplot
    salaryUSDPlot$CurrentJobTitleSelect = reorder(salaryUSDPlot$CurrentJobTitleSelect, salaryUSDPlot$usSalary, median)

    #interpolate more colors onto the Set3 pallette
    cols <- colorRampPalette(brewer.pal(12, "Set3"))
    myPal <- cols(length(unique(salaryUSDPlot$CurrentJobTitleSelect)))

    ggplot(salaryUSDPlot, aes(CurrentJobTitleSelect, usSalary)) + 
      geom_boxplot(aes(fill = CurrentJobTitleSelect), color= "black") + coord_flip() +
      ylab("Responses of a given Salary (in USD)") +
      theme(legend.position="none") +
      labs(x="", y="Salary (USD)\n") +
      ggtitle("Comparing salary across job titles") +
      theme( axis.line = element_line(size=.5, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank()) +
      theme(plot.title = element_text(size = 10,  face = "bold"),
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) +  scale_fill_manual(values = myPal)+ 
      scale_y_continuous(labels = scales::comma)

<img src="survey_files/figure-markdown_strict/unnamed-chunk-14-1.png" width="100%" />

The highest median reported salary comes from those with the job title
"Operations Research Practitioner" followed by "Other" and "Data
Scientist". However, it's important to keep in mind here how many
responents answered in each group. The more respondents, the more
confident we can be that the reported salary is reflective of the true
salary of the job title in the job market. The table below shows how
many respondents answered the salary question in eah job title group to
complement the above boxplot.

    salaryUSDPlot %>% 
      group_by(CurrentJobTitleSelect) %>% 
      summarise(count = n()) 

    ## # A tibble: 16 x 2
    ##                   CurrentJobTitleSelect count
    ##                                  <fctr> <int>
    ##  1                   Computer Scientist   105
    ##  2                           Programmer    88
    ##  3                           Researcher   228
    ##  4                           Data Miner    39
    ##  5                         Data Analyst   477
    ##  6                     Business Analyst   226
    ##  7            Machine Learning Engineer   276
    ##  8                         Statistician   119
    ##  9                 Scientist/Researcher   407
    ## 10                DBA/Database Engineer    66
    ## 11                             Engineer   145
    ## 12 Software Developer/Software Engineer   444
    ## 13                   Predictive Modeler    85
    ## 14                       Data Scientist  1259
    ## 15                                Other   328
    ## 16     Operations Research Practitioner    19

Let's perform the same excercise comparing salary, this time by
educational attainment.

    reportedSalary <- MCData %>% 
      # Keep only the columns with the reported salary and currency
      select(c("CompensationAmount", "CompensationCurrency", "FormalEducation")) %>% 
      # Remove any blank responses
      filter(!CompensationCurrency == "") %>% 
      filter(!CompensationAmount == "") %>% 
      filter(!FormalEducation == "")%>% 
      filter(!FormalEducation == "I prefer not to answer")

    # Combine the reported salary data with the conversion rate
    salaryUSD <- left_join(reportedSalary, conversionRates, by = c("CompensationCurrency" = "originCountry")) %>%
      # Convert the reported salary to a character string
      mutate(CompensationAmount = as.character(CompensationAmount),
             # Remove any commas from the salary entry and convert it to a number
             originalSalary = as.numeric(gsub(",", "", CompensationAmount)),
             # Convert the exchange rate to a number
             exchangeRate = as.numeric(as.character(exchangeRate)),
        # Calculate the salary in USD
        usSalary = originalSalary * exchangeRate,
        # Convert the calculated salary to a number and round to 2 decimal places
        usSalary = as.numeric(format(round(usSalary, 2), nsmall = 2, scientific = FALSE))) %>% 
      arrange(desc(usSalary))

    # For graphing purposes, remove responses over $400,000 (since there are very few above that limit) 
    salaryUSDPlot <- salaryUSD %>% 
      # Remove any salaries (in USD) that are above $400,000 or less than or equal to 0
      filter(usSalary < 400000,
             usSalary > 0)

    # Reorder the data by median of salary by education for boxplot
    salaryUSDPlot$FormalEducation = reorder(salaryUSDPlot$FormalEducation, salaryUSDPlot$usSalary, median)

    #interpolate more colors onto the Set3 pallette
    cols <- colorRampPalette(brewer.pal(12, "Set3"))
    myPal <- cols(length(unique(salaryUSDPlot$FormalEducation)))

    ggplot(salaryUSDPlot, aes(FormalEducation, usSalary)) + 
      geom_boxplot(aes(fill = FormalEducation), color= "black") + coord_flip() +
      ylab("Responses of a given Salary (in USD)") +
      theme(legend.position="none") +
      labs(x="", y="Salary (USD)\n") +
      ggtitle("Comparing salary across education level") +
      theme( axis.line = element_line(size=.5, colour = "black"),
            panel.border = element_blank(), panel.background = element_blank()) +
      theme(plot.title = element_text(size = 10,  face = "bold"),
            text=element_text(size = 9),
            axis.text.x=element_text(colour="black", size = 8),
            axis.text.y=element_text(colour="black", size = 8)) +  scale_fill_manual(values = myPal)+ 
      scale_y_continuous(labels = scales::comma)

<img src="survey_files/figure-markdown_strict/unnamed-chunk-16-1.png" width="100%" />

The highest median salary comes from those who report to have a Doctoral
degree followed by a Master's degree. Interestingly enough, the lowest
median salary comes from those who hold a Bachelor's degree. However, I
believe this is attributed to an issue with sample size. As you can see
in the table below, there were many more respondents with a Doctoral,
Master's, or Bachelor's degree than the other three categories.

    salaryUSDPlot %>% 
      group_by(FormalEducation) %>% 
      summarise(count = n()) 

    ## # A tibble: 6 x 2
    ##                                                     FormalEducation count
    ##                                                              <fctr> <int>
    ## 1                                                 Bachelor's degree  1096
    ## 2                                               Professional degree   129
    ## 3 Some college/university study without earning a bachelor's degree   110
    ## 4          I did not complete any formal education past high school    26
    ## 5                                                   Master's degree  1978
    ## 6                                                   Doctoral degree   966

Wrap-Up
=======

There is a lot of insight job seekers can gain from this survey data to
enhance their job search. Don't forget to upvote if you found this
analysis useful!

Also special thanks to Kaggle user @amberthomas for her kernal "Kaggle
2017 Survey Results" in which I borrowed some code from.
