# Efficient coding and variational inference

Starts from a generative model with latent variables z:

$P(X) = \int P(X|\mathbf{z})P(\mathbf{z})dz$  

Recalling the definition of $D_{KL}(p||q) = E_p[log(\frac{p}{q})]$, we define the divergence between one distribution and the posterior of latent variables given the data



$D_{KL} [Q(\mathbf{z}|X)||P(\mathbf{z|}X)] = E_{z\sim Q}[log(Q(\mathbf{z}|X)) - log(P(\mathbf{z|}X))] = E_{z\sim Q}[log(Q(\mathbf{z|}X)-log(P(X|\mathbf{z})-log(p(\mathbf{z}))] + log(P(X))$   Rearranging, we obtain the well known ELBO


$$
P(X) - D_{KL}(Q(\mathbf{z}|X)||P(\mathbf{z|}X)) = E_{z\sim Q}[log(P(X|\mathbf{z})] + D_{KL}(Q(\mathbf{z}|X)||P(\mathbf{z}))
$$
Therefore, maximize the r.h.s is equivalent to maximize a lower bound to the true likelihood.

The classical approach of VAE's is the following: 

* the prior over the latent variables is usually assumed to be $P(\mathbf{z}) \sim \mathcal{N}(0,I)$

* the conditional distribution is often assumed to be a multivariate gaussian $P(X|\mathbf{z}) = \mathcal{N}(f_\phi(\mathbf{z}),\sigma^2I)$  where we assume that $f$ is a "decoder". Usually this is implemented as a NN

* also the posterior is approximated as a multivariate gaussian distribution:

  $Q(\mathbf{z}|X) = \mathcal{N}(\mu_X,\Sigma_X)$  where  again $\mu_X, \Sigma_X$ are obtained through a NN

The classical approach in this sense is given by using SGD from samples of X with loss function the rhs of Eq34 (with reparameterization trick, ecc..). Actually, a regularization term, if the conditional distribution is assumed to be gaussian, appear naturally as the variance (precision of the reconstruction). Of course this is strictly related to the kind of distribution assumed for the decoder.

The usual problem of this kind of models is that the loss function ignore the joint probabilty $p(X,z)$ . This, in presence of a sufficiently powerful decoder, may lead to generate latent variables that are not informative: every data point is mapped to the prior and the mutual information between the latent space and 

### From VIB: $\beta$ -VAE

A popular loss function for seeking an informative compression of the data $X$ into latent variables is the so called information bottleneck. We now show how in its unsupervised version, it lead to a loss function which is very similar to the ELBO, but with a regularization term.  We seeks a latent rapresentation $\mathbf{z}$ such that we maximize
$$
R_{IB} = I(\mathbf{z},X) - \beta I(\mathbf{z},i)
$$


where $i$ is the identiy of a specific element in the data set. Recalling the defintion of mutual information: $I(X,Z) = \int dx dz p(x,z) log(\frac{p(x,z)}{p(x)p(z)})$ . We can write a variational approximation for 

* the prior $p(z)\sim q(z)$
* the  variatonal decoder $q(x|z)$ 

We obtain an upper bound for the information bottleneck as
$$
R < \int dx p(x)\int dz p(z|x) log (q(x|z)) - \beta \frac{1}{N}\sum_i D_{KL}(p(z_{x_i}|x)||q(z))
$$
This is very similar to the ELBO obtained from a completely different perspective. 
The two approximations have different interpretations: in the first formulation we start from a generative model from a set of latent variables. Then we are forced to approximate the posterior $p(z|x)$ with a stochastic encoder.  In the information bottleneck approach instead, we look for an informative description of our data and we  model the decoding process with a variational distribution.

The regularization parameter, wherever it comes from, is the key to encourage more disentangled representations. Intutively, more weight we give to the divergence between the latent distribution and the prior, more we encoruage disentangled representations.

### Efficient Coding interpretation: cursed paper

The key idea is to consider the stimulus $x \sim \pi(x)$ encoded in a set of $J$ latent categories. The stimulus is encoded as:

 $r_j(x) = p_{\phi}(j|x)$  with the obvious contstraint $\sum_j r_j(x) = 1$ . $r_j(x)$ is naturally interpreted as a set of firing rates normalized . We can write the joint probabiltiy of a stimulus and a category $p(j,x) = \pi(x)p(j|x)$ . We assume then to have a stochastic decoder $\tilde{p}$ which, given a set of firing rates (or latent categories), output the probability of the stimulus $\tilde{p}_\theta(x|\mathbf{r})$ . We model the decoder as the collection of probability assigned to a stimulus given a latent category $\tilde{p}(x|r_j=1) = \tilde{p}(x|j)$  the joint probability of decoding a stimulus $x$ and having a latent category j is given by the product of

$\tilde{p}(x,j) = q_{\theta}(j)\tilde{p}(x|j)$ . $q(j)$ is the prior on the latent categories. Stated like this, the loss function of the $\beta-VAE$ reads
$$
D+\beta R = -E_{\pi(x)} [ \sum_jp_\phi(j|x)log(\tilde{p_\theta}(x|j)) + \beta D_{KL}(p_\phi(j|x)||q_\theta(j))]
$$


where the likelihood of the data given the latent categories is a "cross entropy".

The parametrization assumption is that the decoder tries to approximate the prior as a mixtureof gaussians given the latent categories

$\pi(x) = \sum_j q_j \mathcal{N}(x;\mu_j,\sigma_j)$ .Defining the stochastic decoder
$$
\tilde{p}_\theta(x|r_j=1) \sim  \mathcal{N}(\mu_j,\sigma_j)
$$
Deriving the loss function with respect to the parameters $\phi$, we obtain that the encoder is parametrized by the same parameters of the decoder
$$
\partial_\phi D + \beta R = \sum_j[(-log(\tilde{p}_\theta(x|j)+\beta(log(\frac{p}{q_j})+q_j))\partial_\phi p] =0\\
p_\theta(j|x) = q_j \tilde{p}_\theta(x|j)^{1/\beta}
$$
Note that in this formulation we can use the bayes theorem as $\tilde{p}(x|j) = \frac{\tilde{p}(j|x)\pi(x)}{q_\theta(j)}$ but $\tilde{p}(j|x) \ne p(j|x)$ 

 From this, we can compute the probability of the latent variables

$\tilde{p}(\mathbf{r}) = \Pi_j q_j^{r_j}$ . With the same criterium, we can also compute the posterior probability  of the decoded stimulus given a generic response  $\tilde{p}(x|\mathbf{r}) = \Pi_j (\mathcal{N}(\mu_j,\sigma_j^2)^{r_j}$ .

Summarizing, we have to find the parameters that minimize (36) , with the following definitions of encoder and decoder:
$$
p_\theta(r_j=1|x) = \frac{q_jexp(-\frac{1}{\beta}(log(\sigma_j)+\frac{(x-\mu_j)^2)}{2\sigma_j^2})}{\sum_{j'}q_{j'}exp(-\frac{1}{\beta}(log(\sigma_{j'})+\frac{(x-\mu_{j'})^2)}{2\sigma_{j'}^2})}\\
\tilde{p}_\theta (x|r_j=1) = \frac{1}{\sqrt{2\pi\sigma_j^2}}exp(-\frac{(x-\mu_j)^2}{2\sigma_j^2})
$$
Substituting (39) in (36), we obtain that the loss function become $L=D+\beta R = - \beta \int dx \pi(x) log(\sum_{j'}q_{j'}exp(-\frac{1}{\beta}(log(\sigma_{j'})+\frac{(x-\mu_{j'})^2)}{2\sigma_{j'}^2}))$ , which is simply a "free energy" .

The minimization is done using an expectation maximization algorithm. Parameters are initialized $\theta$  and the loss function is computed. Then parameters ore updated taking the gradient of the loss function.  The minimization is done over the entire dataset. To recall: we compute the loss function using the parameters $\theta^*$. Then, we have to solve the constrained optimization problem $ min L^*= L-\lambda{\sum_jq_j }$. Given N, the number of data points, at each step the parameters are updated according the following rules:
$$
\frac{\partial}{\partial q_j}L^* =0 \rightarrow q_j = \frac{\sum_{n=1}^N p(r_j=1|x_n)}{N} \\
\frac{\partial}{\partial \mu_j}L^*= 0 \rightarrow \mu^*_j = \frac{\sum_n x_n p(r_j=1|x_n)}{\sum_np(r_j=1|x_n)} \\
\frac{\partial}{\partial \sigma_j}L^*= 0 \rightarrow \sigma_j^{2^*} = \frac{\sum_n (x_n-\mu_j)^2 p(r_j=1|x_n)}{\sum_np(r_j=1|x_n)}
$$
In a encoding-decoding scheme, an input $x$ is encoded in a set of weights $\mathbf{r}$ and then decoded again according to $\tilde{p}(x|\mathbf{r})$ 

Remarks:

* the encoder is obtained from the decoder, and is supposed to be optimal
* this naturally give rise to a "pure" coding , with monomodal and equally shaped tuning curves
* actually, I don't understand very well why should be a VAE. The encoder is compute as optimal for the given decoding scheme



## Summarizing and todolist

The hypothesis of efficient codinf has been tackled recently using approaches derived from machine learning. One of them is the information bottleneck method, which in its unsupervised version is very close to the architecture of variational autoencoder. This architecture is used to extract a "meaningful" representation of the incoming stimulus, where the world "meaningful" has to be defined. The neural population  $r$ is assumed to minimize a loss function, which given the stimulus $x$ , is given by 
$$
\mathcal{L} = D + \beta R = - \int dx p(x)\int dr q(r|x) \log(p(x|r)) + \beta \int dx p(x)\int dz q(z|x)\log \frac{q(z|x)}{\tilde{q}(z)}
$$


* Define a model of interest

* Define a stimulus: 

  * Images : high dimensional

    Analytically, work has been done using the linear model with a stimulus $x \sim \mathcal{N}(0,\Sigma_{xx})$  and linear transformations(with and without bottlenecks) that preserve the stimulus properties

    









1. 