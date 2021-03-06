COVID-19 Team Beta Group Project
================
Shree, Andrew, Jack, Oscar, Kelly
2020-10-18

-   [Introduction](#introduction)
-   [Exploratory Data Analysis](#exploratory-data-analysis)
-   [Final Graphs](#final-graphs)
-   [Conclusion](#conclusion)
-   [Sources](#sources)

## Introduction

The Covid-19 pandemic was undoubtedly one of the most devastating and
destructive events any of us have witnessed at a global scale. However,
as with any other major crisis, the pandemic didn’t impact every
demographic the same. This project attempts to use exploratory data
analysis to answer the question, “how did the pandemic and its economic
consequences impact men and women differently in the United States?”.

According to a [policy brief published by the
UN](https://www.unwomen.org/-/media/headquarters/attachments/sections/library/publications/2020/policy-brief-the-impact-of-covid-19-on-women-en.pdf?la=en&vs=1406)
in April of 2020, women and girls experienced more negative economic
consequences, poorer health services (due to the reallocation of medical
resources and priorities), and an increase in gender-based violence
during the Covid-19 pandemic. An [article published by the McKinsey
company](https://www.mckinsey.com/featured-insights/future-of-work/covid-19-and-gender-equality-countering-the-regressive-effects)
stated similar statistics, citing that the majority of job losses were
experienced by women because of existing gender inequalities.

Both of these sources looked at how the pandemic impacted people
globally. Knowing what happened at a global scale, we were curious about
how these trends in gender disparities existed in the United States. Our
project uses datasets published by the CDC and Bureau of Labor
Statistics to examine how the pandemic impacted men and women
differently in the US.

## Exploratory Data Analysis

First, we load in the data we’ll be using.

``` r
cases <- read_csv("./data/Provisional_COVID-19_Deaths_by_Sex_and_Age.csv")
```

    ## Rows: 68850 Columns: 16

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (8): Data As Of, Start Date, End Date, Group, State, Sex, Age Group, Foo...
    ## dbl (8): Year, Month, COVID-19 Deaths, Total Deaths, Pneumonia Deaths, Pneum...

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
df_unemploy <- read_csv("data/gender_and_unemployment.csv")
```

    ## Rows: 56 Columns: 3

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Date, Gender
    ## dbl (1): UnemploymentRate

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
df_industry <- read_csv("data/DS_Industry_Top_Level_Percent.csv")
```

    ## Rows: 13 Columns: 3

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Industry
    ## dbl (1): Percent

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#DF_num_employed.csv was a file we generated from the code in BLS_data_manip.Rmd
df_num_employed <- read_csv("data/DF_num_employed.csv")
```

    ## New names:
    ## * `` -> ...1

    ## Rows: 7353 Columns: 6

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (2): Industry, Group
    ## dbl  (3): ...1, Count, Percent_rate_of_change
    ## date (1): Date

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Using a dataset from the CDC, we examine the number of Covid deaths in
the US, separated by gender

``` r
male_data <- cases %>% 
  filter(Sex == "Male", State == "United States", Group == "By Month") %>% 
  mutate(`Start Date` = as.Date(`Start Date`)) %>% 
  mutate(`End Date` = as.Date(`End Date`, format = "%m/%d/%y"))

female_data <- cases %>% 
  filter(Sex == "Female", State == "United States", Group == "By Month") %>% 
  mutate(`Start Date` = as.Date(`Start Date`)) %>% 
  mutate(`End Date` = as.Date(`End Date`, format = "%m/%d/%y"))
```

``` r
ggplot() +
  geom_smooth(data = male_data, aes(x = `End Date`, y = `COVID-19 Deaths`, color = Sex), se = FALSE) + 
  # geom_point(data = male_data, aes(x = `End Date`, y = `COVID-19 Deaths`), color = "blue") + 
  geom_smooth(data = female_data, aes(x = `End Date`, y = `COVID-19 Deaths`, color = Sex), se = FALSE) + 
  # geom_point(data = female_data, aes(x = `End Date`, y = `COVID-19 Deaths`), color = "red") + 
  ggtitle("Covid Deaths Over Time")
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

![](Project1_files/figure-gfm/unnamed-chunk-3-1.png)<!-- --> *Figure 1*

**Observations**: From the graph above, we can observe the deaths over
time for men and women. Both men and women follow similar trends in
number of deaths, with men having one thousand more deaths per day. From
April to June, there seems to be a steady decrease in the number of
deaths for both men and women. However, there is a drastic increase in
the number of deaths for both men and women from the pandemic from
October onwards. This phenomenon may be caused by the onset of winter
months, where cases frequently increase.

The second figure we wanted to look at was Unemployment rates in the US
separated by gender.

``` r
df_unemploy %>%
  ggplot() + 
  geom_line(aes(Date, UnemploymentRate, group = Gender, color = Gender)) + 
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1)) + 
  labs(
    y = "Unemployment Rate by Percent", 
    title = "Unemployment rates in the US during the Covid-19 pandemic"
  )
```

![](Project1_files/figure-gfm/unnamed-chunk-4-1.png)<!-- --> *Figure 2*

**Observations**: Unemployment rates were roughly the same for men and
women before the covid-19 pandemic, around 3%. During the pandemic, both
men and women experienced a drastic increase in unemployment rates,
however unemployment rates for women were about 3% higher than for men.
Around October of 2020, after unemployment rates decreased for both men
and women, trends returned to patterns we noticed before the pandemic,
where unemployment rates for both men and women are about the same.

In order to better understand why unemployment rates differed so
drastically, we determine which industries had predominantly female
workers.

``` r
OverallPercent <-
  df_industry %>%
  ggplot(aes(Industry, Percent, fill = Percent >= 50)) + 
  geom_col() +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.25, hjust=1)) +
  geom_hline(yintercept = 50, linetype = "dashed") 

OverallPercent + ggtitle("Percent of Women Employed in Industry") +
  xlab("Industry") + ylab("Percent")
```

![](Project1_files/figure-gfm/unnamed-chunk-5-1.png)<!-- --> *Figure 3*

**Observations**: There are only a few industries where women make up
50% or more of the people employed. The four industries that were
predominantly female were Education and health services, Leisure and
Hospitality, Financial Activities, and Other services. On the other
hand, the Construction and Mining industries had the smallest proportion
of women.

Now that we know which industries are predominantly female or male, we
look at the percent rate of change of employment for those industries.

``` r
df_num_employed %>%
  filter(
    !(Industry %in% c("Nondurable goods", "Total nonfarm", "Service-providing", "Government", "Goods-producing")),
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) +
  facet_wrap(vars(Industry)) +
  scale_color_manual(values = c("blue", "red"))
```

![](Project1_files/figure-gfm/unnamed-chunk-6-1.png)<!-- --> *Figure 4*

**Observations**: When looking at the individual industry graphs, we can
see that very few industries have a negative spike where women do not
lose jobs faster than everyone else. In several industries such as
Information and Financial Activities the percent rate of change is
fairly even. In the Services industries, Retail, and Education, the
spike for women is much worse than that of their other coworkers. This
visualization also gives us a sense of which industries were most
impacted by the Covid-19 pandemic as far as employment goes.

## Final Graphs

``` r
theme_common <- function() {
  theme_minimal() %+replace%
  theme(
    axis.text.x = element_text(size = 12),
    axis.text.y = element_text(size = 12),
    axis.title.x = element_text(margin = margin(4, 4, 4, 4), size = 16),
    axis.title.y = element_text(margin = margin(4, 4, 4, 4), size = 16, angle = 90),

    legend.title = element_text(size = 16),
    legend.text = element_text(size = 12),

    strip.text.x = element_text(size = 12),
    strip.text.y = element_text(size = 12),

    panel.grid.major = element_line(color = "grey90"),
    panel.grid.minor = element_line(color = "grey90"),

    aspect.ratio = 4 / 4,

    plot.margin = unit(c(t = +0, b = +0, r = +0, l = +0), "cm"),
    plot.title = element_text(size = 18),
    plot.title.position = "plot",
    plot.subtitle = element_text(size = 16),
    plot.caption = element_text(size = 12)
  )
}
```

``` r
df_num_employed %>%
  filter(
    Industry == "Other services",
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  scale_color_manual(labels = c("Other", "Women"), values = c("blue", "red")) + 
  theme_common() + 
  labs(
    y = "Percent rate of change of employed",
    x = "Date",
    title = "United States Services Industry",
    color = "Gender"

  )
```

![](Project1_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
df_num_employed%>%
  filter(
    Industry == "Education and health services",
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  scale_color_manual(labels = c("Other", "Women"), values = c("blue", "red")) + 
  theme_common() + 
  labs(
    y = "Percent rate of change of employed",
    x = "Date",
    title = "Education and Health Services Industry in the United States",
    color = "Gender"

  )
```

![](Project1_files/figure-gfm/unnamed-chunk-8-2.png)<!-- -->

``` r
df_num_employed %>%
  filter(
    Industry == "Total nonfarm",
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  scale_color_manual(labels = c("Other", "Women"), values = c("blue", "red")) +
  theme_common() +
  labs(
    y = "Percent rate of change of employed",
    x = "Date",
    title = "Overall Employment in the United States",
    color = "Gender"
  )
```

![](Project1_files/figure-gfm/unnamed-chunk-8-3.png)<!-- -->

**Observations** In the two industries that were most dominated by
women, “Other services” and “Education and health services”, women lost
their jobs at a higher rate than men even when accounting for gender
imbalance in the industry. Looking at the min and max peaks of these
graphs, we can deduce that after the peak of the pandemic when job
losses were greatest, increase in employment did not completely offset
the number of people who lost their jobs. We know this is true because
looking at figure 2, unemployment rates are still higher than
pre-pandemic levels.

## Conclusion

The figure 1 we examined answers how the pandemic impacted men and women
differently in terms of Covid deaths. When looking at the number of
Covid death in the US, we see that more men died of Covid than women.
After seeing this data we pivoted our question to focus more on the
economic impacts of Covid on different genders rather than just the
direct physical impacts of the pandemic. Our second EDA graph revealed
that while unemployment rates spiked for both men and women in the US,
women experienced a noticeably more drastic increase in unemployment
rates than men (figure 2).

To begin to understand why that might be, we looked at each industry
that was categorized by the Bureau of Labor Statistics, and determined
which ones were predominantly women (figure 3). Next, we looked at the
percent change in employment rates for women and men in each of those
industries (figure 4).

For our final graphs, we examined the “Education and health services”
industry and “Other services” industry because they were the two
industries with the highest proportion of women. In both of these
industries, women were let go at a higher percent rate of change than
men. These two industries also experienced more job losses than most
other industries. In other words, not only did predominantly female
industries experience more job losses than other predominantly male
industries, but within those female dominated industries, women were let
go at a higher rate than men.

Unsurprisingly, our findings corroborate those that were stated in the
UN report and McKinsey article. It’s a commonly recognized reality that
events like a pandemic serve to exacerbate existing inequalities,
whether that’s in regards to race, gender, or wealth. In going through
these processes of retrieving raw data, shaping it into usable
information, and visualizing it to examine patterns and outliers, we
hope to gain a better understanding of how data can be used to educate
us about ways in which our systems can be improved.

## Sources

“CDC Covid Data Tracker.” Centers for Disease Control and Prevention,
Centers for Disease Control and  
      Prevention,
<https://covid.cdc.gov/covid-data-tracker/#demographics>.

“Databases, Tables & Calculators by Subject.” U.S. Bureau of Labor
Statistics, U.S. Bureau of Labor Statistics,
      <https://www.bls.gov/data/#unemployment>.

Madgavkar, Anu, et al. “Covid-19 and Gender Equality: Countering the
Regressive Effects.” McKinsey & Company,       McKinsey & Company, 10
Apr. 2021,
<https://www.mckinsey.com/featured-insights/future-of-work/covid-19-and->
      gender-equality-countering-the-regressive-effects.

“Www.unwomen.org.” UNWomen.org, United Nations, 9 Apr. 2020,
      <https://www.unwomen.org/-/media/headquarters/attachments/sections/library/publications/2021/egm-background>
      -papers-financing-for-gender-equality-in-the-hiv-response-en.pdf?la=en&vs=5214.
