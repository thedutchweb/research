Canonical Camera Perspective
===================

INTRODUCTION
-------------
Determining canonical camera perspectives can serve Borges in two ways. First it can greatly improve the thumbnail images given back in the search results to show the most pretty angle of the image, and second it can be used to do help out with object classification and matching.

#### Goal
One of our goals for this quarter is to perform categorisation of 3D objects by taking one or more 2D images of this object, and feeding those to a neural network that has been trained to categorise 2D images.

In order to obtain the best possible results, these 2D images should be taken from a camera perspective that maximizes the amount of information that can be gleaned from them. Such a camera perspective is known as a canonical perspective.

#### Deliverable
The goal of this issue is to research how such a canonical camera perspective should be implemented. The deliverable is a short report detailing what approach we should take and why.

OPTIONS FOR IMPLEMENTATION
-------------

As far as implementation goes there are three possible options.

\# | Option     | Time estimation (rough) | Est. Effectivity | Additional remarks
--| -------- | --- | ---
1 | LibIGL Canonical Quaternions    | Between one and two weeks | Sufficient | No pointcloud analysis
2 | Simple canonical views | Between two and five weeks | Good |
3 | Segmentation analysis    | more than six weeks | ? | Insufficient research

We'll discuss the options in a little more detail down here and finish with a conclusion in the last section.

### 1. Standard LibIGL canonical orientations

In libIGL they use a set of standard quaternions based on the orientation of the XY plane. Their assumption is that most 3d objects are oriented correctly to start with.

This would deliver the following algorithm for implementation:

1. Determine all canonical quaternions \[ \mathbf{H_c} \subset \mathbf{H} \] by taking the set of orthogonals on the XY plane, the XZ plane and the YZ plane.
2. Determine all possible quaterninons $\mathbf{H_{h}}$ which have a 45 degree angle with an element in $\mathbf{H_c}$.
3. Generate images for each $h \in \mathbf{H_c} \cup \mathbf{H_h}$, and tag those images using your favourite image tagging implementation.
4. Select the image with the highest likelihood, for it's tags as the best camera perspective.

The advantage of this method is that it's fairly easy to implement and that it can serve as a base implementation for **Option 2**.

### 2. Simple canonical views
In the paper **Simple canonical views** [1], by P. Hall and M. Owen, a method is described for determining canonical views based on a set of pictures or a point cloud. Each object is supposed to have approximately 2500 data points $x_i \in \mathbf{X}$ which represent either images or point cloud information.

From these 2500 images an eigenmodel is calculated: $$\Omega = (n,\mu,\mathbf{U}, \Lambda)$$ where:
$$
\begin{align*}
n &= \# \mathbf{X} \\
\mu &= \frac{1}{n} \sum_{i}^{n} x_i \\
\mathbf{U} &= corresponding \space eigenvector \space matrix \\
\mathbf{\Lambda} &= corresponding \space diagonal \space eigenvalue \space matrix
\end{align*}
$$
The canonical views algorithm will roughly work as follows:

1. Determine a new (compressed) eigenmodel $\mathbf{U'} \subseteq \mathbf{U}$ and $\Lambda' \subseteq \Lambda$ with $\# \mathbf{U'} \leq 20$ using an incremental approach as described in [3].
2. Define the Mahalanobis distance function by: $$ m(x_i) = (x_i-\mu)^T\mathbf{U'}^T {\Lambda'}^{-1}\mathbf{U'} (x_i-\mu)$$
And choose $x_k$ with $\max_i m(x_i)$ and corresponding $f_k$ as the camera orientation.
3. Let $X_{f_k}$ be the set of almost orthogonal images to $f_k$, and do '2' to determine $x_l$ and corresponding $f_l$
4. Let $X_{f_k, f_l}$ be the set of almost orthogonal images to $f_k$ and $f_l$, and do '2' to determine $x_m$ and corresponding $f_m$
5. Take the exact opposite of the three orientations as your 'front', 'top' and 'left' view.
6.  Convert the orientations to three quaternions and call the set of those quaternions $\mathbf{H_c}$.
7.  Determine all possible quaterninons $\mathbf{H_{h}}$ which have a 45 degree angle with an element in $\mathbf{H_c}$.
8. Generate images for each $h \in \mathbf{H_c} \cup \mathbf{H_h}$, and tag those images using your favourite image tagging implementation.
9. Select the image with the highest likelihood, for it's tags as the best camera perspective.

The final steps are similar to **Option 1** which supports the serving as a base implementation for this option.

### 3. Canonical view determined by object segmentation
Too little research has been done on how this should be implemented. It seemed like a lot of work but I could be mistaken and therefore I'd still like to mention it here as a possible option.

CONCLUSION
-------------
My suggestion would be to start with implementing **Option 1** first and in time look at implementing **Option 2** on top of that.

The libIGL canonical views method sets up the same framework as that for simple canonical views, and if we later on need that approach it would be easy to build this on top of **Option 2**.

Furthermore it might be worth it to expand the document a little bit by researching **Option 3** later on, but for now **Option 1** and possibly adding **Option 2** would be sufficient to reach our targeted goal.


REFERENCES
-------------

[1] P. Hall and M. Owen. Simple canonical views. *Proc. BMVC, pages 839–848*, 2005

[2] P.Hall, D.Marshall, and R.Martin. Merging and splitting eigenspaces. *IEEETrans- actions on Pattern Analysis and Machine Intelligence,* 22(9):1042–1049, September 2000.

[3] P.Hall, D.Marshall, and R.Martin. Incremental Eigenanalysis for Classification. *Proceedings of British machine vision conference* (pp. 286–295), 1998.
