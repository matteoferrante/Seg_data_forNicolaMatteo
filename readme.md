## Prior based automatic segmentation of deep nuclei

In this repository a method for automatic segmentation on older patients deep nuclei using a prior from younger ones is described. The main notebook is called Segmentation-FC.ipynb and inherits functions from segmentation.py

For each target nucleus, the **center of gravity** of the prior mask is computed. A box is constructed with a specific **side** (usually 10-12 voxel) centered on this value, in order to have a **local evaluation**

Coordinates of this box, together with scaled image gray values are the features that we're intereset in.

A Local Fuzzy C Means algorithm is run with **n_cluster** (default=10) in order to find consistent region of images. 

The algorithm choose randomly **n_cluster** centroids and compute the euclidian distance of each point to the centroids, weighted with the probability of belonging to the right cluster. The objective is to minimize this cost function

${\displaystyle J(W,C)=\sum _{i=1}^{n}\sum _{j=1}^{c}w_{ij}^{m}\left\|\mathbf {x} _{i}-\mathbf {c} _{j}\right\|^{2}}$

${\displaystyle w_{ij}={\frac {1}{\sum _{k=1}^{c}\left({\frac {\left\|\mathbf {x} _{i}-\mathbf {c} _{j}\right\|}{\left\|\mathbf {x} _{i}-\mathbf {c} _{k}\right\|}}\right)^{\frac {2}{m-1}}}}}$

Each point will have **n_cluster** distances from each one of the centroids and it is associated the cluster of the closest centroid.
Now, for each cluster, the new centroids are computed as the mean coordinates of the points

${\displaystyle c_{k}={{\sum _{x}{w_{k}(x)}^{m}x} \over {\sum _{x}{w_{k}(x)}^{m}}},}$


and the process is repeated until convergence

When performing predictions, each point of the local box is associated with a probability of belonging to each cluster.
The cluster that contains the center of mass is selected as the target one, and probability of this cluster is increased by $\alpha P_{prior}$, then the argmax is taken to provide final mask.

A connected component analysis in 3D looking for 26 nearest neighbors is conducted to keep the largest connected components and avoid fragmented masks. The final mask is a boolean 3D box.

The box is placed in a 3D zeros array with the same shape of the original image

Masks for the younger subjects are used as priors to compute 12 potential mask candidates and they're finally average.

The output is a probabilistic mask together with boolean masks with threshold of greater than 0, 0.25, 0.50, 0.75.

The higher the threshold, the smaller the mask. From initial qualitative evaluation, seems that 0 and 0.25 generate mask of higher quality.