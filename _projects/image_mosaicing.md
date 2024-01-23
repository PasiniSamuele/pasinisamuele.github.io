---
layout: distill
title: Image Mosaicing
description: RGB Image Mosaicing via Multi-graph synchronization
importance: 1
related_publications: false
bibliography: 2022-10-18-mosaicing.bib

---

Image mosaicing is an effective means of constructing a single seamless image by aligning multiple partially overlapped images.
Over the years several different approaches have been proposed to solve the various steps of the mosaicing pipeline such as alignment and compositing.
My project aims to propose a solution for Global Homography Estimation, a fundamental step of image alignment, based on the multi-graph synchronization algorithm MULTISYNC, <b>Dal Cin et al.</b> <d-cite key="DalCin2021"></d-cite>

I presented this project with Dr. Francesco Azzoni and Dr. Corrado Fasana to the Computer Vision exam in Politecnico di Milano.

I already analyzed the basic idea behind MULTISYNC approach in <a href = "https://pasinisamuele.github.io/blog/2022/mosaicing/"> this blog post </a> which I suggest you to read before continuing, since I will start from it and I will go more in depth to analyze our project and our experiments.

To summarize, MULTISYNC is based on multi-graph synchronization used for the image alignment step.
The whole procedure can be seen as composed of three main steps:
<ul>
    <li> <b>Graph Building</b>: in this phase, the graph representation of the pairwise homographies, able to align one image to another one, is built.</li>
    <li> <b>Image Alignment</b>: in this phase, multi-graph synchronization is applied to the previously constructed graph.
    Thanks to the synchronization algorithm, the unknown states representing global homographies are inferred.
    Differently from pariwise hommographies, global homographies are able to align the images to the common coordinate system.</li>
    <li> <b>Image Stitching</b>: in this phase, the stitched image is created. Each image is transformed in the reference frame by exploiting the estimated global homographies and fused with the others.</li>
</ul>

In the following, we provide some definitions that are useful for a better understanding of the project.

<h4>Multi-graph</h4>

A multi-graph $$\mathcal{G} = (\mathcal{V}, \mathcal{E}, s, t)$$ is a directed graph with multi-edges, where $$\mathcal{V}$$ is the set of vertices, $$\mathcal{E}$$ is the set of edges, $$s: \mathcal{E} \to \mathcal{V}$$ is a function that maps an edge to its source vertex, and $$t: \mathcal{E} \to \mathcal{V}$$ is function mapping an edge to its target vertex.



<h4>Multi-edge (Multi-arc)</h4>

Considering a multi-graph $$\mathcal{G} = (\mathcal{V}, \mathcal{E}, s, t)$$, the multi-edge $$E(i, j)$$ is the set of edges having the same source and destination vertices:

$$\begin{gathered} E(i, j) = \{ e \in \mathcal {E} \colon s(e) = i \land t(e) = j \}. \end{gathered}$$


<h4>Group-labeled multi-graph</h4>

A $$\Sigma$$-labeled multi-graph is a tuple $$\Gamma = (\mathcal{V}, \mathcal{E}, s, t, z)$$ where $$\mathcal{G} = (\mathcal{V}, \mathcal{E}, s, t)$$ is a multi-graph and $$z: \mathcal{E} \to \Sigma$$ is the edge labeling function. The edge set $$\mathcal{E}$$ satisfies the following property: if $$e \in \mathcal{E}$$ with $$s(e) = u \land t(e) = v$$, then $$e' \in \mathcal{E}$$ with $$s(e') = u \land t(e') = v$$, and $$z$$ satisfies: 

$$z(e) = z(e')^{-1}$$


<h4>Consistent labeling</h4>

Let $$\Gamma = (\mathcal{V}, \mathcal{E}, s, t, z)$$ be a $$\Sigma$$-labelled multi-graph and let $$x : \mathcal{V} \to \Sigma$$ be a vertex labeling. We say that $$x$$ is a consistent labeling if and only if the following condition holds $$\forall i, j \in \mathcal{V}$$ and $$\forall e \in \mathcal{E}$$ such that $$(s(e), t(e)) = (i, j)$$:

$$z(e) = x(i) * x(j)^{-1} $$


<h4>Multi-graph synchronization</h4>

Given a $$\Sigma$$-labeled multi-graph $$\Gamma = (\mathcal{V}, \mathcal{E}, s, t, z)$$, the task of multi-graph synchronization is to find the unknown vertex labelling $$x : \mathcal{V} \to \Sigma$$ that is the most consistent w.r.t. $$\Gamma$$. To apply existing synchronization techniques based on spectral solutions, the multi-graph has to be transformed into a simple graph because of the presence of multiple edges.

We can move to the first step of the pipeline, which is <b>Graph Building</b>
The application of <i>MULTISYNC</i> approach <d-cite key="DalCin2021"></d-cite> to the image stitching scenario, requires a consistently group-labeled multi-graph $$\Gamma = (\mathcal{V}, \mathcal{E}, s, t, z)$$ where each vertex $$i \in \mathcal{V}$$ is associated to an image and an edge is present between two vertices $$(i, j)$$ if image $$i$$ and image $$j$$ are matched.

The unknown state of each node $$i \in \mathcal{V}$$ represents the global homography needed to bring image $$i$$ into the global reference frame $$ref$$.
At the same time, the label of every edge $$e: \in E(i, j)$$ represents an estimated homography able to align image $$i$$ to image $$j$$ and normalized so that it belongs to <i>SL(3)</i>.

Being $$K$$ the multi-arc cardinality (or multi-edge degree) of $$\Gamma$$, $$K$$ homographies will be estimated from image $$i$$ to image $$j$$.
We call $$H_{ji}$$ the set of homographies estimated from $$i$$ to $$j$$ and $$H_{ji}^k$$ the $$k$$-th homography estimated from $$i$$ to $$j$$. Thus, $$H_{ji}^k$$ will be the label of the $$k$$-th edge $$e \in E(i, j)$$.


To build the graph, it is necessary to estimate pairwise homographies. To estimate a homography $$h \in H_{ji}$$, it is necessary to extract features from image $$i$$ and image $$j$$ first. To perform feature extraction <i>SIFT</i> <d-cite key="Lowe2004"></d-cite> algorithm is applied to image $$i$$ and image $$j$$. Based on the extracted features, <i>FLANN</i> algorithm <d-cite key="inproceedings"></d-cite> is then used to match features of image $$i$$ and $$j$$, obtaining a set of correspondences.
The low quality matches are filtered out using the Lowe's ratio-test <d-cite key="Lowe2004"></d-cite>. The basic idea is that each key-point $$k$$ of the first image is matched with a set of key-points from the second image. For each key-point $$k$$, the best two matches (in term of distance from $$k$$) are kept aside. Lowe's test checks that the two distances are sufficiently different and if they are not, then $$k$$ is eliminated.
Finally, to perform robust homography estimation, <i>RANSAC</i> <d-cite key="10.1145/358669.358692"></d-cite> algorithm is used on the correspondences between features of image $$i$$ and $$j$$.
The same procedure can be used to estimate every homography $$H_{ji}^k$$. Being <i>RANSAC</i> a non-deterministic algorithm, given $$h1, h2 \in H_{ji}$$, the estimated homographies can be different: $$h1 \neq h2$$.

Once the multi-graph has been designed, the first step of <i>MULTISYNC</i> is to apply an iterative greedy algorithm to expand the multi-graph to obtain a simple graph retaining all the pairwise information. 
The basic idea is to replicate the source or target vertices, allowing to expand each multi-edge into a number of simple edges equal to its cardinality.

{% include figure.liquid path="assets/img/mosaicing/exp_graph.png" title="Multi-graph expansion" class="img-fluid rounded z-depth-1" %}

Notice that in the expanded graph the multi-arc cardinality is $1$, while there are multiple vertices associated with the same image. Hard constraints in the form of identity edges ($$1_\Sigma$$) are added between the original node and the replicas to ensure the equality of the states.

After this step, it is possible to start with <b> image alignement </b>.
To align the images it is necessary to find a solution applying synchronization on the expanded graph. However, it is necessary to adapt the <i>spectral solution</i> proposed by <b>Schroeder et al.</b> <d-cite key="Schroeder2011"></d-cite> by adding identity constraints between the vertex replicas.
To account for this, <b>Dal Cin et al.</b> <d-cite key="DalCin2021"></d-cite>, proposes <i>Constrained eigenvalues optimization</i> that we briefly review in the following, adapting it to the considered scenario.

Let $$n$$ denote the number of vertices of the expanded graph and let's call $$X$$ the $$3n \times 3$$ block matrix collecting all the unknowns and $$Z$$ the $$3n \times 3n$$ one containing all the homographies:

$$\begin{equation*}
        X = \begin{bmatrix} 
                x{(1)} \\ x{(2)} \\ \vdots \\ x{(n)} 
            \end{bmatrix} 
            \!\!, \,\,\,\,
        Z = \begin{bmatrix} 
                {I}_{3} & z{(1,2)} & \dots  & z{(1,n)} \\
                z{(2,1)}  &  I_{3}     & \dots  & z{(2,n)} \\ 
                \vdots    & \vdots   & \ddots & \vdots \\ 
                z{(n,1)}  & z{(n,2)} & \dots  & {I}_{3}
            \end{bmatrix}
    \end{equation*}$$

where $$x(i)$$ is a $$3 \times 3$$ matrix representing the unknown state of vertex $$i$$ and $$z(i,j)$$ is a $$3 \times 3$$ matrix representing the edge label from vertex $$i$$ to vertex $$j$$.

The above equation refers to a complete graph, where all the edge labels are known. In the image alignment case, however, the graph is incomplete due to the presence of uncorrelated images. To deal with incomplete graphs, $$Z$$ is filled with zero $$3 \times 3$$ blocks in correspondence of missing edges.
The notation can thus be modified in this way

$$ {Z}_A = {Z} \circ ({A} \otimes {1}_3 ) $$

where $$A$$ denotes the adjacency matrix of the graph, $\circ$ indicates the Hadamard (or entry-wise) product, $$\otimes$$ denotes the Kronecker product and $$1_3$$ is a $$3 \times 3$$ matrix filled by ones.

To obtain a labeling of the expanded graph consistent with the underlying multi-graph, it is hence necessary to enforce that replicated vertices share exactly the same labels, leading to the constrained version of the <i>spectral solution</i>:

$$ \min _{{X}} \| {M}{X} \|_F^2 \quad \text {subject to } {X}^\top {X} = {I}_3, \quad {C}^\top {X} = 0 $$

where $$ {M} = {Z}_A  - ( {D} \otimes {I}_3)$$ is defined from the matrix of incomplete relative measurements, $${D}$$ is the degree matrix of the graph and $${C}$$ is the matrix used to impose the identity constraints between replicated vertices.

Finally, the stationary points of the cost function can be retrieved using a closed-form solution.
The hidden labels are given by the eigenvectors of 

$${S} = ( {I} - {C}{C}^\dagger){M}^\top {M}$$

where $${C}^\dagger$$ is the pseudo-inverse of $${C}$$ since the solution of the minimization problem is attained when $$x_i$$ are the 3 orthogonal eigenvectors of $${S}$$ corresponding to eigenvalues $$\lambda_{k+1} \dots \lambda_{k+3}$$ in ascending order, where $$k$$ is the rank of $${C}$$.

The reference frame $$ref$$ for the final mosaic is chosen among the input images frames and the homography that brings an image $$i$$ in the reference frame is computed as:

$$ H_{ref,i} = x(ref)  x(i)^{-1} $$

It is possible now to perform <b>image stitching</b>.
Using the global homographies resulting from the image alignment, each image is projected in the reference frame. Finally, the obtained images are fused into a single wide image using the maximum operator. Advanced image compositing techniques could be exploited to improve the qualitative result, but this is not the focus of our research.

<b>Dal Cin et al.</b> <d-cite key="DalCin2021"></d-cite> further illustrates how the multi-graph formulation can be used to deal with partitioned synchronization problems.
Based on this, we propose a partitioned mosaicing approach that could be used to deal with a large number of images to be stitched. In this scenario, a simple graph is considered instead of a multi-graph.
The approach follows the same steps of the author<d-cite key="DalCin2021"></d-cite>:
<ul>
    <li> <b>Graph Partitioning</b>: in this phase, the graph vertices are partitioned into several clusters.</li>
    <li> <b>Patch Synchronization</b>: in this phase, each obtained cluster is synchronized using the basic <i>spectral solution</i>.</li>
    <li> <b>Patch-graph Building</b>: in this phase, a new graph is constructed based on the results of the previous step.</li>
    <li> <b>Patch-graph Synchronization</b>: in this phase, multi-graph synchronization is applied to the patch-graph following the previously described approach.</li>
</ul>
Finally, image stitching is applied as explained before.

{% include figure.liquid path="assets/img/mosaicing/multi_patch.png" title="multi_patch" class="img-fluid rounded z-depth-1" %}

To verify the effectiveness of the proposed approaches, we compared their results with those obtained by applying other methods. In particular, we considered the following algorithms:
<b>Basic Stitching</b>: given a graph-representation $$\Gamma$$ of pair-wise homographies, and a reference image ($$I_{ref}$$), the idea is to estimate the homography between an image $$I_j$$ and $$I_{ref}$$ by combining the pair-wise homographies. In this way, one global homography at a time is computed.

<b>Simple-graph Stitching</b>: the spectral solution to the synchronization problem provided in <d-cite key="Schroeder2011"></d-cite> is applied to the graph-representation $$\Gamma$$ of pair-wise homographies.

<b>Edge-Averaging Stitching</b>: given a graph-representation of pair-wise homographies with multi-edges, instead of expanding the graph as in <d-cite key="DalCin2021"></d-cite>, the idea is to reduce the problem to Simple-Graph stitching by averaging together the $$K$$ multi-edges having the same source and destination nodes and obtaining a simple graph.
The resulting arc from vertex $$i$$ to vertex $$j$$ is labeled with $$H_{ji} = \frac{1}{K} \sum_{k=1}^{K} H_{ji}^k$$.
    
<b>Multi-graph Stitching</b>: this is our first proposed method.

<b>Multi-patch Stitching</b>: this is our second proposed method.

<b>Multi-patch Edge-Averaging Stitching</b>: the basic idea, in this case, is the same as the previous method with the only difference that the naive solution to the multi-graph synchronization problem (<i>edge averaging</i>) is applied on the patch graph.
    

An important remark is that averaging is not an operation that is well-defined for every group <d-cite key="DalCin2021"></d-cite>. In particular, in the case of homographies (<i>SL(3)</i>), averaging is not a theoretically sustained operation.
This means that there are no theoretical guarantees that edge labels obtained in Edge-Averaging Stitching and Multi-patch Edge-Averaging Stitching are homographies.

Eight datasets have been used to test our methods. Each of them is composed of a set of images (from 4 to 10) taken from the same camera rotating around the camera centre. Some of them are taken from <d-cite key="PanP2"></d-cite> (namely: mountain, helens, field, sun, snow). We further collected additional data in other locations under different lighting conditions (namely: pognana, pognana2, house).
The desired mosaics for the datasets are not available, so there is no available ground truth to evaluate the result of the mosaicing procedure. The complete experiments have not been conducted on <i>pognana2, house, field</i> datasets since they represent large panoramas. Dealing with a large horizontal FOV we obtained ambiguous results (<i>e.g.</i>, flipped images) that are probably due to numerical errors or a low number of loops in the graph.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/mosaicing/field.png" title="field dataset" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/mosaicing/mountain.png" title="mountain dataset" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/mosaicing/snow.png" title="snow dataset" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

 To decide the quality of a match the Lowe's ratio-test  is used with a threshold <i>matching\_threshold</i> set to $$0.7$$. Once the salient matches are selected, a homography is estimated from them only if a sufficient number of matches is present, <i>i.e.</i>, if there are more than <i>matches\_th</i>, set to $$30$$. Regarding the RANSAC algorithm, the maximum number of iterations <i>RANSACmaxIters</i> is set to $$2000$$. Also, the cardinality of the multi-arcs is represented by <i>number\_of\_matches</i>, set to $$10$$.
Furthermore, given the absence of ground truth to evaluate the correctness and goodness of the applied methods, we decided to implement a procedure that allows the creation of noisy images. In particular, the idea is to add some Gaussian noise to the points estimated by SIFT. Then, the methods are applied one by one and the resulting global homographies are compared with those obtained by applying simple-graph synchronization in a noise-free scenario, considered as ground truth.
To compare the estimated labels $$x_i$$ with the ground-truth ones $$x_{GT}$$, the error is defined as the Frobenius norm of the deviation from the identity:
$$
    \mathcal{E}_{i} = \| I_3 - (x_{GT} \; x_i^{-1})\|_{F} \quad \quad \forall i \in \mathcal{V}
$$
In this way, it is possible to understand the effectiveness of each method in dealing with noise data. The experiments have been conducted on different datasets 50 times to get a more robust estimate, changing the noise standard deviation and the multi-edge degree. It is important to notice that the multi-edge degree only influences <i>Edge-Averaging stitching</i> and <i>Multi-graph stitching</i> since they work on multi-graphs.

Let's now analyze the results.

<div class="row">
    <div class="col-sm mt-2 mt-md-0">
        {% include figure.liquid path="assets/img/mosaicing/snow_ma_5.png" title="snow_ma_5" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-2 mt-md-0">
        {% include figure.liquid path="assets/img/mosaicing/snow_ma_30.png" title="snow_ma_30" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row">
    <div class="col-sm mt-2 mt-md-0">
        {% include figure.liquid path="assets/img/mosaicing/sun_ma_5.png" title="sun_ma_5" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-2 mt-md-0">
        {% include figure.liquid path="assets/img/mosaicing/sun_ma_30.png" title="sun_ma_30" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row">
    <div class="col-sm mt-2 mt-md-0">
        {% include figure.liquid path="assets/img/mosaicing/sun_noise_2.png" title="sun_noise_2" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-2 mt-md-0">
        {% include figure.liquid path="assets/img/mosaicing/snow_noise_3.png" title="snow_noise_3" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
{% include figure.liquid path="assets/img/mosaicing/legenda.png" title="legenda" class="img-fluid rounded z-depth-1" %}

<i>Basic stitching</i> is the simplest, fastest and most intuitive method that we analysed. It can work even with acyclic graphs, however, given the fact that images are added one at a time to the mosaic, the error tends to accumulate. In fact, this approach does not exploit any global information but only the local one present in the two images that are being stitched. 

<i>Simple-graph stitching</i> tries to overcome this problem by exploiting cycles present in the graph and distributing the error that is present on the single edges. In principle, this should allow producing better results than the previous method. However, we noticed that while on real data (noise-free scenario), the results produced by the two methods are almost identical from a qualitative point of view, those produced on synthetic data vary according to the dataset that is used. 

We believe that this is due to the fact that <i>Simple-graph stitching</i> has a higher probability to encounter outlier edges since it takes into consideration all the edges present in the graph (while <i>Basic stitching</i> only considers a subset of them). Outlier edges are edges heavily affected by noise, that bring a negative contribution to the overall performance of the method. A more robust procedure should be taken into consideration in future works.


Multi-graph stitching (our first proposed approach) reaches the best results on all of the datasets. The key advantage over other methods is that it takes into consideration multi-arcs, thus outperforming methods based on simple graphs. Note that when considering the <i>Multi-arc cardinality</i> equal to one, the performance of <i>Multi-graph stitching</i> and <i>Simple Graph stitching</i> are the same (this is obvious since Multi-graph reduces to Simple-graph). Increasing the <i>multi-arc cardinality</i> reduces the error and results become better and better.
This is due to the fact that having multiple measurements between the same pair of nodes (<i>i.e.,</i> higher multi-arc cardinality) guarantees more robustness to noise.

<i>Edge-Averaging stitching</i> also exploit the information from the multi-edges, in a more naive way w.r.t. <i>Multi-graph stitching</i>. Surprisingly, in our results, the performance of the two methods is the same. Note that no theoretical guarantees are present in the case of <i>Edge-Averaging stitching</i>. Several experiments demonstrated that the average of the edges is still close to be in <i>SL(3)</i>. This aspect should be better analysed in the future.

Regarding <i>Multi-Patch stitching</i> (the other proposed method), we observed that it performs worse than the others on some datasets in a noise-free scenario while it is always the worst when noise is added. Using its variant <i>Multi-Patch Edge-Averaging stitching</i>, performances seem to get better in many cases, being comparable to those provided by <i>Basic stitching</i> and <i>Simple-graph stitching</i>. 

Given that <i>Multi-Patch stitching</i> solves synchronization problems on each patch locally, the solution obtained by synchronizing the patch graph is not guaranteed to be the global optimum since only a portion of the available global information is exploited. Thus, it is reasonable that its performances are worse than methods not based on patches. However, <i>Multi-Patch Edge-Averaging stitching</i> seems to reduce this effect.

Also in this case, there are no theoretical guarantees regarding the average over homographies, thus this could be a starting point for future analysis.
Independently of the method that is used, the error increases when noise is bigger intuitively. Instead, fixing the noise and increasing the multi-edge degree allows to improve the results of methods based on multi-graphs following the theoretical expectations.

The obtained results show the effectiveness of MULTISYNC <d-cite key="DalCin2021"></d-cite> applied to a real image mosaicing scenario, being very robust in particular in presence of noise and showing superiority w.r.t. other approaches.
The results show also similar performances for the naive approach based on edge-averaging, even with the lack of theoretical guarantees.
MULTISYNC does not seem very effective in partitioned mosaicing.
Future research could focus on:
<ul>
    <li> Investigating the relationship between the results obtained with <i>Multi-Graph stitching</i> and <i>Edge-Averaging stitching</i>.</li>
    <li> Investigating the relationship between the results obtained with <i>Multi-patch stitching</i> and <i>Multi-patch Edge-Averaging stitching</i>.</li>
    <li> Considering the introduction of multi-arcs in patch synchronization for partition mosaicing, using MULTISYNC for each cluster instead of the spectral solution.</li>
    <li> Collecting more challenging datasets to evaluate the performances of multi-graph based methods on real data introducing when possible a ground truth.</li>
</ul>

You can find the implementation in the <a href ="https://github.com/fasana-corrado/RGB-Mosaicing-via-Multigraph-Synchronization">linked repository</a>.
