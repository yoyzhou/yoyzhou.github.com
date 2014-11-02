---
layout: post
title: "A Kinda Betweenness Centrality Algorithm"
date: 2013-04-29 15:06
comments: true
categories: [Algorithm, Betweenness, Centrality, SNA]
keywords: Algorithm, Betweenness, Centrality, SNA, Social Network Analysis, Graph 
description: In this post I'd like to demonstrate a algorithm per computing the betweenness centrality which I have introduced in the previous post, for more details of what's Betweenness Centrality, and its measurements, please refer to A Set of Measures of Centrality Based on Betweenness. 
---

In this post I'd like to demonstrate a algorithm per computing the betweenness centrality which I have introduced in the previous post, for more details of what is Betweenness Centrality, and its measurements, please refer to my post [A Set of Measures of Centrality Based on Betweenness][betweenness_post]. 

####Definitions and notations

 Before getting down to the algorithm, first we briefly go through the definitions and notations.

 **_Betweenness Centrality_** is "the degree to which a point falls on the shortest path between others", it's the core of the algorithm. Although computing [shortest path][shortest_path] between point pair is essential to our problem, shortest path algorithm will not be the focus of this post. Readers interested in shortest path algorithm please refer to the shortest path on Wikipedia page [here][shortest_path], kindly note that even if you have no knowledge of shortest path algorithm, it will not affect you reading this post.

In the next section, we use graph-theoretic terminology neutral to interpretation of notations.

Given a _directed graph_ $$\begin{smallmatrix} G = \left( V, E \right) \end{smallmatrix}$$, consists of a set of $V$ of vertices and a set of $E \subseteq V \times V$ of _derected_ edges.

Denote $\sigma(s, t)$ to the number of _shortest_ $(s, t)$-paths, some times called geodesics, and let $$\sigma(s,t \vert v)$$ be the number of shortest path which passing through certain vertex $v$ other than $s, t$, then the _betweenness centrality_ $C_B(v)$ of vertex $v \in V$ is defined to be:

$$\begin{align}  C_B(v) = \sum\limits_{s,t \in V} \frac{\sigma(s,t \vert v)}{\sigma(s, t)} \end{align}$$ 


####The Algorithm
As quoted from "On variants of shortest-path betweenness centrality and their computation":

> Efficient computation of betweenness is based on the fact that the cubic number of _pair-wise dependencies $$\delta(s,t \vert v) = \sigma(s, t \vert v)/ \sigma(s, t)$$_ can be aggregated without computing all of them explicitly.

Defining one-side dependencies:

$$\begin{align} \delta(s \vert v) = \sum\limits_{t \in V} \delta(s, t \vert v)\end{align}$$,

per $$\forall s, v \in V$$ we can exploit that

$$\begin{align} \delta(s \vert v) = \sum\limits_{w: (v, w) \in E  \ and \\\ dist(s, w) = dist(s, v) + 1} \left[ \frac{\sigma(s,v)}{\sigma(s, w)}
\times \left( 1 + \delta(s \vert w) \right) \right]
\end{align}$$

where $dist(s, t)$ denotes the minimum path length of point pair $(s, t)$.

The algorithm asserts that the dependency of a vertex $s$ on some $v$ can be compiled from dependencies on vertices **_one edge farther away_**. 

Since $w$ is connected to $v$ and **one edge farther away** (which determined by $$dist(s, w) = dist(s, v) + 1$$) from $v$, so point $v$ is on the shortest paths of $(s, w)$ and the proportion of betweenness value that $v$ gains from edge $(v, w)$ equals to $\frac{\sigma(s,v)}{\sigma(s, w)} \times 1$; the other part of betweenness gain is from point $w$, that $$\frac{\sigma(s,v)}{\sigma(s, w)} \times \delta(s \vert w)$$, thus sum of the above two parts of betweenness gain is what exactly the equation demonstrates.


####References

\[1\] Linton C. Freeman. A set of measures of centrality based on betweenness. Sociometry, 40(1):35–41, March 1977.

\[2\] Ulrik Brandes. On variants of shortest-path betweenness centrality and their generic computation. Social Networks, 30(2):136–145, May 2008.

`---EOF---`

[shortest_path]:http://en.wikipedia.org/wiki/Shortest_path_problem
[betweenness_post]:/blog/2013/04/28/studying-notes-a-set-of-measures-of-centrality-based-on-betweenness/
