# Notes from reading/familiarity assignments

## Parallel package
Every computer has a number of central processing units (cpu) that it can use for running processes. 
In other words, think of trying to process a timeseries' worth of data over a large geographical area. 
You can easily be using 400,000 NetCDF files. 
You can either run everything on one cpu (which would take forever and you'd probably run out of memory), OR you spread out the calculations across the different cpus so that they're run simultaneously, take up less time, and take up less memory.

Not all of a code can be parallelized, and sometimes, the act of parallelizing code can render the computation time longer than if parallelization did not occur.
This is based on a function between the number of cores (cpus) and the % of code that can be parallelized.
See the graph [here](https://nceas.github.io/oss-lessons/parallel-computing-in-r/parallel-computing-in-r.html) to visualize.

### Code set-up
Ideal code for parallelization consists of loops that are independent of each other, that is, they don't feed in to each other to run.
Running loops is much faster using the apply family compared to base for-loops. 
In the parallel package, you use mclapply, which is analogous to lapply but tells the different functions to run on separate cores.
- NB your independent loops should be specified within functions
- you can also use multiple processors on local/outside cpus using the functions `makeCluster` and `clusterApply`, though you have to manually copy data and code to each cluster member using `clusterExport`.

```
library(parallel)
library(MASS)

starts <- rep(100, 40)
fx <- function(nstart) kmeans(Boston, 4, nstart=nstart)
numCores <- detectCores()
numCores
## [1] 8
```

```
# time elapsed to run the function with lapply
system.time(
  results <- lapply(starts, fx)
)
##    user  system elapsed 
##   1.346   0.024   1.372

# time elapsed to run the function with mclapply
system.time(
  results <- mclapply(starts, fx, mc.cores = numCores)
)
##    user  system elapsed 
##   0.801   0.178   0.367
```

## a different loop
Normally, a for-loop like this produces three values. 

```
for (i in 1:3) {
  print(sqrt(i))
}
```

Using foreach returns instead each of those values inside its own list, vector, or dataframe depending on how you want it.

```
library(foreach)
foreach (i=1:3) %do% {
  sqrt(i)
}

# Return a vector
foreach (i=1:3, .combine=c) %do% {
  sqrt(i)
}

# Return a data frame
foreach (i=1:3, .combine=rbind) %do% {
  sqrt(i)
}
```

The main reason to use foreach is because it works with another package called doParallel, which allows you to loop through funtions while using the different cores on a computer in parallel.

```
registerDoParallel(numCores)  # use multicore, set to the number of our cores
foreach (i=1:3) %dopar% {
  sqrt(i)
}
```

A full example of this is "using %dopar% to parallelize a bootstrap analysis where a data set is resampled 10,000 times and the analysis is rerun on each sample, and then the results combined."

```
# Let's use the iris data set to do a parallel bootstrap
# From the doParallel vignette, but slightly modified
x <- iris[which(iris[,5] != "setosa"), c(1,5)]
trials <- 10000
system.time({
  r <- foreach(icount(trials), .combine=rbind) %dopar% {
    ind <- sample(100, 100, replace=TRUE)
    result1 <- glm(x[ind,2]~x[ind,1], family=binomial(logit))
    coefficients(result1)
  }
})

#this takes 4.94 seconds compared to >20 seconds when not using parallel
```
