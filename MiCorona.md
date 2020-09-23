Michigan COVID Data
================
Adam DenHaan
23 September, 2020

Read in data:

``` r
download.file("https://www.michigan.gov/documents/coronavirus/Cases_by_County_and_Date_2020-09-22_703063_7.xlsx", destfile = "/tmp/file.xlsx")
mi_data = data.frame(readxl::read_excel("/tmp/file.xlsx"))
head(mi_data)
```

    ##   COUNTY       Date CASE_STATUS Cases Deaths Cases.Cumulative Deaths.Cumulative
    ## 1 Alcona 2020-03-01   Confirmed     0      0                0                 0
    ## 2 Alcona 2020-03-02   Confirmed     0      0                0                 0
    ## 3 Alcona 2020-03-03   Confirmed     0      0                0                 0
    ## 4 Alcona 2020-03-04   Confirmed     0      0                0                 0
    ## 5 Alcona 2020-03-05   Confirmed     0      0                0                 0
    ## 6 Alcona 2020-03-06   Confirmed     0      0                0                 0
    ##               Updated
    ## 1 2020-09-22 14:10:26
    ## 2 2020-09-22 14:10:26
    ## 3 2020-09-22 14:10:26
    ## 4 2020-09-22 14:10:26
    ## 5 2020-09-22 14:10:26
    ## 6 2020-09-22 14:10:26

Manipulate Data:

``` r
mi_cases_by_day = mi_data %>% 
  na.omit() %>%
  group_by(Date) %>%
  mutate(
    Cases = sum(Cases),
    Date = date(Date)
  ) 

mi_cases_by_day_exclusive <- mi_cases_by_day %>%
  filter(                                  #most recent data is often inaccurate and revised
    Date < date(now()) - 4,
  )

mi_cases_by_day_last4 <- mi_cases_by_day %>%
  filter(                         
    Date >= date(now()) - 4,
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

![](MiCorona_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

Note that the last 4-5 days of data have been colored red on the graph,
as they frequently change as more infomration becomes available.
