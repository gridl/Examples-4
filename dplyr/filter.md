filter
================
2/4/2018

Set up the problem (from [here](https://github.com/tidyverse/dplyr/issues/3335)).

``` r
library("feather")
df <- as.data.frame(matrix(nrow = 1, 
                           ncol = 100000,
                           data = 0.0))
```

[R](https://www.r-project.org) timing.

``` r
system.time(fetched_sample <- df[df$V1>1, , drop=FALSE])
```

    ##    user  system elapsed 
    ##   0.664   0.011   0.696

[dplyr](https://CRAN.R-project.org/package=dplyr) timing.

``` r
library("dplyr")

system.time(fetched_sample <- filter(df, V1>1))
```

    ##    user  system elapsed 
    ## 148.677  10.173 168.767

``` r
tb <- as_tibble(df)

td <- system.time(fetched_sample <- filter(tb, V1>1))
print(td)
```

    ##    user  system elapsed 
    ## 129.081  16.474 149.685

[data.table](https://CRAN.R-project.org/package=data.table) timing.

``` r
library("data.table")

dt <- data.table(df)

system.time(dfr <- dt[V1>1, ])
```

    ##    user  system elapsed 
    ##   1.895   0.023   1.949

[Python](https://www.python.org) [Pandas](https://pandas.pydata.org) timing.

``` r
start_pandas <- Sys.time()
system.time(write_feather(df, "df.feather"))
```

    ##    user  system elapsed 
    ##   0.271   0.214   0.504

``` python
import pandas
import feather
import timeit
start_time = timeit.default_timer()
df = feather.read_dataframe('df.feather')
print(type(df))
```

    ## <class 'pandas.core.frame.DataFrame'>

``` python
print(df.shape)
```

    ## (1, 100000)

``` python
end_time = timeit.default_timer()
# seconds
print(end_time - start_time)
```

    ## 1.9789928130339831

``` python
start_time = timeit.default_timer()
df_filtered = df.query('V1>1')
sp = df_filtered.shape
end_time = timeit.default_timer()
# seconds
print(end_time - start_time)
```

    ## 4.00986497302074

``` r
end_pandas <- Sys.time()
print(start_pandas)
```

    ## [1] "2018-02-13 07:32:19 PST"

``` r
print(end_pandas)
```

    ## [1] "2018-02-13 07:32:29 PST"

``` r
print(end_pandas - start_pandas)
```

    ## Time difference of 10.10463 secs

Characterize dplyr dependence on column count. In the plot the nearest pure linear and quadratic power laws are plotted as dashed lines.

``` r
library("ggplot2")
library("dplyr")

sizes <- round(exp(seq(from=log(10), 
                       to=log(100000), 
                       length.out=20)))
frames <- lapply(
  sizes,
  function(nc) {
    df <- as.data.frame(matrix(nrow = 1, 
                               ncol = nc,
                               data = 0.0))
    tb <- as_tibble(df)
    gc() # try to kee this out of timing
    ti <- system.time(fetched_sample <- filter(tb, V1>1))
    data.frame(ncol = nc, 
               duration_seconds = as.numeric(ti[["elapsed"]]))
  })
frames <- bind_rows(frames)

WVPlots::LogLogPlot(frames, "ncol", "duration_seconds", 
                    title = "dplyr filter task durations on log-log paper (slope estimates power law)")
```

    ## `geom_smooth()` using method = 'loess'

![](filter_files/figure-markdown_github/shape-1.png)

More discussion can be found [here](http://www.win-vector.com/blog/2018/02/is-10000-cells-big/).
