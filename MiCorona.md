Michigan COVID Data
================
Adam DenHaan
12 October, 2020

Read in data:

``` r
download.file("https://www.michigan.gov/documents/coronavirus/Cases_by_County_and_Date_2020-10-10_704800_7.xlsx", destfile = "/tmp/file.xlsx")

mi_data = readxl::read_excel("/tmp/file.xlsx")

head(mi_data)
```

    ## # A tibble: 6 x 8
    ##   COUNTY Date                CASE_STATUS Cases Deaths Cases.Cumulative
    ##   <chr>  <dttm>              <chr>       <dbl>  <dbl>            <dbl>
    ## 1 Alcona 2020-03-01 00:00:00 Confirmed       0      0                0
    ## 2 Alcona 2020-03-02 00:00:00 Confirmed       0      0                0
    ## 3 Alcona 2020-03-03 00:00:00 Confirmed       0      0                0
    ## 4 Alcona 2020-03-04 00:00:00 Confirmed       0      0                0
    ## 5 Alcona 2020-03-05 00:00:00 Confirmed       0      0                0
    ## 6 Alcona 2020-03-06 00:00:00 Confirmed       0      0                0
    ## # … with 2 more variables: Deaths.Cumulative <dbl>, Updated <dttm>

Wrangle Data:

``` r
mi_cases_by_day = mi_data %>% 
  group_by(Date) %>%
  mutate(
    Cases = sum(Cases),
    Date = date(Date)
  ) 

day_split = 3

mi_cases_by_day_exclusive <- mi_cases_by_day %>%
  filter(                                  #most recent data is often inaccurate and revised
    Date < date(now()) - day_split,
  )

mi_cases_by_day_last4 <- mi_cases_by_day %>%
  filter(                         
    Date >= date(now()) - day_split,
  )
```

Visualization:

``` r
mi_cases_by_day_exclusive %>%
  ggplot(mapping = aes(x = Date, y = Cases)) +
  ylim(c(0,1700)) +
  geom_point() + 
  geom_smooth() +
  geom_point(
    data = mi_cases_by_day_last4,
    mapping = aes(x = Date, y = Cases, color = "red"),
  ) +
  scale_x_date(date_labels = "%m-%d",
               date_breaks = "2 weeks") + 
  theme(legend.position = "none")
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

    ## Warning: Removed 2 rows containing missing values (geom_smooth).

![](MiCorona_files/figure-gfm/viz-1.png)<!-- -->

Note that the last 4-5 days of data have been colored red on the graph,
as they frequently change as more infomration becomes available.
