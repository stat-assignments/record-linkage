# Linking Records

Organizations frequently maintain information about the same individuals
that is collected from different sources.

Combining these databases provides a more complete picture of
individuals, but identifying records that refer to the same person is
challenging. Names may be misspelled, addresses may be incomplete, and
dates of birth or other identifying information may be missing or
incorrectly recorded.

In this assignment, you will investigate and compare results from a
record-linkage procedure using the FEBRL 4 dataset. Your goal is to
identify pairs of records that refer to the same individual, evaluate
the quality of your linkage procedure, and investigate the consequences
of different methodological choices.

## Data

The FEBRL 4 dataset (available through the R package `irelink`) consists
of two files:

- `febrl4a`: 5,000 original records.
- `febrl4b`: 5,000 corresponding duplicate records.

Each record contains information such as first and last name, street
address, suburb, postcode, state, date of birth, and social security
identifier.

The duplicate records contain deliberately introduced errors, including
typographical errors, missing values, and transpositions.

Load the package and the data:

    library(irelink)

    ## Warning: package 'irelink' was built under R version 4.6.1

    ## 
    ## Attaching package: 'irelink'

    ## The following object is masked from 'package:base':
    ## 
    ##     months

    library(tidyverse)

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.2.1          ✔ readr     2.2.0     
    ## ✔ forcats   1.0.1          ✔ stringr   1.6.0     
    ## ✔ ggplot2   4.0.3.9000     ✔ tibble    3.3.1     
    ## ✔ lubridate 1.9.5          ✔ tidyr     1.3.2     
    ## ✔ purrr     1.2.2

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ lubridate::days()    masks irelink::days()
    ## ✖ dplyr::filter()      masks stats::filter()
    ## ✖ lubridate::hours()   masks irelink::hours()
    ## ✖ dplyr::lag()         masks stats::lag()
    ## ✖ lubridate::minutes() masks irelink::minutes()
    ## ✖ lubridate::seconds() masks irelink::seconds()
    ## ✖ lubridate::years()   masks irelink::years()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

    library(stringdist)

    ## 
    ## Attaching package: 'stringdist'
    ## 
    ## The following object is masked from 'package:tidyr':
    ## 
    ##     extract

    data(febrl4a)
    data(febrl4b)

The `rec_id` variable contains information about the underlying entity.
**Do not use this variable when constructing your linkage procedure.**
You may use the `rec_id` variables only when you are asked to evaluate
your results.

# Question 1: Getting an overview of the situation

Before attempting to link the records, familiarize yourself with the two
databases. Answer the following questions and present appropriate tables
and/or visualizations to support your observations.

1.  Report the number of observations and variables in each dataset.
2.  Describe the variables and their types.
3.  Investigate the amount of missing information in each variable.
    Compare the amount of missingness between the two databases.
4.  Which variable(s) appear to be particularly useful for record
    linkage? Explain your choice(s).
5.  Investigate the quality of the identifying information:
    - Are there duplicate values within either database?
    - Are there unusual or inconsistent values?
    - Which variables appear to have been corrupted?

# Question 2: Set up an Evaluation Framework

1.  Construct a dataset `AB` of all possible index pairs between the two
    datasets using all combinations of the `rec_id` variables.

2.  Use the values on the two index variables to create a variable
    `match_gt` that contains the ground truth for matches, i.e. is
    `TRUE` if the pair of indices corresponds to a match and `FALSE`
    otherwise.

3.  Create a summary table of the `match_gt` variable and discuss.

# Question 3: Exact Matching

1.  Identify pairs in the two data sets that agree exactly on:
    - Given name and surname.
    - Date of birth.
    - Postcode.
2.  Construct a linkage procedure that identifies pairs agreeing on all
    three criteria.
3.  Identify pairs that agree on the social security id.
4.  Evaluate the resulting matches from 2 and 3 in terms of the number
    of identified pairs, true matches, false matches, and missed
    matches.

Discuss why exact matching is unlikely to identify all true matches.

# Question 4: String Comparisons and Similarity Measures

Exact matching is often too restrictive. For example, two records may
contain the names “Katherine” and “Katharine”, or addresses may differ
by a single character.

Investigate methods for comparing strings.

1.  Use the `stringdist` package to calculate string similarity measures
    for all pairs of given names, surnames, and addresses. Consider both
    Jaro-Winkler similarity and Levenshtein distance.
2.  Compare the similarity distributions for true matching and
    non-matching pairs. Is one similarity measure always ‘better’
    (define!) than the other?
3.  What would be good thresholds for each of the variables?

Illustrate your findings with appropriate plots.

# Question 5: Candidate Generation and Blocking

Comparing all possible pairs between two datasets of 5,000 records each
requires 25 million comparisons.

To make record linkage more computationally efficient, **blocking**
variables are used, i.e. only those records are compared that agree on
selected characteristics. These records are called candidate pairs. Two
important measures are considered when using blocking:

- **Pair completeness:** proportion of true matches retained in the
  candidate set.
- **Reduction ratio:** proportion of all possible pairs eliminated by
  blocking.

1.  Create candidate pairs for all records that agree on (a) birth
    year, (b) birth day (without the year), and (c) the first two
    letters of the surname.
2.  For each blocking strategy, report the number of candidate pairs.
3.  Calculate pair completeness and reduction ratio for each of the
    blocking strategies.
4.  Discuss the trade-off between computational efficiency and the risk
    of excluding true matches.

# Question 6: Probabilistic Record Linkage with `ireLink`

The `irelink` package provides functions for specifying comparisons,
blocking, estimating model parameters, and predicting matches. See below
for instructions on how to use the package.

The overall goal of this question is to use the `irelink` package to fit
and evaluate a record linkage model.

1.  Create a specification with `il_spec` that compares records between
    the two datasets using the insights from the previous questions.
    Explain your choices.

2.  Fit the model and predict match probabilities (use a threshold of
    0.5 to start).

3.  Evaluate the identified pairs against ground truth.

The `irelink` website [describes its functionality at the example of the
febrl
data](https://christophertkenny.com/irelink/articles/record-linkage.html).

An example specification is:

    library(irelink)

    spec <- il_spec() |>
      il_compare(given_name, cl_name()) |>
      il_compare(surname, cl_name()) |>
      il_compare(date_of_birth, cl_exact()) |>
      il_compare(postcode, cl_exact()) |>
      il_block_on(surname) |>
      il_block_on(given_name)

Fit a linkage model:

    con <- DBI::dbConnect(duckdb::duckdb())

    ## duckdb is storing downloaded extensions and secrets under ~/.duckdb:
    ## ℹ /Users/hofmann/.duckdb
    ## This persists across sessions and is shared with the DuckDB CLI and other clients.
    ## ℹ Run duckdb(shared_home = FALSE) to use a temporary directory instead.
    ## ℹ See ?duckdb_storage for details and alternatives.

    model <- il_model(
      febrl4a,
      febrl4b,
      spec = spec,
      con = con,
      link_type = "link"
    )

Estimate model parameters (Note, that this is a pipeline that
successively changes the model!):

    model <- il_estimate_prior(
      model,
      block_on(given_name, surname),
      block_on(surname, suburb),
      recall = 0.6
    )

    model <- il_estimate_u(model, max_pairs = 1e5)

    model <- il_estimate_em(model, block_on(surname))

    ## EM trained: given_name, date_of_birth, and postcode | skipped (blocked on):
    ## surname

    model <- il_estimate_em(model, block_on(suburb))

    ## EM trained: given_name, surname, date_of_birth, and postcode

Inspect the estimated model:

    autoplot(model)

![](README_files/figure-markdown_strict/unnamed-chunk-4-1.png)

    autoplot(model, type = "parameters")

![](README_files/figure-markdown_strict/unnamed-chunk-4-2.png)

    il_weights(model)

    ## # A tibble: 14 × 5
    ##    comparison    gamma_level  m_prob   u_prob weight
    ##    <chr>               <int>   <dbl>    <dbl>  <dbl>
    ##  1 given_name              0 0.157   0.971    -2.63 
    ##  2 given_name              1 0.0135  0.0234   -0.787
    ##  3 given_name              2 0.0138  0.000816  4.08 
    ##  4 given_name              3 0.127   0.00115   6.79 
    ##  5 given_name              4 0.689   0.00381   7.50 
    ##  6 surname                 0 0.125   0.980    -2.97 
    ##  7 surname                 1 0.00698 0.0146   -1.06 
    ##  8 surname                 2 0.0165  0.000515  5.00 
    ##  9 surname                 3 0.182   0.000768  7.89 
    ## 10 surname                 4 0.670   0.00418   7.32 
    ## 11 date_of_birth           0 0.104   1.000    -3.26 
    ## 12 date_of_birth           1 0.896   0.000243 11.8  
    ## 13 postcode                0 0.165   0.999    -2.59 
    ## 14 postcode                1 0.835   0.00112   9.54

Generate predictions:

    predictions <- predict(model, threshold = 0.5)

Documentation and examples:
<https://cran.r-project.org/web/packages/irelink/vignettes/record-linkage.html>

## Submission

Make sure that your file `index.qmd` includes all details to make your
answers fully reproducible. Ensure that the file renders properly. Add
all relevant(!) files to the repository, commit, and push!
