---
layout: distill
title: Facial Expression DA
description: A generative approach for Facial Expression Data Augmentation
importance: 2
related_publications: false
bibliography: 2022-11-27-cycle.bib

---

Image-to-Image translation aims to transfer images from a source domain to a target one while preserving the content representations. It can be applied to a wide range of applications, such as style transfer, season transfer, and photo enhancement. 

To accomplish this task several architectures have been proposed, including CycleGANs <d-cite key="Zhu2017"></d-cite> which are composed of a pair of GANs, and their improved version Enhanced CycleGAN (ECycleGAN) <d-cite key="Zhang2020"></d-cite>.
I already analyzed these networks in <a href = "https://pasinisamuele.github.io/blog/2022/cycle/"> this blog post </a> which I suggest you to read before continuing.

In the context of Deep Learning, one major problem is the need for huge datasets to effectively train a deepmodel. However, the amount of available images can be limited and dependent on the specific class, leading to unbalanced datasets. To solve this problem, a large amount of different data augmentation techniques have been developed during the last years.

I presented this project with Dr. Francesco Azzoni and Dr. Corrado Fasana to the Advanced Deep Learning Models and Methods exam in Politecnico di Milano.

The work exploits the advancements in Image-to-Image translation to perform data augmentation of facial expression data.

The effectiveness of the proposed method is assessed analysing the classification performance on the unbalanced FER2013 facial expression dataset <d-cite key="fer2013"></d-cite>.

Before exploiting ECycleGANs to perform data augmentation, an attempt was made to reproduce the results obtained by Zhu et al. <d-cite key="Zhu2018"></d-cite> to have a model for comparison. The authors used a subset of the dataset to perform the experiments, but the sampling strategy is left unspecified. The result obtained using a random sampling strategy did not match those published in the paper. By analyzing the original dataset it is evident that a random sampling strategy led to the usage of low quality samples jeopardizing the result, and more sophisticated ways to select the dataset subset must be explored.

The original FER2013 dataset <d-cite key="fer2013"></d-cite>  has 7 classes, namely: <i>Angry, Disgust, Fear, Happy, Sad, Surprise, Neutral </i>. The total number of samples is 35887, and some classes are more represented then others (<i>e.g.,</i> <i>Disgust</i> has 547 samples while <i>Happy</i> 8989). However, due to the previous observations, a filtered version of the dataset was used.
The filtered dataset instead has 28941, keeping the same imbalanced nature of the classes (<i>e.g.,</i> <i>Disgust</i> samples: 421, <i>Happy</i> samples: 7967). 100 images per class are kept apart to test the multi-class classifier, while the remaining ones are used for training.

FER2013 dataset <d-cite key="fer2013"></d-cite> is characterized by high intra-class diversity, high inter-class similarity, and the presence of mislabeled samples and samples belonging to other domains. Thus, the following techniques were employed to try to mitigate these problems and check whether the performances reported in <d-cite key="Zhu2018"></d-cite> could be replicated:
<ul>
    <li><b>Gaussian likelihood</b>: the idea is to perform instance selection by removing those instances that lie in low-density regions of the data manifold. This is done by exploiting the method proposed in DeVries et al. <d-cite key="NEURIPS2020_99f6a934"></d-cite> which computes the likelihood of each image using a Gaussian model fit on feature embeddings produced by a pre-trained Inceptionv3 classifier <d-cite key="inceptionV3"></d-cite>. Then, only those samples with a likelihood greater than a certain threshold are kept. 
    </li>
    <li> <b> Confidence Filtering </b>: the idea is that first a classifier is trained on the whole dataset. All the samples are then evaluated using the classifier and ranked according to the probability of belonging to their annotated class. Using as threshold a minimum confidence or a number of samples it is possible to obtain a filtered version of the dataset where most of the ambiguous samples are discarded. </li>
</ul>
However, even though both these approaches seemed to correctly remove ambiguous samples, the performance reported in the paper <d-cite key="Zhu2018"></d-cite> could not be reached.

Once the dataset has been filtered, the next step is the implementation and training of the EcycleGAN model. Starting from the existing implementation of CycleGAN provided by <d-cite key="Zhu2017"></d-cite> several adjustments are made to match the architecture described in <d-cite key="Zhang2020"></d-cite>:

<ul>
    <li> Instead of the original residual block <d-cite key="resnet"></d-cite>  of the CycleGAN generator a deeper and more complex one is employed, called RDNB.</li>
    <li> The perceptual loss is implemented in place of the original pixel-wise loss. </li>
    <li> The Wasserstein GAN objective with gradient penalty (WGAN-GP) <d-cite key="wgangp"></d-cite> is implemented as an alternative to the LSGAN objective function <d-cite key="LSGAN"></d-cite> to stabilize the training. Both have been tested and compared in the experiments section.</li>
</ul>

Given the fact that the number of samples of some classes is very limited, the discriminator of such classes tends to overfit. Several GAN-specific techniques have been proposed to mitigate this problem:
<ul>
    <li> <b>One-sided Label smoothing</b> <d-cite key="label_smoothing"></d-cite>: it is concerned with the addition of label noise. More precisely, the discriminator is trained on randomly flipped labels instead of real labels. This label noise is applied only to the samples of the less represented class when fed to the discriminator.</li>
    <li> <b>Instance Noise</b> <d-cite key="label_smoothing"></d-cite>: in this case, the discriminator sees the correct labels, but its input sample is noisy. This avoids the saturation of the discriminator objective, reducing overfitting. The added noise is Gaussian with zero mean and decaying standard deviation during training.</li>
    <li> <b>Alternate training</b>: when the training procedure is initiated (after a fixed number of epochs) the discriminator of the less represented class is not trained at every iteration. Since the number of samples is small the discriminator is prone to overfitting, thus it can discriminate between real and fake samples with very low uncertainty. Hence, training it less frequently than the generator gives the latter more time to learn.</li>
</ul>

After the implementation of the proposed approaches, to compare the quality of the synthetic data generated using CycleGAN and especially ECycleGAN, several experiments were conducted using different parameters and finally compared. The final performance of the classifier on the original and augmented dataset are assessed using 3x10-fold cross-validation and averaging the results.

For the first experiments, the employed CycleGAN architecture and parameters are the same as <d-cite key="Zhu2018"></d-cite>.

The paper does not consider the identity loss, so different experiments were performed weighting the identity loss in different ways ($$\lambda_{idt} = [0.0; 0.3; 0.5]$$). For each configuration, after training, 100 <i>Disgust</i> images are generated starting from a fixed set of <i>Neutral</i> images and added to the original dataset to assess the impact on the classifier performance.
Initially, the cycle consistency loss weight was set to $$\lambda_{cyc} = 10$$ as in the paper<d-cite key="Zhu2018"></d-cite>. However, given that the generated images were extremely similar to the input ones, the $$\lambda_{cyc}$$ was fixed to 1 for most of the following attempts, leading to a more significant impact on the image, while maintaining a convincing cycle consistency and a low reconstruction error. Finally, some generated samples are also manually selected to compare the different settings from a qualitative perspective.

{% include figure.liquid path="assets/img/cycle_gan/cycle_gan.png" title="cycle gan" class="img-fluid rounded z-depth-1" %}

The experiments of ECycleGAN started replicating the same base configuration reported in Zhang et al. <d-cite key="Zhang2020"></d-cite>., which makes use of WGAN-GP loss function <d-cite key="wgangp"></d-cite>. and RDNBs with 3 dense-blocks, each composed of 5 convolutional densely connected sub-blocks.

Unfortunately due to the limited available resources neither training such a powerful network nor performing a complete hyper-parameters tuning is feasible. Thus, the number of dense blocks and sub-blocks is decreased to 2 and 3 respectively.
Given the poor results and the divergence problems that arose using the proposed WGAN-GP, the following experiments were performed using LSGAN <d-cite key="LSGAN"></d-cite>. that resulted in a more stable training. 
The coefficient for the Cycle-loss $$\lambda_{cyc}$$ is set to the same value used in the CycleGAN case, and the same holds for the $$\lambda_{idt}$$ values. 
Regarding the perceptual loss, the balancing coefficients $$\alpha$$ and $$\beta$$ are both set to $$0.5$$ after some trial and error, and the feature map considered for the feature-loss component is the one generated by the last convolutional layer of the VGG19 feature-extractor.
Finally, the residual scaling of the RDNBs is tuned by choosing between the values $$\lambda_{idt} = [0.3; 0.5; 0.7]$$, balancing the influence of the residual connection.
As done for CycleGANs, generated samples are manually selected to compare the different settings from a qualitative perspective.

{% include figure.liquid path="assets/img/cycle_gan/ecycle_gan.png" title="ecycle gan" class="img-fluid rounded z-depth-1" %}

Given that the discriminator of the less represented class tends to overfit in the previous models, the model providing the more promising results was selected and enhanced with the techniques previously to try to mitigate this problem. 
More specifically, Label-smoothing was applied with a label flip probability of $$1\%$$. Instance-noise was implemented using a Gaussian noise starting with a standard deviation of $$0.1$$ and $$0.05$$ then linearly decaying, while in the case of Alternate-training the training ratio generator-discriminator was set to $$10:1$$ or $$5:1$$. 
As done for CycleGAN, for each trained model, 100 <i>Disgust</i> images are generated starting from the same fixed set of <i>Neutral</i>  images for better comparison and added to the original dataset to assess the impact on the classifier performance. Then, the most promising model is used to generate also 200 and 500 images to check whether a bigger augmentation can further boost the classifier performance.
Finally, a few experiments were performed to generate <i>Surprise</i>  images from <i>Neutral</i>  ones, to check the impact of the imbalance gap.

The obtained results were evaluated both qualitatively and quantitatively. In particular, a qualitative evaluation of the images generated by each experiment was performed first. Then, a quantitative evaluation of multi-class classification was assessed using different metrics.
<ul>
    <li> Mean <i>Precision</i>. </li>
    <li> Per-Class <i>Precision</i>. </li>
    <li> Mean <i>Receiver Operating Characteristic - Area Under the Curve (ROC-AUC)</i>. </li>
    <li> Per-Class <i>ROC-AUC</i>. </li>
</ul>

The idea is that the Precision can provide a first very intuitive indication of the classifier's capabilities, while the AUC tells how much the model is capable of distinguishing between classes (in a one-vs-all setting).

The main improvement from CycleGAN to ECycleGAN is related to the quality of the generated images.

{% include figure.liquid path="assets/img/cycle_gan/ecycle_vs_cycle.png" title="ecycle_vs_cycle" class="img-fluid rounded z-depth-1" %}


Also, the ratio between the number of images of the two domains, and the consequent loss behavior, have a huge impact on the quality of the generated samples. The smaller the gap (<i>e.g., Neutral-Surprise</i> translation), the better the quality.

{% include figure.liquid path="assets/img/cycle_gan/ecycle_vs_cycle_surprise.png" title="ecycle_vs_cycle_surprise" class="img-fluid rounded z-depth-1" %}


The techniques previously proposed to tackle the overfit were adopted to force the loss behavior of the Neutral-Disgust experiment in which the samples gap is far larger, to be similar to the Neutral-Surprise one, wishing to increase the quality of the results. While the loss behavior was effectively changed as desired, the quality of the images does not show a convincing improvement.

{% include figure.liquid path="assets/img/cycle_gan/overfitting_mitigation.png" title="overfitting_mitigation" class="img-fluid rounded z-depth-1" %}

The same classification setup is used for all the experiments, taking into consideration previously cited metrics for evaluation. The classification performance on the non-augmented filtered dataset is used as the baseline.

Among the CycleGAN experiments there is a noticeable improvement in the <i>Disgust</i> precision w.r.t. the baseline, but at the same time the other classes' precision decreases, resulting in a mean precision that is similar to the baseline one. This is because only the samples that are clearly disgusted are classified as such, reducing the false positives of class <i>Disgust</i>. The other less certain Disgust samples are assigned to other classes, increasing their false positives and reducing their precision. According to the AUC the quality of the augmentation depends on the value of $$\lambda_{idt}$$ used: the higher the value the lower the AUC. Thus, it is possible to derive that synthetic samples that have features similar to real faces (more probable with higher $$\lambda_{idt}$$) are more easily misclassified since the stronger identity constraint does not allow to make them look disgusted. Thus, CycleGAN architecture is not powerful enough to allow better discrimination between classes, leading to a mean AUC which is similar to that of the baseline.

Regarding the ECycleGAN the first experiments were performed to establish the best value of residual scaling ($$\alpha$$) and identity loss weight $$\lambda_{idt}$$ hyper-parameters. When using $$\lambda_{idt} = 0.5$$ and $$\alpha=0.5$$ the best overall improvement is obtained for both metrics. Different values of these parameters cause input images to be modified too much or not enough.
Using the best model, different techniques to avoid overfitting were employed, however even though there is an improvement concerning the generator loss convergence, no significant performance boost was observed. 
Increasing the number of augmented images to 200 or 500 decreases the performance due to the introduction of too many bad-quality samples. Finally, considering a more represented class such as <i>Surprise</i>, the ECycleGAN qualitative performance is surprising also w.r.t. CycleGAN. However, there is still not much improvement in the classifier performances since this class is already quite distinguishable from the others and thus, the introduction of some lower quality samples can even slightly decrease the performances w.r.t. the baseline.

{% include figure.liquid path="assets/img/cycle_gan/precision.png" title="precision" class="img-fluid rounded z-depth-1" %}

{% include figure.liquid path="assets/img/cycle_gan/auc.png" title="auc" class="img-fluid rounded z-depth-1" %}

Our experiments show that ECycleGAN can be more effective than CycleGAN for Facial Expression Data Augmentation.
Despite the quality of some generated samples is good, the influence of bad generated samples is too high to augment the dataset significantly. While introducing a low number of generated samples (around 100) leads to a limited performance boost, when introducing more samples (from 200 on), the major influence of the bad-quality ones results in a degradation of the classification performances since the intra-class diversity increases too much.

However, the improvement w.r.t. the previously proposed CycleGAN model is present from both qualitative and quantitative viewpoints.

Further experiments and studies could be conducted by adopting the complete ECycleGAN architecture and performing a finer hyper-parameter tuning if resources are available. Moreover, other filtering methods for the input dataset could be explored, as well as an instance selection algorithm to choose the best samples generated by ECycleGANs. Other techniques to avoid overfitting the discriminator could be considered. Finally, pretraining the VGG19 network used for the perceptual loss with a pretext task on human faces could help to extract more suitable features to be considered for the consistency losses, boosting the performance.

You can find the implementation in the <a href ="https://github.com/PasiniSamuele/ECycleGAN-face-emotion">linked repository</a>.
