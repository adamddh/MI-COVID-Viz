Michigan COVID Data
================
Adam D. DenHaan
May 12, 2021

``` python
from urllib.request import urlopen
from bs4 import BeautifulSoup


url = "https://www.michigan.gov/coronavirus/0,9753,7-406-98163_98173---,00.html"

html = urlopen(url).read().decode("utf-8")
start_index = html.find("shortdesc")
end_index = html.find("footerArea")
data = html[start_index:end_index]

soup = BeautifulSoup(data)
links = [link.get('href') for link in soup.find_all('a')]

finallink = "https://michigan.gov" + \
    [i for i in links if "by_Date" in i][0]
```

Read in data:

``` r
link = py$finallink

download.file(link, destfile = "data/covid.xlsx")

mi_data = readxl::read_excel("data/covid.xlsx")

glimpse(mi_data)
```

    ## Rows: 77,093
    ## Columns: 8
    ## $ COUNTY            <chr> "Alcona", "Alcona", "Alcona", "Alcona", "Alcona", "A…
    ## $ Date              <dttm> 2020-03-01, 2020-03-02, 2020-03-03, 2020-03-04, 202…
    ## $ CASE_STATUS       <chr> "Confirmed", "Confirmed", "Confirmed", "Confirmed", …
    ## $ Cases             <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
    ## $ Deaths            <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
    ## $ Cases.Cumulative  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
    ## $ Deaths.Cumulative <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
    ## $ Updated           <dttm> 2021-05-12 13:16:45, 2021-05-12 13:16:45, 2021-05-1…

Wrangle Data:

``` r
date_update = format(max(mi_data$Updated), '%d %b %Y')

#simplify cases
simple_cases <- aggregate(
  mi_data$Cases,
  by = list(Category = mi_data$Date),
  FUN = sum
) %>%
  rename(Date = Category, Cases = x)

#simplify deaths
simple_deaths <- aggregate(
  mi_data$Deaths,
  by = list(Category = mi_data$Date),
  FUN = sum
) %>%
  rename(Date = Category, Deaths = x)

#join data
simple_data <-
  inner_join(simple_cases, simple_deaths) %>%
  mutate(Date = date(Date))
```

    ## Joining, by = "Date"

``` r
day_split = 7

#split data by date
mi_cases_by_day_exclusive <- simple_data %>%
  dplyr::filter(                                   
    Date < date(now()) - day_split + 1,
  )

mi_cases_by_day_last4 <- simple_data %>%
  dplyr::filter(                         
    Date >= date(now()) - day_split + 1,
  )
```

Visualization:

``` r
mi_cases_by_day_exclusive %>%
  ggplot(mapping = aes(x = Date, y = Cases)) +
  geom_vline(xintercept = today() - 28, color = "orange") +
  ylim(c(0,NA)) +
  geom_point(alpha = .5) +
  geom_smooth(
    method = "gam",
    formula = y ~ s(x, bs = "cs", k = 20),
    se = FALSE
  ) +
  geom_point(
    data = mi_cases_by_day_last4,
    mapping = aes(x = Date, y = Cases, color = "red"),
  ) +
  scale_x_date(date_labels = "%m-%d",
               date_breaks = "1 month") +
  theme(legend.position = "none") +
  labs(title = paste("Michigan Coronavirus Cases, updated", date_update))
```

    ## Warning: Removed 1 rows containing missing values (geom_smooth).

![](MiCorona_files/figure-gfm/viz-1.png)<!-- -->

``` r
mi_cases_by_day_exclusive %>%
  ggplot(mapping = aes(x = Date, y = Deaths)) +
  ylim(c(0,NA)) +
  geom_point(alpha = .5) + 
  geom_smooth(
    method = "gam",
    formula = y ~ s(x, bs = "cs", k = 20),
    se = FALSE
  ) +
  geom_point(
    data = mi_cases_by_day_last4,
    mapping = aes(x = Date, y = Deaths, color = "red"),
  ) +
  scale_x_date(date_labels = "%m-%d",
               date_breaks = "1 month") + 
  theme(legend.position = "none") +
  labs(title = paste("Michigan Coronavirus Deaths, updated", date_update))
```

    ## Warning: Removed 2 rows containing missing values (geom_smooth).

![](MiCorona_files/figure-gfm/viz2-1.png)<!-- -->

Note that the last 7 days of data have been colored red on the graph, as
they frequently change as more information becomes available. Vertical
orange line at 4 weeks in the past (as hospitalizations and deaths
usually follow cases by three weeks, and the deaths graph regression
line is behind by another week).
