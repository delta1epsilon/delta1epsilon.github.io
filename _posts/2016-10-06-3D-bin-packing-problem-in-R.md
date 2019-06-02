---
title:  "3D bin packing problem in R"
date:   2016-10-06 15:04:23
categories: [R, 3D bin packing problem, Genetic Algorithm]
tags: [R, 3D bin packing problem, Genetic Algorithm]
---
First of all, let's define what does "3D bin packing problem" (3DBPP) stand for.
In 3DBPP rectangular boxes must be efficiently orthogonally packed into containers (bins). Packing is said to be efficient if it's done in a way that maximizes containers utilization ratio.


There are a lot of variations of the problem (2D, 3D, packing into single bin or multiple identical bins, with or without different boxes orientation, etc.) and a lot of approaches for solving them (two-level Branch & Bound algorithm, biased random-key genetic algorithm, mixed integer linear programming model, etc.). However, we are going to solve 3D bin packing problem where containers are of various sizes and different orientations of boxes are considered.

The problem is NP hard. However, we can get a good approximate solution to the problem using Genetic Algorithms (GA). GA are never guaranteed to find an optimal solution for any problem, but they will often find a good solution if one exists.

GA are very effective way of quickly finding a reasonable solution to complex problem.
It is very general algorithm and so will work well in any search space.
Sometimes they can take quite a while to run and are therefore not always feasible for real time use.


The algorithm we are going to study is described in ["A Genetic algorithm for the three-dimensional bin packing problem with heterogeneous bins"](https://www.researchgate.net publication/273121476_A_genetic_algorithm_for_the_three-dimensional_bin_packing_problem_with_heterogeneous_bins) by Xueping Li, Zhaoxia Zhao and Kaike Zhang.


## Outline of the algorithm

### Initialization

Assume we need to pack `m` boxes into `N` containers.

We start with initializing a population of `n` chromosomes - suitable solutions for the problem. Each chromosome consists of 2 parts: box packing sequence (BPS) and container loading sequence (CLS). Box packing sequence is permutations of `{1,...,m}` that defines an order the boxes are packed and container loading sequence is permutation of `{1,...,N}` containers that defines an order in which the containers are loaded.

Four special chromosomes are created in a population by sorting BPS in descending order according to boxes volume, length, width and height. The rest of chromosomes are generated randomly.

### Packing

In next step each chromosome is being mapped into a packing solution.

A concept of empty maximal spaces (EMS) is used to represent free space in containers. Essentially, EMS is a list of largest empty cubic space available for packing not contained in any other EMS. Empty maximal spaces are represented with their maximum and minimum coordinates. EMS is displayed in picture below.

![](http://delta1epsilon.github.io/assets/ems_plot.png)

[Figure source](https://www.researchgate.net/publication/273121476_A_genetic_algorithm_for_the_three-dimensional_bin_packing_problem_with_heterogeneous_bins)


We prefer EMS 1 to EMS 2 if minimum coordinates of 1 are smaller than minimum coordinates of EMS 2.

Once we have chosen EMS, we have to choose box orientation. When a box has several possible placements in one EMS, than the one with smallest margin is selected.  


### Evaluation

After a packing procedure for current population was done, we evaluate a fitness of each chromosome using measure of free space in the containers.

### Selection

A new population is created by performing **Selection** and **Crossover** on existing population.

`E` chromosomes with the best fit within the population are selected directly to next population, so the best solution found can survive to end of run. And the rest chromosomes are selected for tournament.

To perform tournament we randomly select two chromosomes and compare them. Winner goes to mating pool as parent in hope that the better parents will produce better offsprings.


### Crossover and mutation

When promising parents are selected it's time for crossover.

Crossover creates a new offsprings using parents chromosomes.
Crossover is made in hope that new chromosomes will have good parts of old chromosomes and maybe the new chromosomes will be better.
However, need to leave some part of population to survive to next generation.

Each pair of chromosomes in mating pool goes to the next population with some probability (it's a parameter) and otherwise two their offsprings are generated with crossover.

We randomly select two cutting points `i` and `j`. First offspring gets genes `i+1:j` of first parent and the rest missing genes it gets by sweeping second parent circularly starting from `j+1` and checking wether it has appeared in offspring. The second offspring is created by changing roles of first and second parent.

After a crossover is done, we perform mutation on new offsprings with some probability (it's a parameter).
It's done to prevent falling all solutions in the population into a local optimum. Mutation changes randomly the new offspring. For each gene sequence we randomly select 2 genes and switch them.

### End

After some number of iterations through **Packing** -> **Evaluation** -> **Selection** -> **Crossover**  return the best solution in last population.

### Parameters

1. **Population size** defines number of chromosomes in population.
2. **Elitism size** number of the best chromosomes (in context of fitness) to be copied to new population.
3. **Crossover probability** says how often will be crossover performed. If there is no crossover, offspring is exact copy of parents. If there is a crossover, offspring is made from parts of parents' chromosome.
4. **Mutation probability** says how often will be parts of chromosome mutated.


```
x_1
\mu
```

## BoxPacking: R package for solving 3D bin packing problem

The package is available on my [github](https://github.com/delta1epsilon/BoxPacking).

Below is the simple example of the package use.

``` r
library(BoxPacking)

# create containers
containers <- list()
n_containers <- 4

for (i in 1:n_containers) {
    containers <- c(containers,
                    Container(length = 2, height = 2, width = 2)
                    )
}


# create boxes
boxes <- list()
n_boxes <- 20

for (i in 1:n_boxes) {
    length <- sample(c(0.4, 0.5, 1), 1)
    height <- sample(c(0.4, 0.5, 1), 1)
    width <- sample(c(0.4, 0.5, 1), 1)

    boxes <- c(boxes,
               Box(length = length, height = height, width = width)
               )
}

# Box Packing
solution <-
    PerformBoxPacking(containers = containers,
                      boxes = boxes,
                      n_iter = 4,
                      population_size = 20,
                      elitism_size = 5,
                      crossover_prob = 0.5,
                      mutation_prob = 0.5,
                      verbose = TRUE,
                      plotSolution = TRUE
                      )
```

And here is how the package visualizes the packing solution.

![](http://delta1epsilon.github.io/assets/giphy.gif)
