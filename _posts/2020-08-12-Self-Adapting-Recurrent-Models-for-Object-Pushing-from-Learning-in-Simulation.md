---
layout: post
title: "Self-Adapting Recurrent Models for Object Pushing from Learning in Simulation"
date: 2020-08-12
---


[Lin Cong](https://tams.informatik.uni-hamburg.de/people/cong/), [Michael Görner](https://tams.informatik.uni-hamburg.de/people/goerner/), [Philipp Ruppel](https://tams.informatik.uni-hamburg.de/people/ruppel/), [Hongzhuo Liang](https://tams.informatik.uni-hamburg.de/people/liang/), [Norman Hendrich](https://tams.informatik.uni-hamburg.de/people/hendrich/), [Jianwei Zhang](https://tams.informatik.uni-hamburg.de/people/zhang/)

## Related works and why do we propose this model

Planar pushing problem has attracted many famous research groups' attention. Planar object pushing with a single contact is a typical under-actuated instance of robot manipulation. The uncertainty of different physics parameters and the pressure distribution makes it difficult to build a precise motion model for real interactions. Learning a data-driven model ([M.Bauza & A.Rodriguez](https://arxiv.org/abs/1704.03033)) is an effective method due to the stochastic nature of the pushing process. Valuable dataset including [Omnipush Dataset](http://web.mit.edu/mcube/omnipush-dataset/) by MIT, [Pushing Dataset](https://sites.google.com/site/brainrobotdata/home/push-dataset) by Google Brain are open to the whole community. Besides the model-based method, end-to-end policy learning by mapping from state space to action space directly is also becoming a trend.

Recurrent Neural Networks (RNNs) have long been used for fitting nonlinear dynamic models.
With the distinctive ability to recognize patterns in sequences of data, the LSTM module is chosen to fit the object motion dynamics during the pushing process. We use an LSTM-based model to predict the motion of objects with unknown parameters
and apply [Model Predictive Path Integral](https://homes.cs.washington.edu/~bboots/files/InformationTheoreticMPC.pdf) as control strategy to push the objects to target poses with a single point of contact.

![Planar pushing object motion prediction problem]({{ site.url }}/data/images/self-adapting-description.png)
{: style="width: 70%;" class="center"}
*Figure 1. The planar pushing object movement prediction problem*

## Recurrent Model

Humans can approximate dynamic models from several pushing steps during the interaction with an object and choose the suitable pushing direction and velocity given a target pose. Inspired by this memory learning process, we choose an LSTM module as the main part of our dynamic model.

![LSTM]({{ site.url }}/data/images/self-adapting-LSTM.png)
{: style="width: 100%;" class="center"}
*Figure 2. The recurrent model*


To train the model, we collect motion trajectories of objects with different physical properties in [openai gym](https://gym.openai.com/). The pushing policy is trained by [DDPG with HER](https://arxiv.org/abs/1707.01495). Left video shows the trained policy on the prototypical object, the right video is the policy applied on objects with different properties including size, weight and friction.

![Data collection]({{ site.url }}/data/videos/self-adapting-data-collection.gif)
{: style="width: 100%;" class="center"}
*Data collection from gym*

Now let's see how the recurrent model perform. In the video you can see the data replay (two objects of size L and S in the left column) which is used to train the LSTM and the trained result of the network (two objects in the right column). The video shows 50 motion steps in which the first 5 steps are fixed used as the "warm-up stage" for the LSTM and the 45 steps after are the prediction results from the network. The network take the latest 5 history steps from the trajectory as input. The whole prediction trajectory is a recurrent sequence from the network output.

![Recurrent model performance]({{ site.url }}/data/videos/self-adapting-motion-prediction.gif)
{: style="width: 100%;" class="center"}
*Data replay and the performance of the recurrent model*

## Recurrent Model Predictive Path Integral

This part we elaborate how we use the recurrent model and how the model achieve the online self-adapting. In order to endow the original [MPPI](https://homes.cs.washington.edu/~bboots/files/InformationTheoreticMPC.pdf) with a memory mechanism, we add a history buffer $$\mathcal{H}$$ into the algorithm. Algorithm 1 gives details of the whole framework. The results from our real experiments prove the effectiveness of RMPPI.

![Algorithm]({{ site.url }}/data/images/self-adapting-algorithm.png)
{: style="width: 100%;" class="center"}
*Figure 3. RMPPI*


## Through domain randomization, transfer the model learned from simulation to real world.

The DDPG policy can be transfered to real environment directly. The shortcoming is that the determined policy can not be generalized to a different object.

![Data collection]({{ site.url }}/data/videos/self-adapting-model-free.gif)
{: style="width: 100%;" class="center"}
*DDPG policy on real pushing task*

By contrast, combining the proposed recurrent model and the RMPPI together can achieve generalization to push objects with different properties. In the video, all 1024 sampled trajectories are plotted, with the color implying the score from the RMPPI. Blue line is the optimization action.

![Data collection]({{ site.url }}/data/videos/self-adapting-model-based.gif)
{: style="width: 100%;" class="center"}
*Recurrent model with RMPPI*

Refer to our [paper](https://arxiv.org/abs/2007.13421) for more details.
