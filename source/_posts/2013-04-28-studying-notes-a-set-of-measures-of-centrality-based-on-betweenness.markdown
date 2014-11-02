---
layout: post
title: "A Set of Measures of Centrality Based on Betweenness"
date: 2013-04-28 23:49
comments: true
categories: [Notes, SNA, Betweenness, Social, Studying]
keywords: Notes, SNA, Betweenness, Social, Studying,  中介中心性, 社会网络分析, 学习笔记
description: This paper by Freeman (1977) is an introductory and the most classic paper of betweenness centrality. Based on the betweenness centrality concept first introduced by Bavelas on 1948, the paper introduces a set of measures of centrality, including point centrality, scale free (relative) point centrality, and graph centrality. 
---

This post is a studying notes of paper "A Set of Measures of Centrality Based on Betweenness", the paper [\[pdf\]][pdf], wrote by Freeman in 1977, is an introductory and the most classic paper of [betweenness centrality][betweeness]. Based on the betweenness centrality concept first introduced by Bavelas in 1948, the paper introduces a set of measures of centrality, including point centrality, scale free (relative) point centrality, and graph centrality. 


####Betweenness and Point Centrality
 
The classical centrality measures of Bavelas (1950) etc. can not be used by unconnected networks, since they define the centrality of a point as the sum of the minimum distance between that point and all others, thus all distance sums are infinite in unconnected networks.  

In order to obtain a more satisfactory solution per centrality measurements, Freeman adopts the betweenness concept first introduced by  Bavelas, betweenness of a point is "the degree to which a point falls on the shortest path between others", and therefore has a potential per control of communication, persons in such central positions could influence the group by " withholding information (or) coloring, or distorting it in transmission".

####Measurement of Point Centrality
Consider a unordered pair of  points {$$p_i, p_j$$} $$(i \neq j)$$. Either $$p_i \text{ and } p_j$$ are unreachable from each other or  there are one or more paths between them. In the later case, each path has a length equals to the number of edges contained in it. Among the paths connecting pi and pj, one or more have the shortest length: the **geodesics**.

If $p_i$ and $p_j$ are unreachable from each other, then $$b_{ij}(p_k)$$, denotes betweenness centrality of point $$p_k$$ with respect to $$p_i$$ and $$p_j$$, is zero;

If $$p_i$$ and $$p_j$$ are _adjacent_, means that there is only one edge connecting them, since $$p_k$$ does not reside on the shortest paths of $$p_i$$ and $$p_j$$, $$b_{ij}(p_k)$$ also equals to zero;

If $$p_k$$ is on one or more shortest paths of $$p_i$$ and $$p_j$$, in which situations, $$p_k$$ has a proportional control of communication between $$p_i$$ and $$p_j$$, and intuitively the proportion is the percentage/extent to which $$p_k$$ is on the shortest paths, given the total shortest paths of $$p_i$$ and $$p_j$$ is $$g_{ij}$$ and the number of $$p_k$$ falls in the shortest paths is $$g_{ij}(p_k)$$, then we get $$b_{ij}(p_k) = g_{ij}(p_k)/g_{ij}$$. As the equation shows below:

$$b_{ij}(p_k) = \begin{cases} 
0 & p_i \mbox{ and } p_j \mbox{ are unreachable or adjacent} \\ 
g_{ij}(p_k)/g_{ij} & \text{otherwise} 
\end{cases}$$

To determine the overall centrality of $$p_k$$ in the graph, we need merely to sum the partial betweenness value of all unordered pair of points {$$p_i, p_j$$} in the graph:

$$\begin{align} C_B(p_k) = \sum\limits_{i}^n \sum\limits_{j}^n b_{ij}(p_k) \mbox{ where } i \lt j \mbox{ and } i,j \neq k  \end{align}$$


####Scale-free Point Centrality

One problem with the above point centrality is it not point independent/scale-free, it is related to how many points in the graph, thus comparing point centrality defined above with different graphs which may contain different amount of point is meaningless. A concreted example is pointed out that  6 betweenness value in 5 points graph compared with 6 betweenness value in 25 points graph, although they have the same betweenness centrality value but the influence of them is different.

In the later section Freeman introduced a relative point centrality, which is scale free to how many points in the graph, where point centrality $$C'_B(p_k)$$ is divided by the maximum centrality of their graph.
 
$$\begin{align} C'_B(p_k) = \dfrac{\sum\limits_{i}^n \sum\limits_{j}^n b_{ij}(p_k)} {\mbox{Max } C_B(p_k^*)} \end{align} $$

where $$n$$ is the number of points in the graph, and $$\begin{smallmatrix} \text{Max } C_B(p_k^*) \end{smallmatrix}$$ is only determined by n, which is $$\begin{smallmatrix} \frac{n \times (n-1)}{2} - (n-1) = \frac{n^2 - 3n + 2}{2} \end{smallmatrix}$$, therefore we obtain:

$$\begin{align} C'_B(p_k) = \dfrac{2 \times \sum\limits_{i}^n \sum\limits_{j}^n b_{ij}(p_k)}{n^2 - 3n + 2} \end{align} $$

To get more information of how $$\begin{smallmatrix} \text{Max } C_B(p_k^*) \end{smallmatrix}$$ is computed, please refer to the paper for details, you can imagine such a point that stands on the shortest path of all other pairs of {$$p_i, p_j$$}, where $$p_i$$ and $$p_j$$ have merely **one** shortest path, such a graph looks like a star graph, that one point in the central and all others surround it.


####Graph Centrality

Freeman defines the graph centrality as the average difference between the most central point and all others, which sounds like a little bit of **_information entropy_**, I think.

$$\begin{align}
C'_B = \dfrac{\sum\limits_{i}^n \left( C'_B(p_k^*) - C'_B(p_k) \right)}{n - 1} 
\end{align}$$

where $$\begin{smallmatrix} C'_B(p_k^*) \end{smallmatrix}$$ is the relative point centrality value of the most central point $$p_k^*$$ in the graph.

From the equation, we can see if all the points have the same centrality value then the graph centrality is 0; and if the graph is a star or wheel graph, then graph centrality is 1.


`---EOF---`

[pdf]: https://www.tribler.org/trac/raw-attachment/wiki/ReputationFunction/23.pdf
[betweeness]: http://en.wikipedia.org/wiki/Betweenness_centrality


