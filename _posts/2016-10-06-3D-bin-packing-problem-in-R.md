---
comments: true
title:  "3D bin packing problem in R"
date:   2016-10-06 15:04:23
categories: [R, 3D bin packing problem, Genetic Algorithm]
tags: [R, 3D bin packing problem, Genetic Algorithm]
---
In this post we'll go through algorithm for solving 3D bin packing problem.  

First of all, let's define what does "3D bin packing problem" (3DBPP) stand for. In 3DBPP rectangular boxes must be packed into rectangular containers (bins) in a way that maximizes containers utilization ratio.

There are a lot of variations of the problem and a lot of approaches for solving them. However, we are going to solve 3DBPP where containers are of various sizes and different orientations of boxes are considered.

The algorithm we are going to study is called ["A Genetic algorithm for the three-dimensional bin packing problem with heterogeneous bins"](https://www.researchgate.net/publication/273121476_A_genetic_algorithm_for_the_three-dimensional_bin_packing_problem_with_heterogeneous_bins) developed by Xueping Li, Zhaoxia Zhao and Kaike Zhang.

It's based on [Genetic Algorithm](http://www.obitko.com/tutorials/genetic-algorithms/index.php) and specific packing procedure.

#### Outline of the algorithm

##### Initialization

We start with initializing a population of `n` chromosomes (suitable solutions for the problem). A chromosome consists of 2 parts: box  packing sequence (BPS) and container loading sequence (CLS). BPS and CLS are permutations of `{1,...,m}` and `{1,...,N}` respectively where `m`, `N` - number of boxes and containers respectively. So, BPS is an  order in which the boxes are packed and CLS is an order in which the containers are loaded.

4 special chromosomes in a population are created by sorting BPS in descending order according to boxes volume, length, width and height. The rest chromosomes are generated randomly.

##### Packing

In next step each chromosome is being mapped into a packing solution. A concept of empty maximal spaces (EMS) is used to represent free space in containers. Essentially, EMS is a list of largest empty cubic space available for packing not contained in any other EMS. EMS are represented with their maximum and minimum coordinates. EMS is displayed in picture below.

![](http://delta1epsilon.github.io/assets/ems_plot.png)

[Figure source](https://www.researchgate.net/publication/273121476_A_genetic_algorithm_for_the_three-dimensional_bin_packing_problem_with_heterogeneous_bins)

Box packing procedure is based on defined priority of EMS and specific placement selection. We prefer EMS 1 to EMS 2 if minimum coordinates of 1 are smaller than minimum coordinates of EMS 2. Once we have chosen EMS, we have to choose box orientation. When a box has several possible placements in one EMS, than the one with smallest margin is selected.  

##### Evaluation

After a packing procedure for current population was done, we evaluate a fitness of each chromosome using measure of free space in the containers.

##### Selection

Now we are done with current population and we have to create a new population by performing **Selection** and **Crossover** on existing population. `E` chromosomes with the best fitness within the population are selected directly to next population (so the best solution found can survive to end of run). And the rest are selected for tournament where we randomly select 2 chromosomes and the better one (better fitness) goes to mating pool as parent (in hope that the better parents will produce better offspring).   

##### Crossover and mutation

Now it's time for crossover. Crossover selects genes from parent chromosomes and creates a new offspring. Crossover is made in hope that new chromosomes will have good parts of old chromosomes and maybe the new chromosomes will be better. However it is good to leave some part of population survive to next generation.

Each pair of chromosomes in mating pool goes to the next population with some probability (it's a parameter) and otherwise 2 their offsprings are generated with crossover. We randomly select 2 cutting points `i` and `j`. First offspring gets genes `i+1:j` of first parent and the rest missing genes it gets by sweeping second parent circularly starting from `j+1` and checking wether it has appeared in offspring. The second offspring is created by changing roles of first and second parent.     

After a crossover is done, we perform mutation on new offsprings with some probability (it's a parameter). It's done to prevent falling all solutions in population into a local optimum. Mutation changes randomly the new offspring. For each gene sequence we randomly select 2 genes and switch them.

##### End

After some number of iterations though **Packing** -> **Evaluation** -> **Selection** -> **Crossover**  return the best solution in last population.

##### Parameters

1. **Population size** defines number of chromosomes in population.
2. **Elitism size** number of the best chromosomes (in context of fitness) to be copied to new population.
3. **Crossover probability** says how often will be crossover performed. If there is no crossover, offspring is exact copy of parents. If there is a crossover, offspring is made from parts of parents' chromosome.
4. **Mutation probability** says how often will be parts of chromosome mutated.


#### R package

I developed R package for solving 3D bin packing problem. It's available on my [github](https://github.com/delta1epsilon/BoxPacking).
