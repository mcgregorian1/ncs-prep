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

READ MORE ABOUT FOREACH AND DOPARALLEL
