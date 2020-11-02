Michigan COVID Data
================
Adam D. DenHaan
Nov 02, 2020

Read in data:

``` r
link = "https://www.michigan.gov/documents/coronavirus/Cases_and_Deaths_by_County_and_by_Date_of_Symptom_Onset_or_by_Date_of_Death2020-10-31_706668_7.xlsx"
download.file(link, destfile = "/tmp/file.xlsx")

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

# linkdate <- as.Date(strsplit(link, "_")[[1]][16])
# linkandnowdiff <- day(now()) - day(linkdate)

day_split = 4

mi_cases_by_day_exclusive <- mi_cases_by_day %>%
  filter(                                   #most recent data is often inaccurate and revised
    Date < date(now()) - day_split + 1,
  )

mi_cases_by_day_last4 <- mi_cases_by_day %>%
  filter(                         
    Date >= date(now()) - day_split + 1,
  )
```

Visualization:

``` r
mi_cases_by_day_exclusive %>%
  ggplot(mapping = aes(x = Date, y = Cases)) +
  ylim(c(0,NA)) +
  geom_point() + 
  geom_smooth() +
  geom_point(
    data = mi_cases_by_day_last4,
    mapping = aes(x = Date, y = Cases, color = "red"),
  ) +
  scale_x_date(date_labels = "%m-%d",
               date_breaks = "2 weeks") + 
  theme(legend.position = "none") +
  labs(title = "Michigan Coronavirus Cases")
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

    ## Warning: Removed 2 rows containing missing values (geom_smooth).

![](MiCorona_files/figure-gfm/viz-1.png)<!-- -->

Note that the last 2 days of data have been colored red on the graph, as
they frequently change as more information becomes available.
