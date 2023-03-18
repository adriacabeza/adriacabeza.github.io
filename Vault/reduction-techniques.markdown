
The purpose of this post is to give a simple explanation about two of the most common dimensionality reduction methods: PCA and t-SNE.

Dimensionality reduction is very important in data science when you are working with a large number of features because they allow us to work with much smaller and manageable datasets which is easier and faster.

# PCA

Principal Component Analysis (PCA) is actually a widely known method in data science. It is based on transforming a large set of variables into a smaller one trying to keep as much information as possible. Its basic idea is trying to find better axis to project the data in a more meaningful way (keeping the information).

## Standarization
Since we might be working with different features, we first need to standarize them. There might exist some features ranging from 0 to 1000 and other ones from 0 to 1 and we do not want that difference of scale. There shouldn't be any feature with more/less importance. This is done by taking each value, substracting the mean and dividing by the standard deviation of its feature.

<div  align="center">
  $z=\frac{x-\mu}{\sigma}$
</div>

## Covariance Matrix
The next step consists in creating a matrix that contains how much two variables means change with respect to each other. This one is very important because it tells us if there is any relationship between variables. Computing the covariance matrix is pretty easy, you only have to do the following:
<div  align="center">
	$cov_{x,y}=\frac{\sum_{i=1}^{N}(x_{i}-\bar{x})(y_{i}-\bar{y})}{N-1}$
</div>

## Eigendecomposition
The final process is the eigendecomposition of the covariance matrix. Eigendecomposition is the method to decompose a square matrix into its eigenvalues and eigenvectors. Eigenvectors and eigenvalues will be key to minimize redudancy and maximize variance to express better the data.

<br>
For a matrix $A$, if
<div align="center">
$Av=\lambda v$
</div>
then $v$ is the eigenvector and $\lambda$ is its corresponding eigenvalue. That is, if the matrix $A$ is multiplied by a vector and the result is a scaled version of the same vector, then that vector is an eigenvector of $A$ and the scaling factor is the eigenvalue. If you wanna know more about them I highly reccomend [this 3BlueBrown video](https://www.youtube.com/watch?v=PFDu9oVAE-g).

Once we have the eigenvectors, we must normalize each of the orthogonal eigenvectors to turn them into unit vectors. Once this is done, each of the mutually orthogonal, unit eigenvectors can be interpreted as an axis of the data. These new axis are the projections allow us to reduce our dataset and maintain the information. Why? Well, these projections are selected in such a way that they explain better the data than the ones that existed in the original one. This means that we might be able to remove dimensions because a smaller number of dimensions was enough to explain the data.

<div align="center">
<img class="ui image" width="600" src="https://www.analyticsvidhya.com/wp-content/uploads/2016/03/2-1-e1458494877196.png">
</div>
<br>

When choosing the components, we take the eigenvectors in a decreasing order according to its $\lambda$ because the bigger $\lambda$ is, the better because it is explaining more variance.

## The code
If you wanna try it out, it is pretty goddamn easy to do it thanks to sklearn. 

```python
import numpy as np
import matplotlib.pyplot as plt

from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler


def PCA(data):
    x = StandardScaler().fit_transform(data)
    pca = PCA(n_components=2) 
    # you can also set a float value  i.e. 0.95
    # which will choose automatically the components
    # to explain the 95% of the variance of the original data
    principalComponents = pca.fit_transform(x)
```

Actually, you can even check how much of the total variance is explained by each component. That is great if you want to see how many components are enough for your reduced dataset:

```python
    var=np.cumsum(np.round(pca.explained_variance_ratio_, decimals=3)*100)
    plt.ylabel('% Variance Explained')
    plt.xlabel('# of Features')
    plt.title('PCA Analysis')
    plt.ylim(30,100.5)
    plt.grid()
    plt.plot(var)
    plt.show()
```

# t-SNE

t-distributed Stochastic Neighbor Embedding (t-SNE) was developed by Laurens van der Maaten and Geoffrey Hinton in 2008. It is a nonlinear dimensionality reduction method which means that it allows us to separate data that cannot be separated by any straight line (unlike PCA).

Basically, the t-SNE approach tries to maintain the local structure: it tries to map points in high dimensional space to a lower dimension so that the distances between the points remains almost the same.

<div align="center">
  <img src="https://miro.medium.com/max/600/0*4Uv73sfML2MrEtoK.png"> <br> 
  Linearly nonseparable data. Source: <a href="https://distill.pub/2016/misread-tsne/">Distill</a>
</div>
<br>

The first part of the algorithm consists in creating a probability distribution that represents similarities between neighbors. Imagine two datapoints $x_i$ and $x_j$; we define the similarity between those two points as the conditional probability, $p_{i\|j}$, that $x_i$ would pick $x_j$ as its neighbor if neighbors were picked in proportion to their probability density under a specific Gaussian distribution. 

<div align="center">
  <img width="80%" src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c8/Gaussian_distribution.svg/1200px-Gaussian_distribution.svg.png"> <br> 
Gaussian distribution. Source: Wikipedia
</div>
<br>


This Gaussian is centered at $x_i$ and has a variance, $\sigma_i$ adapted to the density of the data: smaller values of $\sigma_i$ are used in denser parts of the data space. The way $\sigma_i$ is set varies depending on a tuneable parameter called perplexity which tells us how to balance attention between local and global aspects. The parameter is, in a sense, a guess about the number of close neighbors each point has. The original paper says, “The performance of SNE is fairly robust to changes in the perplexity, and typical values are between 5 and 50.”

<div align="center" width="100%">
$p_{j \mid i}=\frac{\exp \left(-\left\|\mathbf{x}_{i}-\mathbf{x}_{j}\right\|^{2} / 2 \sigma_{i}^{2}\right)}{\sum_{k \neq i} \exp \left(-\left\|\mathbf{x}_{i}-\mathbf{x}_{k}\right\|^{2} / 2 \sigma_{i}^{2}\right)}$
</div>
<br>

Note that the Gaussian kernel uses the Euclidean distance $\|x_{i}-x_{j}\|$. This means that it is affected by the **curse of dimensionality** which means that distances become less meaningful when dealing with high dimensional data. Some methods have been proposed to deal alleviate this like [adjust distances with a power transform](https://doi.org/10.1007%2F978-3-319-68474-1_13).

The second part is trying to recreate a low-dimensional space that follows that distribution as best as possible. To this end, it measures similarities $q_{ij}$ between two points in the low-dimensional space $y_i$, $y_j$ using a very similar approach. Specifically, if $i \neq j$:

<div align="center" width="100%">
$q_{i j}=\frac{(1+\|y_{i}-y_{j}\|^{2})^{-1}}{\sum_{k \neq i}(1+\|y_{i}-y_{k}\|^{2})^{-1}}$
</div>
<br>

Here, a Student t-distribution is used to measure similarities between low-dimensional points. This allows us to model far apart dissimilar objects.

And how do we compute the location of the points $y_{i}$? We determine them by minimizing the Kullback-Leibler divergence of the distributions $P$ and $Q$. This minimization is performed by using gradient descent.

It is important to note that gradient descent is an iterative process so we cannot apply the same transformation to another dataset. PCA instead, uses the covariance matrix to reduce the data, so we can perform the same transformations to a new set of data (i.e. the test split). t-SNE is mostly used to understand high-dimensional data and project it into a low-dimensional space (like 2D or 3D to plot it).

<div align="center">
  <img src="https://nlml.github.io/images/tsne/tsne-mnist.png" width="80%"> <br> 
  Result of applying t-SNE to a dataset of MNIST features. <a href="https://nlml.github.io/">Source</a>
</div>

# Sources

- [How to USE t-SNE Effectively](https://distill.pub/2016/misread-tsne/)
- [Original Paper](http://www.jmlr.org/papers/volume9/vandermaaten08a/vandermaaten08a.pdf)
