Michigan COVID Data
================
Adam D. DenHaan
Nov 29, 2020

``` python
from urllib.request import urlopen
from bs4 import BeautifulSoup


url = "https://www.michigan.gov/coronavirus/0,9753,7-406-98163_98173---,00.html"

page = urlopen(url)
html_bytes = page.read()
html = html_bytes.decode("utf-8")
start_index = html.find("shortdesc")
end_index = html.find("footerArea")
data = html[start_index:end_index]

soup = BeautifulSoup(data)
links = []

for link in soup.find_all('a'):
    links.append(link.get('href'))

for link in links:
    try:
        link.index("by_Date")
        finallink = "https://michigan.gov" + link
        break
    except:
        pass
```

Read in data:

``` r
link = py$finallink

download.file(link, destfile = "/tmp/file.xlsx")

mi_data = readxl::read_excel("/tmp/file.xlsx")

glimpse(mi_data)
```

    ## Rows: 48,054
    ## Columns: 8
    ## $ COUNTY            <chr> "Alcona", "Alcona", "Alcona", "Alcona", "Alcona", "…
    ## $ Date              <dttm> 2020-03-01, 2020-03-02, 2020-03-03, 2020-03-04, 20…
    ## $ CASE_STATUS       <chr> "Confirmed", "Confirmed", "Confirmed", "Confirmed",…
    ## $ Cases             <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …
    ## $ Deaths            <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …
    ## $ Cases.Cumulative  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …
    ## $ Deaths.Cumulative <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …
    ## $ Updated           <dttm> 2020-11-28 13:25:03, 2020-11-28 13:25:03, 2020-11-…

Wrangle Data:

``` r
date_update = format(max(mi_data$Updated), '%d %b %Y')

mi_cases_by_day = mi_data %>% 
  # filter(COUNTY == "Kent") %>%
  group_by(Date) %>%
  mutate(
    Cases = sum(Cases),
    Date = date(Date)
  ) 

# linkdate <- as.Date(strsplit(link, "_")[[1]][16])
# linkandnowdiff <- day(now()) - day(linkdate)

day_split = 7

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
  geom_smooth(method = "gam") +
  geom_point(
    data = mi_cases_by_day_last4,
    mapping = aes(x = Date, y = Cases, color = "red"),
  ) +
  scale_x_date(date_labels = "%m-%d",
               date_breaks = "3 weeks") + 
  theme(legend.position = "none") +
  labs(title = paste("Michigan Coronavirus Cases, updated ", date_update))
```

    ## `geom_smooth()` using formula 'y ~ s(x, bs = "cs")'

    ## Warning: Removed 1 rows containing missing values (geom_smooth).

![](MiCorona_files/figure-gfm/viz-1.png)<!-- -->

Note that the last 6 days of data have been colored red on the graph, as
they frequently change as more information becomes available.
