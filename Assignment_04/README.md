Assignment 4
================
Christina Lin
11/16/2021

# Part 1: HPC

## Problem 1

``` r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}

fun1alt <- function(mat) {
  rowSums(mat)
}

# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}

fun2alt <- function(mat) {
  t(apply(X = mat,MARGIN = 1, FUN = cumsum))
}


# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), unit = "relative", check = "equivalent"
)
```

    ## Warning in microbenchmark::microbenchmark(fun1(dat), fun1alt(dat), unit
    ## = "relative", : less accurate nanosecond times to avoid potential integer
    ## overflows

    ## Unit: relative
    ##          expr      min       lq     mean   median       uq       max neval
    ##     fun1(dat) 30.94215 29.22846 15.42998 28.81139 28.13158 0.4210267   100
    ##  fun1alt(dat)  1.00000  1.00000  1.00000  1.00000  1.00000 1.0000000   100

``` r
# Test for the second
microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), unit = "relative", check = "equivalent"
)
```

    ## Unit: relative
    ##          expr     min       lq     mean   median      uq       max neval
    ##     fun2(dat) 4.06914 3.549097 2.637867 3.498348 3.45955 0.2318784   100
    ##  fun2alt(dat) 1.00000 1.000000 1.000000 1.000000 1.00000 1.0000000   100

## Problem 2

``` r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

    ## [1] 3.132

``` r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

    ## [1] 3.14124

    ##    user  system elapsed 
    ##   0.690   0.197   0.888

Now re-writing using parLapply:

``` r
library(parallel)

system.time({
  cl <- makePSOCKcluster(4L)
  clusterSetRNGStream(cl,1231)
  clusterExport(cl,"sim_pi")
  ans <- unlist(parLapply(cl, rep(4000,10000),sim_pi))
  print(mean(ans))
  stopCluster(cl)
})
```

    ## [1] 3.141521

    ##    user  system elapsed 
    ##   0.006   0.002   0.447

# Part 2: SQL

``` r
# install.packages(c("RSQLite", "DBI"))

library(RSQLite)
library(DBI)

# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
film <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film.csv")
film_category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film_category.csv")
category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/category.csv")

# Copy data.frames to database
dbWriteTable(con, "film", film)
dbWriteTable(con, "film_category", film_category)
dbWriteTable(con, "category", category)
```

## Question 1

How many movies aew there avaliable in each rating catagory.

``` sql
SELECT rating, COUNT (*) AS count
FROM film
GROUP BY rating
```

| rating | count |
|:-------|------:|
| G      |   180 |
| NC-17  |   210 |
| PG     |   194 |
| PG-13  |   223 |
| R      |   195 |

5 records

## Question 2

What is the average replacement cost and rental rate for each rating
category.

``` sql
SELECT 
  AVG(replacement_cost) AS avg_replacement_cost,
  AVG(rental_rate) AS avg_rental_rate,
  rating
FROM film
GROUP BY rating
```

| avg_replacement_cost | avg_rental_rate | rating |
|---------------------:|----------------:|:-------|
|             20.12333 |        2.912222 | G      |
|             20.13762 |        2.970952 | NC-17  |
|             18.95907 |        3.051856 | PG     |
|             20.40256 |        3.034843 | PG-13  |
|             20.23103 |        2.938718 | R      |

5 records

## Question 3

Use table film_category together with film to find the how many films
there are with each category ID

``` sql
SELECT COUNT(*) AS "N_Films", category_id
FROM film
INNER JOIN film_category
ON film.film_id = film_category.film_id
GROUP BY category_id
```

| N_Films | category_id |
|--------:|------------:|
|      64 |           1 |
|      66 |           2 |
|      60 |           3 |
|      57 |           4 |
|      58 |           5 |
|      68 |           6 |
|      62 |           7 |
|      69 |           8 |
|      73 |           9 |
|      61 |          10 |

Displaying records 1 - 10

## Question 4

Incorporate table category into the answer to the previous question to
find the name of the most popular category.

``` sql
SELECT COUNT(*) AS "N_Films", name
FROM film
  INNER JOIN film_category
    ON film.film_id = film_category.film_id
  INNER JOIN category
    ON film_category.category_id = category.category_id
GROUP BY name
ORDER BY N_Films DESC
```

| N_Films | name        |
|--------:|:------------|
|      74 | Sports      |
|      73 | Foreign     |
|      69 | Family      |
|      68 | Documentary |
|      66 | Animation   |
|      64 | Action      |
|      63 | New         |
|      62 | Drama       |
|      61 | Sci-Fi      |
|      61 | Games       |

Displaying records 1 - 10
