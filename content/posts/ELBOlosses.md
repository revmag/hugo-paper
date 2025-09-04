+++
title = "ELBO losses and Posterior Approximation"
weight = 4
+++

The ultimate goal is to maximize the evidence( or the log of the likelihood)
which is similar as minimizing the negative log likelihood, and hence we compute gradients of the loss( which is negative log likelihood), to move in a direction where loss gets reduced).
Loss is not a fixed quantity, loss depends on parameters, and hence we can tune the parameters, so that we can get a reduced loss.

When using Invertible Neural  network, scale and shift network
→ to find loss here:
Just calculate log likelihood of data to get the loss.

Here’s the whole interconnection 
( Loss is log likelihood of data P(D).
Here’s another way to get at it:

We want to minimize the distance between the true posterior and our assumed distribution.
( KL(q||p) ⇒( KL(q||p*) + log(P(D)).
We want KL(q||p) to be as low as possible.
The main takeaway is log(P(D) = KL(q||p) -KL(q||p*)

And we want to minimize KL(q||p), that is equivalent to maximizing  -KL(q||p*). a.k.a ELBO, we kind of assume KL(q||p) is =0 for the best case, and hence -KL(q||p*) is the lower bound of the evidence( log(P(D)) is called evidence), hence its called ELBO.

Here’s the catch:

1) In Invertible Neural Network, we can just calculate the loss directly:
log(P(D), as its being transformed to another dimension, we don’t have to use ELBO and all.

2) In Diffusion, we can’t directly calculate log(P(D)).
Hence we have to use ELBO here, to get closest to Log(P(D)), which tells us the loss.

3) In Bayesian Neural Network, same as diffusion,
We use ELBO, but this time, we use Variational Inference Mean filled method to get direct Gaussian Estimates of the parameters, we kind of do direct solving of the equations, unlike using gradient descent to get closer to the optimum value.