---
:title: "Dealing with dirty data: useful functions for data cleaning in R"
:created_at: 2018-08-15 09:00:00.000000000 +00:00
:kind: article
:tags:
- analytics
- data science
- FreeAgent interns
:author: charlottewoolley
:slug: charlottewoolley_data_science_blog_three
---
<style>
body {
text-align: left}
</style>


In this blog post, I’ll explain how to use some simple R-based data cleaning solutions (mostly in the ‘tidyverse’ package<sup>1</sup>) to address the most common dataset errors with the help of my favourite analogy: the untidy kitchen!
&nbsp;

NB: There are a plethora of valuable data cleaning tools in other software and even within R there are many different tools available. While the approach that I describe here is not necessarily ‘the best’ way of doing things, I’ve found that it’s what works for me.

## Setting the scene
The kitchen areas at FreeAgent are usually very clean but in this scenario, let’s imagine that we have been very messy recently! We haven't been putting our mugs in the dishwasher or generally keeping the kitchen clean so we’ve designed a daily rota to make sure that one person is responsible for giving the kitchen a five-minute blitz after lunchtime.

The data science team are interested in analysing the data to find out why people became so messy in the first place, so we asked everyone to keep details of their cleaning duty in a shared spreadsheet. We asked them:

<ul type="square">
    <li>their name</li>
    <li>the date when they last cleaned the kitchen</li>
    <li>whether the dishwasher was full at the time</li>
    <li>the number of mugs they found on the side</li>
    <li>whether they wiped the sides or sink</li>
</ul>


We made the ‘number of mugs’ question compulsory because we thought people might be lazy and not want to count them. We also allowed people to write any notes in a separate column. We were excited to see the data after the first two weeks of the rota system but we found that it was very difficult to interpret. Although we had greatly reduced the amount of mess in the kitchen, we now had a new chore: data cleaning!

![FreeAgent mugs](/assets/images/2018/05-cw-summer-blog-three/blog_pic1.jpg){:.aligncenter}

<div style="text-align: center"><i>Is the cleaner getting mugged off?</i></div>
&nbsp;

## Tidying tools

If you would like to follow along with this tutorial, the data we collected can be downloaded [here.](/assets/data/2018/05-cw-summer-blog-three/cleaning_data.csv)
Let’s take a look at the data we collected:

<pre>
# Import the libraries needed

library(tidyverse)
library(lubridate)

# Read in the csv with the "read.csv" function

dat <- read.csv("cleaning_data.csv")
</pre>

![Uncleaned data](/assets/images/2018/05-cw-summer-blog-three/blog_table1.png){:.aligncenter}

<div style="text-align: center"><i>The uncleaned dataset</i></div>
&nbsp;

We can see that many of the common errors I identified in my most recent blog post are present in this dataset:

*Removing NA misclassifications and white space*

First of all, it looks like some ‘NAs’ (blank spaces, ‘N/A’, ‘NA’) were not recognised when the data was read in. Also, we suspect there might be some leading and trailing white space because there were free text boxes in the survey.
<pre>
# Read in the csv with the alternative "write_csv" function and the
# following options to remove NA misclassifications and white space

dat <- read_csv("cleaning_data.csv", na = c("", "N/A", "NA"), trim_ws = TRUE)
</pre>

*Removing duplicate data entries*

We can also see the final row is duplicated, where “Davie” accidentally copied and pasted a row!
<pre>
# The %>% function pipes (transfers along) the data to the next
# function
# The "distinct" function removes duplicated rows

dat <- dat %>%
      distinct()

# NOTE: Sometimes duplications might not be fixable with "distinct".
# Imagine if Davie had copied the row but then changed the value
# in the "no_of_mugs" column to 5. This would mean that the data
# was similar rather than identical and the distinct function
# would no longer be effective
</pre>

*Re-classifying dates in different time formats*

The ‘date_cleaned’ column was recorded in a free text box so is classed as a character string rather than a date and has been inputted in lots of different time formats, making date-based calculations impossible.
<pre>
# The "mutate" function is used to make new columns
# The "parse_date_time" function converts dates written in different
# formats as strings into a UTC format. The "dmy" option tells R
# that the day comes first, month second and year last

dat <- dat %>%
      mutate(date_cleaned = parse_date_time(date_cleaned, "dmy"))  
</pre>

*Separating information that has been merged together in one column*

The ‘sides_and_sink’ column is difficult to interpret because it contains information about whether both the sides and the sink were cleaned, which would be easier to analyse if it was in two separate columns.
<pre>
# The "case_when" function allows values to be chosen based
# on conditional boolean arguements (if something is true/false:
# do something, if not: do something else)
# The "str_detect" function allows chosen words to be detected
# within a string
# The "is.na" function identifies if a value is NA

dat <- dat %>%
      mutate(clean_sides = case_when(str_detect(sides_and_sink, c("side|both")) == TRUE ~ TRUE,
              is.na(sides_and_sink) == TRUE ~ NA,
              TRUE ~ FALSE),
             clean_sink = case_when(str_detect(sides_and_sink, c("sink|both")) == TRUE ~ TRUE,
              is.na(sides_and_sink) == TRUE ~ NA,
              TRUE ~ FALSE))

# NOTE: Take care when using "str_detect" because of language
# misinterpretation. If someone had written "Cleaned the sides
# but not the sink", this would have lead to a positive value
# for cleaning the sink although it should have been negative.
# Similarly if we had searched for "sides" rather than "side",
# then we wouldn't have detected all of the instances of when
# the sink had been cleaned. There are other tools to deal with
# truncated words which are not covered in this tutorial.
</pre>

*Changing alphabetic case and removing special characters*

The ‘dishwasher_full’ column is difficult to analyse because the character cases are inconsistent and the column contains a special character (‘!’).
<pre>
# The "toupper" function converts a string to upper case
# The "str_replace_all" function with the option "[[:punct:]]"
# removes special characters including "!"

dat <- dat %>%
      mutate(dishwasher_full = str_replace_all(toupper(dishwasher_full), "[[:punct:]]", ""))
</pre>

*Re-coding erroneous answers and inconsistent data recording*

In the ‘notes’ column, we can see that “Davida” records that she filled in the ‘no_of_mugs’ column with ‘0’ because the question was compulsory but the value should be ‘NA’ because everyone was at a company conference that day. We can also see that the ‘no_of_mugs’ column contains text in brackets, which we need to remove so we can analyse the numbers. Additionally, some of the numbers given are a range of numbers instead of an exact number. We ideally want the mean of the range of numbers to make analysis easier.
<pre>
# The function "gsub" with the regular expression ""&bsol;&bsol;(.&ast;","""
# converts values to character strings and removes all of
# the string after the first bracket
# The function "as.Date" converts a character string to a
# date (so it can be recognised)
# The "separate" function with the "-" option splits a column
# into two separate columns, based on the character "-"

dat <- dat %>%
      mutate(no_of_mugs = case_when(date_cleaned == as.Date("2018-07-23") ~ NA_character_,
              TRUE ~ gsub("&bsol;&bsol;(.&ast;","",no_of_mugs)))  %>%
      separate(no_of_mugs, c("lower", "upper"), "-") %>%
      mutate(no_of_mugs = case_when(!is.na(upper) == TRUE ~ (as.numeric(lower)+as.numeric(upper))/2,
              TRUE ~ as.numeric(lower)))

# NOTE: "NA_character_" is given as the value rather than "NA"
# to ensure that it is accepted by the function (the class of "NA"
# needs to be the same as the rest of the data in that column)
</pre>

*Removing and re-ordering columns*

The final step is to remove and re-order any columns that we generated or rearranged during the cleaning process or that we no longer need.
<pre>
# The "select" function chooses specific columns and put them
# in a given order

dat <- dat %>%
      select(date_cleaned, name, clean_sides, clean_sink, dishwasher_full, no_of_mugs, notes)
</pre>

![Cleaned data](/assets/images/2018/05-cw-summer-blog-three/blog_table2.png){:.aligncenter}

<div style="text-align: center"><i>The cleaned dataset</i></div>
&nbsp;

## Purifying the preparations
So there you have it: a few useful R-based data cleaning techniques that can help you deal with dirty data after it’s been recorded. But what if we could actually reduce the amount of cleaning we had to do in the first place? In my next post I’ll look at how to reduce common errors and bias by improving survey design.

## References
1. Wickham. H. 2017. tidyverse: Easily Install and Load the "Tidyverse". R package version 1.2.1. Available from: https://CRAN.R-project.org/package=tidyverse

&nbsp;
&nbsp;
