COVID-19
================
Andrew DeCandia
10/6/2021

``` r
library(tidyverse)
```

    ## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --

    ## v ggplot2 3.3.5     v purrr   0.3.4
    ## v tibble  3.1.4     v dplyr   1.0.7
    ## v tidyr   1.1.3     v stringr 1.4.0
    ## v readr   2.0.1     v forcats 0.5.1

    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(ggrepel)
```

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
filename <- "./data/DS_monthly_breakdowns_all.csv"
df_all <- read_csv(filename)
```

    ## Rows: 2451 Columns: 4

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (2): Month, Industry
    ## dbl (2): Year, Count

    ## 
    ## i Use `spec()` to retrieve the full column specification for this data.
    ## i Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
filename <- "./data/DS_monthly_breakdowns_women.csv"
df_women <- read_csv(filename)
```

    ## Rows: 2451 Columns: 4

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (2): Month, Industry
    ## dbl (2): Year, Count

    ## 
    ## i Use `spec()` to retrieve the full column specification for this data.
    ## i Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
colnames(df_women)[4] <- "Count_Women"

colnames(df_all)[4] <- "Count_All"
```

``` r
df_women <-
  df_women %>%
  mutate(
    Date = as.Date(paste(Year, Month, 1, sep = " "), format = '%Y %b %d')
  )

df_all <-
  df_all %>%
  mutate(
    Date = as.Date(paste(Year, Month, 1, sep = " "), format = '%Y %b %d')
  )
```

``` r
df_all_vs_women <-
  df_all %>%
  left_join(df_women) %>%
  subset(select = c(3,4,5,6))
```

    ## Joining, by = c("Month", "Year", "Industry", "Date")

``` r
df_all_vs_women <-
  df_all_vs_women %>%
  mutate(
    Count_notWomen = Count_All - Count_Women
  ) %>%
  pivot_longer(
    names_to = c(".value", "Group"),
    names_sep = "_",
    cols = c(Count_All, Count_Women, Count_notWomen)
  )
```

``` r
df_all_vs_women <-
  df_all_vs_women %>%
  group_by(Industry, Group) %>%
  arrange(Industry, Group, Date) %>%
  mutate(
    Percent_rate_of_change =  100 * (Count - lag(Count))/lag(Count)
  ) %>%
  ungroup()
```

``` r
df_all_vs_women %>%
  filter(
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  facet_wrap(vars(Industry))
```

![](AD_project_01_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
df_all_vs_women %>%
  filter(
    Industry == "Retail trade",
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  facet_wrap(vars(Industry)) +
  scale_color_manual(values = c("blue", "red")) + 
  theme_common()
```

![](AD_project_01_files/figure-gfm/unnamed-chunk-8-2.png)<!-- -->

``` r
df_all_vs_women %>%
  filter(
    Industry == "Service-providing",
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  facet_wrap(vars(Industry)) + 
  scale_color_manual(values = c("blue", "red")) + 
  theme_common()
```

![](AD_project_01_files/figure-gfm/unnamed-chunk-8-3.png)<!-- -->

``` r
df_all_vs_women %>%
  filter(
    Industry == "Education and health services",
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  facet_wrap(vars(Industry)) + 
  scale_color_manual(values = c("blue", "red")) + 
  theme_common()
```

![](AD_project_01_files/figure-gfm/unnamed-chunk-8-4.png)<!-- -->

``` r
df_all_vs_women %>%
  filter(
    Industry == "Total nonfarm",
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  facet_wrap(vars(Industry)) + 
  scale_color_manual(values = c("blue", "red")) +
  theme_common()
```

![](AD_project_01_files/figure-gfm/unnamed-chunk-8-5.png)<!-- -->

``` r
df_all_vs_women %>%
  filter(
    Industry == "Goods-producing",
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  facet_wrap(vars(Industry)) + 
  scale_color_manual(values = c("blue", "red")) + 
  theme_common()
```

![](AD_project_01_files/figure-gfm/unnamed-chunk-8-6.png)<!-- -->

``` r
df_all_vs_women %>%
  filter(
    Industry == "Construction",
    Group != "All",
    Date > "2018-12-1"
  ) %>%
  ggplot(aes(x = Date, y = Percent_rate_of_change)) + 
  geom_line(aes(color = Group)) + 
  facet_wrap(vars(Industry)) + 
  scale_color_manual(values = c("blue", "red")) + 
  theme_common()
```

![](AD_project_01_files/figure-gfm/unnamed-chunk-8-7.png)<!-- -->
