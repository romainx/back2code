---
title: 'Data Deprecation in R-package'
date: '2020-05-08'
categories: ['data']
tags: ['R']
---

After my [recent article]({{< ref "deprecated_r" >}}) on marking **deprecated** code in R, I had the same problem on R data package.
Unfortunately I was not able to find a convenient out-of-the-box solution, see [this question][LK1] on SO.

So after a first draft, I've applied a process that seems to be a reasonable--good enough--solution.

# Move the data file

The first step is to **move the data file** from its default location `./data` to another location in order to avoid its automatic loading--even if it's a lazy loading.

*Note: For this article I have created a dataset called `my_data` (see the Example section for a guideline on how to create an example dataset) that I will use as an example to deprecate it.*

The target location is `./data-raw` it's a directory that is used by convention to store script and raw data in order to be able to update or reproduce the production of the exported dataset--more on that in the [Data chapter of the book *R packages*][LK2].

I'm using by convention a `leg_` prefix to flag it as a **legacy dataset**.

```bash
$ mv ./data/my_data.rda ./data-raw/leg_my_data.rda
```

# Write a script to transform the dataset

The code used to **transform the dataset from its legacy to its new format** is stored along with the legacy dataset in  `./data-raw/my_data.R`. This will make the **whole process reproducible**. 

```R
# my_data new version

library(tidyverse)

# Load legacy data -----
load("data-raw/leg_my_data.rda")
leg_my_data <- my_data

# Create the new dataset -----
# Perform here every change that has to be performed
my_data <- leg_my_data %>%
  rename(cat = categ) %>%
  arrange(categ)

# Write the new dataset ----
usethis::use_data(my_data, overwrite = TRUE, compress = 'xz')
```

Source the file and you're good the new version is live!

```R
source('./data-raw/my_data.R', echo=TRUE)
# ✓ Saving 'my_data' to 'data/my_data.rda'
# ● Document your data (see 'https://r-pkgs.org/data.html')

my_data
# A tibble: 10 x 2
#   categ   val
#   <fct> <int>
# 1 a         9
# 2 a         6
# 3 a         4
```

# Secret sauce

In the `./R/my_package-package.R` file, create a **`legacy_mode` function**. This function will be a **way for the users to load the previous (legacy) version of the datasets if they need to use them for compatibility reason**.

```R
#' Load legacy version of datasets.
#'
#' Load legacy (previous) version of all the datasets for compatibility reason.
#' The environment where data will be loaded can be chosen.
#'
#' @param envdir the environment where the data should be loaded.
#' @param verbose should item names be printed during loading?
#'
#' @export
#'
#' @examples
#' \dontrun{
#' # Default version
#' head(my_data, 3)
#'
#' # A tibble: 3 x 2
#' # categ   val
#' # <fct> <int>
#' # 1 a         9
#' # 2 a         6
#' # 3 a         4
#'
#' # Activate the compatibility mode
#' legacy_mode()
#'
#' # Loading objects:
#' #  my_data
#' # Warning message:
#' # This function replaces datasets with the previous (legacy) version for compatibility reason
#'
#' # Back to legacy (previous) version
#' head(my_data, 3)
#'
#' # A tibble: 3 x 2
#' # cat     val
#' # <fct> <int>
#' # 1 a         9
#' # 2 c         2
#' # 3 b         3
#' }
legacy_mode <- function(envdir = parent.frame(), verbose = TRUE) {
  .Deprecated(msg = "This function replaces datasets with the previous (legacy) version for compatibility reason")
  # TODO: To be improved to load a subset of datasets
  paths <- sort(Sys.glob(c("data-raw/leg_*.rda", "data-raw/leg_*.RData")))
  for (i in 1:length(paths)) {
    load(paths[i], envir = envdir, verbose = verbose)
  }
}
```

# Result

And so now you have access **to both the new version of the dataset available by default and the legacy version if needed for compatibility reasons**.

```R
# The current version
head(my_data, 3)
# A tibble: 3 x 2
  categ   val
  <fct> <int>
1 a         9
2 a         6
3 a         4

# Activation of the legacy mode
legacy_mode()
# Loading objects:
#   my_data
# Warning message:
# This function replaces datasets with the previous (legacy) version for # compatibility reason 

# Legacy version
head(my_data, 3)
# A tibble: 3 x 2
#   cat     val
#   <fct> <int>
# 1 a         9
# 2 c         2
# 3 b         3
```

Do not forget to **document your changes** by updating the dataset documentation in `R/my_data.R`. You can mention in a note the **legacy mode**.

# Notes

## Example

Here is a way to create the example dataset used in this article.

### Create and save the dataset

```R
# A test data frame
set.seed(2)
my_data <- tibble(cat = factor(sample(c("a", "b", "c", "d"), 10, replace = TRUE)), 
       val = sample(1:10))
# Writing the dataset
usethis::use_data(my_data, overwrite = TRUE, compress = 'xz')

# ✓ Saving 'my_data' to 'data/my_data.rda'
# ● Document your data (see 'https://r-pkgs.org/data.html')
```

### Document and export the dataset

Document the dataset by creating the following R script `R/my_data.R`.

```R
#' An example dataset
#'
#' This is an example dataset to illustrate the deprecation process.
#'
#' @docType data
#'
#' @format A tibble with 10 rows and 2 columns
#' \describe{
#'   \item{cat}{A dummy category}
#'   \item{val}{A dummy value}
#' }
#'
#' @rdname my_data
#'
#' @examples
#' head(my_data)
#'
"my_data"
```

Document and export it.

```R
devtools::document(roclets = c('rd', 'collate', 'namespace'))
# Writing my_data.Rd
# Writing NAMESPACE
```

It can now be seen in the package and used directly.

```R
data(package="my_package")
# Data sets in package ‘my_package’:
# my_data         An example dataset

my_data
# A tibble: 10 x 2
#   cat     val
#   <fct> <int>
# 1 a         9
# 2 c         2
```

## References

* [Data Deprecation in R-package][LK1]
* [Data chapter in the book *R packages*][LK2]
* [Taking your data to go with R packages][LK3]

[LK1]: https://stackoverflow.com/questions/33304651/data-deprecation-in-r-package/61168929#61168929
[LK2]: http://r-pkgs.had.co.nz/data.html
[LK3]: https://www.davekleinschmidt.com/r-packages/