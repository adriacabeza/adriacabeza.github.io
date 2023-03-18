Since I discovered the existance of spectral graph theory (during my master's course Algorithmic Methods of Data Mining at Aalto), I discovered that I didn't know shit about graphs. I was completely blown away with the properties of the spectrum of a graph. I could not believe them: they are not straight-forward at all. So an effort was made in order to understand them and go throught some of the formal proofs. This post is a introduction of some of the cool things that I learned. Maybe I can get you as interested as I was at that time.

The idea of spectral graph theory is to look at graph with a linear algebra lens and try to identify connections between eigenvalues and eigenvectors of the matrix and structural properties of the graph.

> I will suppose that you already know what eigenvalues and eigenvectors are.

## Cool properties

By choosing the proper matrix (a real and symmetric $n\times n$ matrix $A$ with graph information), we can get very cool properties appear from those eigenvalues. For example, if we choose the Laplacian matrix:
```
Laplacian = DiagonalMatrix - AdjacencyMatrix
```

We can order its eigenvalues $\lambda$ and get the following properties:
<div align="center">
$\lambda_2 = 0$ if $G$ is disconnected.<br>
...<br>
$\lambda_k = 0$ if $G$ has at least $k$ connected components.<br>
$\lambda_n=2$ if G has a bipartite connected component<br>
</div> 

### Graph clustering

Spectral clustering methods use the spectrum of a graph to perform dimensionality reduction before clustering in fewer dimensions. The idea is based on computing the eigenvectors of the second-smallest eigenvalue of the normalized Laplacian or some eigenvector of some other matrix representing the graph structure. The resulting eigenvector is used as a vertex embedding of the graph to determine the clustering. 

<img src="http://lordsantanna.com/img/spectral.png">

For example, if a graph G is a collection of k disjoing cliques, the normalized Laplacian is a block-diagonal matrix that has eigenvalue zero with multiplicity $k$ and the corresponding eigenvectors serve as indicator of the membership of each clique: 

The eigenvector $v_i$ has a different value or a larger magnitude for the vertices that are inside the clique $i$ than the other vertices. Thus, there is an underlying structure that can be seen using the eigenvectors of the Laplacian. Moreover, if we introduce edges between the cliques we will see that $k-1$ of the $k$ eigenvalues that were zero will become slightly larger than zero so it is a robust method.

If you want to know more about it, check the work we did my fellow mate Alvaro Órgaz and I during the Aalto's course: [It's an experimental study about graph clustering into communities for large-scale networks.](https://github.com/adriacabeza/GraphClustering/blob/master/report/report.pdf). 

## Sources
- [Spectral Theory, CS Washington](https://courses.cs.washington.edu/courses/cse521/16sp/521-lecture-12.pdf)
-  [Graph Communities](https://chih-ling-hsu.github.io/2020/05/25/Graph-Communities)
-  Mining of massive datasets, Cambridge University Press, 2012