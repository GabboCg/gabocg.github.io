## Why did I finally write an R package?

My love of data and the NBA naturally pushes me toward a lot of side projects involving basketball. These projects inevitably involve some sort of data retrieval and visualization. After scraping data from multiple sources, generally in a way that was specific to whatever project I was doing, reproducibility was possible, but time consuming as my scripts were often disorganized or slightly different. Organizing everything into a single package made a ton of sense and would allow me to gain some good experience with R packages.  

Rather than the basketball analytics side of things, this post will focus on where I got stuck with my package and build on some of the excellent resources already out there.


## Getting Started

There are already a lot of resources out there to get you started with creating your first R package. The two that I relied on most for setting up this package were:<br/>
* Hilary Parker's classic ['Writing an R package from scratch'](https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/)
* Fong Chun Chan's ['Making Your First R Package'](http://tinyheero.github.io/jekyll/update/2015/07/26/making-your-first-R-package.html)

Both of these posts are incredible resources. Hilary Parker's post will get you up and running really quickly, while Fong Chun Chan's goes into a bit more detail. Don't skip either one. 

## Where I Still Ran Into Problems

Despite the excellent resources noted above, there were still a few places where I ran into problems when my package started to get more complicated. These were mostly really simple obstacles, but I just happened to miss the discussion of them in the above posts and ended up wasting a bunch of time trying to figure things out so thought maybe others had the same problem. 

When I started writing my package, I simply created a new file for each function. This worked beautifully up to a point. However, I got to a place where it just didn't make sense to create a new file for a small helper function that would only be used by the main function within the file. 

So here's a version of what I tried first using some toy examples. __SPOILER: it doesn't work.__ Let's set aside the horrific redundancy of the code below and just focus on the exporting of the functions. We want the user of the package to have access to `good_basketball_fan` after installing the package. 

What happens here is that ONLY the first function `is_awesome()` will be exported into the NAMESPACE and the function we actually want to use will be ignored. Installing our package with the documentation  in this form will only give us access to `is_awesome()`.


```
#' 
#' This function tells you what kind of basketball fan you are.
#' @param player string representing name of basketball player
#' @export
#' @examples
#' good_basketball_fan('stephen curry')

#helper function for good_basketball_fan()
is_awesome <- function(player='stephen curry') {
  if (player=='stephen curry') {
    return(TRUE)
  } else {
    return (FALSE)
  }
}

good_basketball_fan <- function(player) {
  if (is_awesome(player)) {
    print("I'm a super awesome basketball fan!")
  } else {
    print("I'm a terrible basketball fan.")
  }
}
```

### The Fix

Why doesn't the code above work? The `@export` tag is only being applied to the `is_awesome` function. There are two ways to fix this. Using the `@export` tag adds the chosen function to the package NAMESPACE and then exposes it to users. So the first way is to just use documentation in the same format before each function, sequentially in the file. However, say we don't want `is_awesome` to be exposed to the user. In that case we can explicitly export `good_basketball_fan` as follows:

```
#' 
#' This function tells you what kind of basketball fan you are.
#' @param player string representing name of basketball player
#' @export good_basketball_fan
#' @examples
#' good_basketball_fan('stephen curry')

#helper function for good_basketball_fan()
is_awesome <- function(player='stephen curry') {
  if (player=='stephen curry') {
    return(TRUE)
  } else {
    return (FALSE)
  }
}

good_basketball_fan <- function(player) {
  if (is_awesome(player)) {
    print("I'm a super awesome basketball fan!")
  } else {
    print("I'm a terrible basketball fan.")

```

The only difference between the first and second code snippets are that in the second, we explicitly export the main function in our file. Without the explicit call to `good_basketball_fan` (that we see in the second code snippet) the `@export` tag is applied to the top function and the user won't be exposed to the function `good_basketball_fan`.

## Importing Other Packages

As Fong Chun Chan notes, the beginning of a file in your package should never look like this:

```
library(tidyverse); library(viridis)
```

Loading entire packages is going to slow the performance of your package. Instead, when you make calls to functions from packages, it should always go something like this:

```
url <- "https://raw.githubusercontent.com/emilykuehler/basketballstatsR/master/data-raw/league-shot-chart-summary.csv"
shot_zone_league_df <- readr::read_csv(url) %>%
    dplyr::filter(year==year)
```

But wait! We forgot something. That's going to return an error along the lines of `Error: could not find function "%>%"` 

For extremely common functions such as the pipes operator, the `@importFrom` tag is extremely useful. The documentation then just looks like:

```
#' 
#' This function tells you what kind of basketball fan you are.
#' @param player string representing name of basketball player
#' @importFrom magrittr %>%
#' @export good_basketball_fan
#' @examples
#' good_basketball_fan('stephen curry')
```

With this tag you can conveniently import common functions without slowing down the performance of your package by loading an entire package.

## Yay, R packages!

R Packages are really helpful for organization, reproducibility, maintenance and distribution. I can't believe it took me this long to build an R package. It's pretty straightforward and really helpful with workflow. I hope some of this is helpful for building your next R package!








