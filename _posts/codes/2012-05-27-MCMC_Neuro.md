---
priority: 0.13
title: "MCMCneuro"
excerpt: "Data driven graph model for neuronal interactions using Bayesian statistics and Markov Chain Monte Carlo"
categories: coding
background-image: MCMCneuro_back.jpg
options: minihead, notitle
code-logo: MCMCneuro.jpg
code-repo: github
code-url: https://github.com/brandimarte/MCMCneuro
code-author: <u>Pedro Brandimarte</u>
code-lictype: open
lang:
  - C
  - shell script
  - R
tags:
  - neuronal interactions
  - graph model
  - data structure
  - skip-list
  - Bayesian statistics
  - Markov Chain Monte Carlo
---

#### About

The goal of **MCMCneuro** is to find the most representative graph model for neuronal interactions via Markov Chain Monte Carlo.

___

#### Description

This is a *very* short description of the implementation. A more detailed description of the problem and the implemented algorithms is available (in Portuguese) here:
&nbsp;
<a target="_blank" href="{{ site.baseurl }}/documents/2012.06.18.MCMCneuro.pdf">
  <span class="icon fa-file-pdf-o fa-lg style4"></span>
</a>

##### graphPenalty

Computes a Markov Chain Monte Carlo on graphs representing neuronal connectivity to estimate for each mouse the graph that best represents the observed data in the first and third parts of the experiment (before and after the mouse got in touch with geometric objects) considering different values of the penalty constant considering 3 different methods for computing the posterior probability:

 1. &#139;Xi&#124;Xj&#155;=1 if Xi=1 and Xj=1
 2. &#139;Xi&#124;Xj&#155;=1 if Xi=Xj or &#139;Xi&#124;Xj&#155;=0 (if Xi&ne;Xj)
 3. &#139;Xi&#124;Xj&#155;=1 if Xi=Xj or &#139;Xi&#124;Xj&#155;=-1 (if Xi&ne;Xj)

The output files are sent to `out/outPenalty` folder.

For example:

 - the file `outputM6HPp1Met1.dat` contains general information about the run for mouse 6 (`M6`) in the hippocampus region (`HP`) in the first part of the experiment (`p1`) and where the posterior probability were computed with method 1 (`Met1`).
 - the file `penal1M6HPp1Met1.dat` contains the penalty constant versus the logarithm of the non-normalized posterior probability (with penalty included).
 - the file `penal2M6HPp1Met1.dat` contains the penalty constant versus the logarithm of the non-normalized posterior probability (without penalty).
 - the file `penal3M6HPp1Met1.dat` contains the penalty constant versus the empirical probability (obtained from the Monte Carlo).

##### bestGraph

For a given penalty constant, chosen based on the previous executable, it computes a Markov Chain Monte Carlo on graphs representing neuronal connectivity to estimate the most representative graph for each set of observed neurons.

The output files are sent to `out/outBestGraph` folder.

For example:

 - the file `outputHPMet1.dat` contains general information about the run for all mouses where the hippocampus region (`HP`) where observed and where the posterior probability were computed with method 1 (`Met1`).
 - the file `adjM6HPp1Met1.dat` is the adjacent matrix of the most probable graph of mouse 6 (`M6`), in the hippocampus region (`HP`), in the first part of the experiment (`p1`) and where the posterior probability was computed with method 1 (`Met1`).

___

#### Scripts

 1. **linux.sh** - user friendly script, for linux users, to compile and run the program.
 2. **mac.sh** - user friendly script, for mac users, to compile and run the program.
 3. **jobPen.x** - example of Portable Batch System (PBS) to run the `graphPenalty` executable in a cluster.
 4. **jobBGr.x** - example of Portable Batch System (PBS) to run the executable `bestGraph` in a cluster.
 5. **rotula.sh** - this script verifies the experimental data at `data` folder and creates files in `path` folder describing them.
 For example, the file `HPmousesP1.dat` contains information about the observed data from hippocampus (`HP`) in the first part of the experiment (`p1` - before the mouses got in touch with geometric objects).
 The line `5 13 1.380850 3610` indicates that from mouse 5 was observed 13 neurons at hippocampus where the (maximum) start of observation happened at 1.380850 seconds and where the minimum time where it got in touch with the geometric objects happened at 3610 seconds.
 This line is followed by 13 lines containing the name and the path with the spikes data of each neuron.
 6. **graphs.r** - script in `R` to draw the graphs from the adjacent matrix.
 It is not automated, so the user has to edit it.

___

#### Source code
 
 1. **graphPenalty.c** - client program that estimates for each mouse the graph that best represents the observed data, in the first and third parts of the experiment, considering different values for the penalty constant and considering the 3 different methods for computing the posterior probability (see executable `graphPenalty` above).
 2. **bestGraph.c** - client program that computes a Markov Chain Monte Carlo on graphs representing neuronal connectivity to estimate the most representative graph for each set of observed neurons, given a penalty constant (chosen form a previous analysis with `graphPenalty` program) and given a specific method for computing the posterior probability (see executable `bestGraph` above).
 3. **Neuro.h** - interface for Markov Chain Monte Carlo on graphs representing neuronal connectivity.
 4. **Neuro.c** - Implementation of Markov Chain Monte Carlo on graphs representing neuronal connectivity. Given the experimental data, it generates a Markov Chain, on an undirected graph space, whose limit distribution is given by the posterior probability `P(g|X)`.
 This is done via Monte Carlo method with Metropolis algorithm.
 5. **ST.c** - abstract data type implementation for skip list symbol-table whose items have a counter that is incremented each time the item is searched (to avoid the need of storing each generated item, i.e., only distinct items are stored).
 6. **ST.h** - abstract data type interface for symbol-table whose items have a counter that is incremented each time the item is searched.
 7. **Item.c** - abstract data type implementation for graph items (i.e. strings containing `0`s and `1`s).
 8. **Item.h** - abstract data type interface.
 9. **Utils.c** - implementation of alternative versions of well known functions from `stdio.h` and `stdlib.h` to avoid code repetition.
 10. **Utils.h** - interface for alternative versions of well known functions from `stdio.h` and `stdlib.h` to avoid code repetition.
 11. **Makefile** - this file is read by `make` program for compiling all the files (or the changed) of the program.

___

#### Running

 - *linux*: open a terminal and go to the program main directory.
 Then run `linux.sh` script (`./linux.sh`) and follow the instructions.
 - *mac*: open a terminal and go to the program main directory.
 Then run `mac.sh` script (`./mac.sh`) and follow the instructions.
 - *clusters with PBS*: edit the scripts `jobPen.x` and `jobBGr.x` according to the machine specifications and submit (`qsub jobPen.x`).

___

#### Dependencies

 - **gcc** (tested with versions from 4.1 to 4.6)
 - **GNU Make** (tested with version 3.81)
 - to run the script `graph.r` it is necessary to have **gRbase** library installed

___

#### Acknowledgments

PBM thanks Prof. Antonio Galves for introducing the problem and providing the experimental data, the students Aline Duarte de Oliveira and Guilherme Ost de Aguiar for the discussions, and Prof. Yoshiharu Kohayakawa for fundamental aids and hints on the algorithm implementation.
