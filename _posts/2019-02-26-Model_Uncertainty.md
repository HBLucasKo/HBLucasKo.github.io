---
title: Introduction to Bayesian Deep Learning
category: Bayesian Deep Learning
tag: BDL
---

## 0. Introduction to Bayesian Learning

In analyzing data or making decisions, it is necessary to be able to tell whether a model is certain about its output, being able to ask “maybe I need to use more diverse data? Or change the model? Or perhaps be careful when making a decision?”. Such questions are of fundamental concern in Bayesian machine learning. As mentioned earlier before this post, deep learning models generally only have point estimates of parameters and predictions at hand. The use of such models forces us to sacrifice the tools for answering the questions above, potentially leading to situations where we can’t tell whether a model is making sensible predictions or just guessing at random. Most deep learning models are viewed as deterministic functions and as a result viewed as operating in a very different setting to the probabilistic models which possess uncertainty information. I will not go over basic deep learning criteria, but you will find a lot of posts about it.

## 1. Model uncertainty

Deep learning can be applied for diverse applications such as skin cancer diagnosis, autonomous vehicles, and dog breed classification websites. For example, given several pictures of dog breeds as training data, when a user uploads a photo of his dog, the website returns a prediction of the breed with high confidence. But what will happen if we input a cat’s image instead of a dog image? The image of a cat is an example of *out of distribution* test data. The model has been trained on photos of dogs of different breeds and learned how to distinguish them by giving out an output. However, by inputting a cat’s image, the model will not say anything about “cats” but will output one of the dog breeds with confidence. The illustrative example can be extended to more serious settings, such as MRI scans for patients, or autonomous car steering system. These examples are directly related to a person’s life and should be seriously considered. This means that we really must trust the **AI** by guarantying our only life. Therefore, we want our model to possess some quantity conveying a high level of uncertainty with such inputs. This means we want our model to say, “I don’t know!!” if the models receive a strange input or have low confidence.

<center><img src="https://i.imgur.com/gZpsuGI.png"></center>

Other situations that can lead to uncertainty include

*Noisy data (for example because of measurement imprecision, leading to *aleatoric uncertainty*)

* Uncertainty in *model parameters* that best explain the observed data (for example, weights and biases for deep neural network in which case we might be uncertain about what parameter should we choose.)

* And *structure uncertainty* (what model structure should we use?)

Uncertainty information is also very important for the practitioner. Understanding if a model is under-confident or falsely over-confident (which means its uncertainty estimates are too small), can help better performance out of it. Interestingly, perhaps much more important, model uncertainty information can be used in systems that make decisions that affect human life such as in medical domain or autonomous control of drones or cars. This can all be considered under the umbrella field of *AI safety*. The more the consequence for the prediction, the more we should concern the uncertainty.

Let’s consider about some of the examples. With self-driving cars, low level feature extraction such as image segmentation and image localization are used to process raw sensory input. The output of such models is then fed into higher-level decision-making procedures. The higher-level decision making can be done through expert systems for example relying on a fixed set of rules (“if there is a pedestrian at your right, you should not steer right.”) However, mistakes done by lower-level components can propagate up the decision-making process and lead to devastating results. This means one little failure of the algorithms can result in serious problems. In such modular systems, one could use the model’s confidence in low-level components and make high-level decisions given this uncertainty information. For example, a segmentation system which can identify its uncertainty in distinguishing between the sky and another vehicle could alert the user to take control over the steering.

## 2. Applications of model uncertainty

Beside AI safety, there are many more applications which rely on model uncertainty. These can include choosing what data to learn from or exploring an agent’s environment efficiently. Common to both these tasks is the use of model uncertainty to learn from *small amounts of data*. This is often a necessity setting in which data collection is expensive or time consuming. 

Many machines learning algorithms, including deep learning, often require large amounts of labelled data to generalize well. The amount of labelled data required increases with the complexity of the problem or the complexity of the input data. We should note that as complexity of the problem increases, we need way more data. To automate MRI scan analysis for example, this would require an expert to annotate many MRI scans, labelling them each part by part and indicate whether the patient have cancer or not. But expert time is very expensive and obtaining large amount of annotated data is a serious expensive issue. So how can we learn in settings where labelled data is scarce and expert knowledge is expensive? 

One possible approach to the task could rely on active learning. This means the model itself would be able to choose what unlabeled data would be most informative for it and ask an external “oracle” (human annotator for example) for a label only for these new datapoints. The choice of data points to be labelled is done through an **acquisition function** which ranks points based on their **potential informativeness**. By following this learning framework, we can decrease large amount of required data by orders of magnitude, while still maintaining good model performance. In the result, we should aim to produce good uncertainty estimates for image data and rely on these to design an appropriate function. For later, we will try to cover and develop extensions of such tools that can be deployed in small data regimes and provide good model confidence.

## 3. Model uncertainty in deep learning


It is important to note that most deep learning models do not offer model confidence. Regression models output a single vector regressing to the mean of the data and classification models output a probability vector (SoftMax) at the end of the pipeline which is often erroneously interpreted as model confidence. 

<center><img src="https://i.imgur.com/92mMsHI.png"></center>

The image above shows a sketch of softmax input and output for an idealized binary classification problem. When training data is given between the dashed grey lines, function point estimate is shown with a solid line. Function uncertainty is shown with a shaded area. Marked with a dashed red line is a point x* far from training data and by ignoring function uncertainty, point x* is classified as class 1 with probability 1. This shows a model can be uncertain in its predictions even with a high softmax output. Passing a point estimate of a function trough a softmax results in extrapolations with unjustified high confidence for points far from the training data. However, passing the distribution (shaded area for the picture) through a softmax, better reflects classification uncertainty far from the training data. You might not get this part clearly. But let’s try to slowly figure this out. 

Even though modern deep learning methods do not capture model confidence, they are closely related to a family of probabilistic models which induce probability distributions over functions: the **Gaussian process**. By placing a probability distribution over each weight (other than just a constant), a Gaussian process can be recovered in the limit of infinitely many weights. When we place distributions over the weights, we call that model, *Bayesian neural networks*. These networks may look fancy but some of these models are quite difficult to work with-often requiring many more parameters to be optimized and haven’t really caught-on within the deep learning community, perhaps because of their limited practicality.

So how should we make it practical?  Well these methods should apply well to modern architectures including CNNs and RNNs. And we will thus concentrate on the development of practical techniques to obtain model confidence in deep learning, techniques which are also well rooted within the theoretical foundations of probability theory and Bayesian modelling. We will make use of stochastic regularization techniques (SRTs). These techniques adapt the model output stochastically as a way of model regularization which results in the loss becoming a random quantity, which is optimized using tools from the stochastic non-convex optimization literature. Popular SRTs include dropout, multiplicative Gaussian noise, drop Connect, and countless other recent techniques.

Let’s say that we can take almost any network trained with an SRT and given some input x’ obtain a predictive mean E[y’] (the expected model output given our input), and predictive variance Var[y’] (how much the model is confident in its prediction). Then we simulate a network output with given x’, applying SRT as if we were using the model during training (obtaining a random output through a stochastic forward pass). We repeat this process several times (for T repetitions), sampling outputs {y1’(x’), y2’(x’), …yT’(x’)}. As will be explained below, these are empirical samples from an *approximate predictive distribution*. We can get an empirical estimator for the predictive mean of our approximate predictive distribution as well as the predictive variance (uncertainty) from these samples:

<center><img src="https://i.imgur.com/QmbvJ89.png"></center>

More justification and explanation will be given on next post. Equation (1.4) results in uncertainty estimates which are practical with large models and big data, and that can be applied in image-based models, sequence-based models, and many different settings such as reinforcement learning and active learning.

For future work, we will try to apply this Bayesian concept to many models today. The most notable of these are a theoretical analysis of Monte Carlo estimator variance used in variational inference, a survey of measures of uncertainty in classification tasks, an empirical analysis of different Bayesian neural network priors and posteriors with various approximating distributions, new quantitative results comparing dropout to existing techniques, tools for
heteroscedastic predictive uncertainty in Bayesian neural networks, applications
in active learning, a discussion of what determines what our model uncertainty looks like, an analytical analysis of the dropout approximating distribution
in Bayesian linear regression, an analysis of ELBO-test log likelihood correlation, discrete prior models, an interpretation of dropout as a proxy posterior in spike and slab prior models, as well as a procedure to optimize the dropout probabilities based on the variational interpretation to separate the different sources of
uncertainty.
