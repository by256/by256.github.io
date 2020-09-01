# Spectral Graph Convolutions -  Fourier Transforms (on Graphs) for the Birdwatcher

In the spirit of Francis Crick, who in trying to explain X-ray crystallography to a colleague, simplified the mathematical concepts such that he considered writing up his explanations under the title 'Fourier Transforms for the Birdwatcher' - this post will attempt to do the same with spectral graph convolutions.

### Convolution

Before we jump into spectral convolutions on graphs, we need to understand how convolution works in the spatial and frequency domains. On grid structured data such as 1D signals and 2D images, we can conveniently perform convolutions directly in the spatial domain. This is achieved by passing a filter $w$ over each pixel $h$ and computing the inner product between the neighbourhood of each pixel and the filter values.

$$w * h = \sum_{j \in \mathcal{N}_{i}} \langle w_{j}, h_{i-j}\rangle $$

where $\mathcal{N}_{i}$ is the neighbourhood of pixel $i$. This is possible because pixels exist in a grid with indices which define exactly where they exist on that grid. The vertices of a graph however are permutationally invariant and are not indexed in a way that encodes their structure. As a result, we can't perform convolution directly like this. This forces us into the Fourier domain. The convolution theorem states that the Fourier transform of the convolution of two functions is the pointwise product of their Fourier transforms.

$$w * h = \mathcal{F}^{-1}(\mathcal{F(w)*\mathcal{F}(h)})$$

Thus we can obtain the convolution by first multiplying our functions in the Fourier domain, and subsequently returning via the inverse Fourier transform. Ok, but if $h$ is a node feature vector on a graph, how do you take the Fourier Transform of nodes on a graph?

### Fourier Transform of a Graph

The Fourier transform of a real-valued function is obtained by multiplying by a set of Fourier basis vectors. Each of these basis vectors $w_k \in \mathbb{C}^N$ is a sinusoid with frequency $\frac{2\pi k}{N}$. Multiplying a function by the matrix containing the Fourier basis vectors can be viewed from the lens of template matching or correlation. Each inner product $\langle w_{k}, x\rangle$, measures the similarity between $x$ and the basis vector $w_k$ - how much does this function resemble a sinusoid of frequency $\frac{2\pi k}{N}$. If we compute these inner products for all Fourier basis functions, we obtain the frequency-space representation of $x$. So how do we do it for graphs? First, a definition that is key to understanding this...

#### Graph Laplacian

The (normalised) Laplacian of a graph is defined as 

$$\Delta = I - D^{-\frac{1}{2}}AD^{-\frac{1}{2}}$$

where A is an adjacency matrix, such that $A_{ij} = 1$ if node $i$ is connected to node $j$, and 0 otherwise; and D is the degree matrix - a diagonal matrix where $D_{ii}$ is the number of nodes connected to node $i$ and $D_{ij} = 0 \:\: \forall \:\: (i, j), i \neq j$. Intuitively, the Laplacian is a measure of local smoothness in the graph. Multiplying a node signal $h_i$ by $\Delta$ represents how much a node differs from its neighbours. 

$$\Delta h_i = h_i - \frac{1}{d_i}\sum_{j \in \mathcal{N}_i}A_{ij}h_j$$

If the signal is "smooth", or the neighbourhood of node $h_i$ does not vary much from itself, this difference will be close to zero. However, if there is high variance in $\mathcal{N}_i$, the difference will be large. Thus, the Laplacian is simply a measure of the smoothness of a function on a graph. What's the point of the Laplacian? **The eigenvectors of the Laplacian matrix of a graph are the Fourier basis vectors that we will use to take the Fourier transform of signals defined on that graph.** By taking the eigendecomposition of $\Delta$, we obtain the eigenvectors $\Phi$ and hence Fourier basis vectors.

$$\Delta = \Phi^T \Lambda \Phi$$

#### Spectral Graph Convolutions

Now we can take our graph signal $h$ into Fourier space as we would with a real-valued function:

$$\hat{h} = \mathcal{F}(h) = \Phi^Th$$

We do the same with the weights of the convolutional filter:

$$\hat{w} = \mathcal{F}(w) = \Phi^Tw$$

Just as we mentioned, to perform convolution in the frequency domain we take the pointwise product of $\hat{h}$ and $\hat{w}$. Finally, we can obtain our desired convolution by inverse Fourier transforming back to the spatial domain.

$$w * h = \Phi(\hat{w} \odot \hat{h})$$

where multiplication by $\Phi$ constitutes the inverse Fourier transform. 

*Note: this can actually be simplified further with a bit of algebra. I'm going to leave it there however, since I feel that this illustrates what we're trying to achieve in a way that's super intuitive if you know a bit of signal processing/Fourier analysis.*

### Conclusion

That's it. The key here is that we're able to obtain the Fourier basis vectors from the eigendecomposition of the graph Laplacian. By taking advantage of the convolution theorem, we can perform convolution on signals on graphs by taking a short detour into the frequency domain to multiply, and returning via the inverse Fourier transform to obtain the result we're after.

A quick warning though. While this would actually be fairly straightforward to implement, in practice, this method is quite inefficient. Plenty of tricks have been discovered recently to increase the efficiency of spectral graph convs and it should be possible to uncover them with a quick search of the literature.

Here are a few resources that were useful in understanding spectral graph convolutions and in writing this:

[1] J. Bruna, W. Zaremba, A. Szlam, Y. LeCun. Spectral Networks and Deep Locally Connected Networks on Graphs. *arXiv:1312.6203* (2013)\
[2] M. Henaff, J. Bruna, Y. LeCun. Deep Convolutional Networks on Graph-Structured Data. *arXiv:1506.05163*, (2015)\
[3] M. Defferrard, X. Bresson, P. Vandergheynst. Convolutional Neural Networks on Graphs with Fast Localized Spectral Filtering. *Advances in Neural Information Processing Systems* (2016)\
[4] F. Chung. Spectral Graph Theory. *American Mathematical Society* (1997)\
[5] X. Bresson. Lecture: Graph Convolutional Networks. *https://www.youtube.com/watch?v=Iiv9R6BjxHM* (2020)\