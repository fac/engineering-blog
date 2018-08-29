---
:title: "Clean house: clear mind. Clean data: clear findings."
:created_at: 2018-07-30 09:00:00.000000000 +00:00
:kind: article
:tags:
- analytics
- data science
- FreeAgent interns
:author: charlottewoolley
:slug: charlottewoolley_data_science_blog_two
---
<style>
body {
text-align: left}
</style>

Soon after settling in at FreeAgent and getting to grips with my role as a data science intern, I got the opportunity to present some of the data that I had been working on at a ‘town hall’, a company-wide weekly meeting where everyone gets together to present their work, share news and pitch ideas. The data I presented was attitudinal survey data from accountancy practices that had contracts with FreeAgent. This might sound fairly simple, but don’t be fooled! I would like to explain the process of what had to happen before I could even think about presenting this information: data cleaning.

## Janitorial justification

In 2016, the International Business Machines (IBM) estimated that the US lost $3 trillion in GDP due to poor quality data and one in three business leaders did not trust the data sources they were using to make decisions<sup>1</sup>. One way that these losses can be minimised with the best possible quality of data preserved is data cleaning.
Sometimes referred to as data cleansing, data scrubbing or data wash, data cleaning is defined as ‘the process of detecting, diagnosing and editing faulty data’<sup>2</sup>. All three of these steps are equally important - I’ll use the analogy of cleaning a kitchen to explain. Not taking the time to detect errors would be like declaring your kitchen clean when you haven’t checked the bins - just because you don’t look for it doesn’t mean it’s not there. Not diagnosing the errors would be like not asking anyone why the bins are full - if you don’t find out what/who is responsible then nothing will change in the future. Not editing the errors would be like checking the bins, seeing they were full but going for a nap instead of emptying them - acknowledging there is a problem but ignoring it!

![data cleaning diagram](/assets/images/2018/04-cw-summer-blog-two/blog_pic1.png){:.aligncenter}

<div style="text-align: center"><i>Mopping up can be a daunting task...</i></div>
&nbsp;

## Considering the contaminants
The methods with which information is collected, recorded, stored and retrieved can all introduce errors into the data, which means that every dataset has its own individual data cleaning challenges. Although some datasets (like kitchens) are easier to clean than others, the vast majority of datasets contain some form of errors (even new kitchens have dust!). Errors may appear in many different forms, caused by many different reasons.

Sometimes data cleaning is not about removing errors but rather making data interpretable. In large datasets, free text boxes are usually ignored because they are notoriously difficult to interpret. However, free text boxes are valuable sources of information and can provide clues about the types of error that may be encountered during the data cleaning process. Even if it is difficult to clean the free text boxes themselves, manual visualisation of their contents can be very useful during the data cleaning process.

Although it is impossible to identify every error or discrepancy that might occur, the following errors are common across many different types of datasets:

1. Duplications: where one row contains identical/similar information to another row.
2. ‘NA’ misclassifications: where empty values are misclassified as known values or vice versa
3. Erroneous answers: where incorrect values are entered, either accidentally or deliberately (a common effect of compulsory questions that cannot be answered)
4. White space and alphabetical case errors: where values appear to mean the same thing but are classified differently due to white space or alphabetical case
5. Spelling mistakes, typing errors or special character errors: where values appear to mean the same thing but are classified differently due to spelling mistakes, typing errors or special characters
6. Inconsistent time formats: where dates/times have been inputted in several different time formats, making it impossible to make time-based calculations
7. Data merging ambiguity: where information that could be recorded in different columns is merged into one column, making the information difficult to interpret
8. Data recording ambiguity: where information is recorded in different ways (e.g. giving a range of values rather than a single value)

Finally, data cleaning should be undertaken carefully so that unanticipated biases are not introduced into the dataset. It is good practice to analyse the data before and after the data cleaning process to see how this affects the results.

![Winston health and safety](/assets/images/2018/04-cw-summer-blog-two/blog_pic2.png){:.aligncenter}

<div style="text-align: center"><i>Winston always takes care when cleaning...</i></div>
&nbsp;

## Next time: why, where, who and what… but how?
We’ve looked at why data cleaning is important (it improves data quality), where the errors exist (most datasets), who needs to consider data cleaning (everyone that performs data analysis) and what errors exist (a huge variety!) - but how do we actually go about cleaning the data? This can be a daunting task if you don’t know any data cleaning tools (just as cleaning a kitchen would be if you didn’t have any materials!) so in my next blog post, I’ll demonstrate some R-based data cleaning techniques for the most common types of error.

## References
1. IBM. 2016. The four V’s of big data. Retrieved from: http://www.ibmbigdatahub.com/infographic/four-vs-big-data. [Accessed 25 July 2018]
2. Van Den Broeck, J., Cunningham, S. A., Eeckels, R., & Herbst, K. (2005). Data cleaning: Detecting, diagnosing, and editing data abnormalities. PLoS Medicine, 2(10), 0966–0970. https://doi.org/10.1371/journal.pmed.002026

&nbsp;
&nbsp;
