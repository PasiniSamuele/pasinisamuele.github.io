---
layout: distill
title: Image Mosaicing
date: 2022-10-18 16:24:16
description: Image alignement for mosaicing images, with focus on synchronization algorithms
tags: university computer-vision
categories: papers
bibliography: 2022-10-18-mosaicing.bib

---
Image mosaicing <d-cite key="GHOSH20161"></d-cite> is an effective means of constructing a single seamless image by aligning multiple partially overlapped images.
In Computer Vision (CV), many applications, such as super-resolution imaging and medical imaging, require image mosaicing <d-cite key="Szeliski2006"></d-cite> require image mosaicing. This technique can also be used in panoramic stitching, allowing the creation of wide-angle images (without using fish-eye lenses) overcoming the difficulties in taking a photo with a very large field of view (FOV).
Thus, image stitching algorithms have been used for decades to create the high-resolution photo-mosaics used to produce digital maps and satellite photos. Ideally, the resulting stitched image should be as natural as a real photo that covers the entire scene.

However, while creating a mosaic from only two overlapping images is a relatively easy task and standard techniques can provide very good results, aligning multiple images is much more difficult, particularly if some input images do not overlap.

More in depth, image mosaicing could be regarded as a special case of scene reconstruction where the images are related by planar homography only. This is a reasonable assumption if the images exhibit no parallax effects, <i>i.e.</i>, when the scene is approximately planar or the camera purely rotates about its optical centre.

In general, this procedure can be divided into <b>image alignment</b> and <b>image compositing</b> steps.
The goal of image alignment is to align the images into a common coordinate system (most of our work is related to this step). 
The goal of image compositing is to overlay the aligned images on a larger canvas by merging pixel values of the overlapping portions and retaining pixels where no overlap occurs. It is usually performed in two steps: colour correction and blending. Colour correction is needed since neighbouring images can present different colours due to factors such as the exposure level and differences in the lighting condition.

As stated in <d-cite key="Santellani2018"></d-cite> the result of image mosaicing should not be confused with orthophotos. In fact, the former allows the visualization of a wide area on a single image under perspective projection, whereas the latter considers orthographic projections.
For this reason, image mosaicing does not need any prior Structure-from-Motion or dense matching phases, that are instead required to generate orthophotos.

Several methods for automatic image mosaicing are present in the literature, presenting a complete pipeline for the final mosaic generation <d-cite key="698630"></d-cite><d-cite key="1315099"></d-cite><d-cite key="Brown2007"></d-cite><d-cite key="doi:10.1080/02533839.2005.9670998"></d-cite> or focusing on one of the previously cited steps <d-cite key="Schroeder2011"></d-cite><d-cite key="LI20161"></d-cite><d-cite key="5995658"></d-cite>.

The main focus of this post is related to the image alignement.
The image alignment step refers to the alignment of the images into a common coordinate system using the computed geometric transformations.
Existing algorithms <d-cite key="Szeliski2006"></d-cite> for this task are broadly categorised based on the information they extrapolate from the image.

<i>Direct methods</i> exploit the entire image data, thus providing very accurate registration but requiring at the same time a close initialization. They either compute the similarity based on image intensity values or based on the quantity of information (mutual information) shared between two images.

In contrast, <i>feature-based algorithms</i> rely on the computation of transformations using a sparse set of low-level features and can be computationally less expensive. Commonly used low-level features (<i>e.g.</i>, edges, corners, pixels, colors, histograms) can be extracted exploiting a variety of approaches <d-cite key="Lowe2004"> </d-cite><d-cite key="10.1007/11744023_32"> </d-cite><d-cite key="10.1007/11744023_34"></d-cite><d-cite key="6885761"></d-cite>. In particular, <b>Brown et al.</b> <d-cite key="Brown2007"></d-cite> proved that formulating stitching as a multi-image matching problem and using invariant local features to find matches between the images, allows building a method insensitive to ordering, orientation, scale and illumination of the input images.

Image alignment and colour correction can be both solved using graph synchronization techniques.
The synchronization problem can be defined as follows: 
{% quote %}
Given a graph where nodes are characterized by an unknown state, and edges measure the ratio (or difference) between the states of the connected nodes, try to infer the unknown states from the pairwise measures.
{% endquote %}

More precisely, states are represented by the elements of a specific group (this is why the problem is referred to as group synchronization). Recently, the synchronization problem has been extensively investigated in the Computer Vision community <d-cite key="Schroeder2011"></d-cite><d-cite key="Arrigoni2020"></d-cite><d-cite key="DalCin2021"></d-cite>.
<b>Schroeder et al.</b> <d-cite key="Schroeder2011"></d-cite> proposed four closed-form solutions to the synchronization problem for the specific task of image alignment.
In this specific scenario, the global homographies represent the unknown states of a graph where the edges are pairwise homographies. For this reason, states belong to the <i>SL(3)</i> group (Special Linear group, <i>i.e.</i>, set of 3x3 matrices with unit determinant).

<b>Arrigoni et al.</b> <d-cite key="Arrigoni2020"></d-cite> review several methods based on synchronization where the groups have a matrix representation, that allow closed-form solutions.

<b>Dal Cin et al.</b> <d-cite key="DalCin2021"></d-cite> proposed an algorithm (MULTISYNC) for solving the synchronization problem in the case of multi-graphs (graphs with multiple edges connecting the same pair of nodes) based on an expansion algorithm coupled with a constrained spectral solution to deal with replicated nodes. Our work moves in the direction of <d-cite key="DalCin2021"></d-cite> trying to apply multi-graph synchronization in the image alignment scenario as considered in <d-cite key="Schroeder2011"></d-cite>.

As explained in <d-cite key="DalCin2021"></d-cite>, the basic solution to multi-graph synchronization is <i>edge averaging</i>, <i>i.e</i>, converting a multi-graph into a simple-graph by averaging the measurements of the edges having the same source and destination nodes. However, edge-averaging is not well defined for all the groups. For instance, while it is possible to average rotations <d-cite key="Hartley2012RotationA"></d-cite>, there is not a theoretically sustained averaging for homographies. Thus, we study the results provided by edge averaging in the case of homographies and compare them with multi-graph synchronization.
The same multi-graph framework <d-cite key="DalCin2021"></d-cite> can be applied to partition classical synchronization tasks, achieving a good trade-off between accuracy and complexity.
The whole procedure can be seen as composed of three main steps:
<ul>
    <li> <b>Graph Building</b>: in this phase, the graph representation of the pairwise homographies, able to align one image to another one, is built.</li>
    <li> <b>Image Alignment</b>: in this phase, multi-graph synchronization is applied to the previously constructed graph.
    Thanks to the synchronization algorithm, the unknown states representing global homographies are inferred.
    Differently from pariwise hommographies, global homographies are able to align the images to the common coordinate system.</li>
    <li> <b>Image Stitching</b>: in this phase, the stitched image is created. Each image is transformed in the reference frame by exploiting the estimated global homographies and fused with the others.</li>
</ul>

I proposed applied MULTISYNC to solve partitioned synchronization problems estimating global homographies for image mosaicing.
You can find more detail about this projects <a href = "https://pasinisamuele.github.io/projects/image_mosaicing/">at this link</a>.