+++
title = "Sparse AutoEncoders(SAEs) and compression"
weight = 2
+++

To find important features in A*x=b
There is another method than directly using Lasso regularization to solve this equation.

That’s where SAEs come in, Sparse AutoEncoders.
This is specifically for LLMs.   
So hop on.

Lets say I have 4 words in my input, so total size of input is 4* d_model ( lets say 512 in this case).
And thats lets assume for gemma-2b, so 18 layers in total. ( so 4*512*18 total numbers).

We pick a specific layers, and then we project that layers to a higher dimension ( so 512 dimensional vector projected to 2048 dimensional vector), and we make the 2048 dimensional vector sparse.
That’s what SAE does.

And here’s how we identify features through SAEs. We pass some inputs, and we pick up a layer, and then we reconstruct the activations of a layer by multiplying the projected dimension by decoder weights. 
So, we have an approximation of the real activation. And then we multiply the activation with the unembedding matrix( W_U) to get the logits of the target vocab. 
Here, we have an idea of what the next words are going to be based on manual inspection of the output vocab.
If then, we find some patterns, like for a similar set of prompts, a particular index in projected dimension is getting activated, and it is mostly producing tokens that are similar in semantics, then we have found a feature which gets activated on a specific prompt.

So, through SAELens, you just need to select the model, the layer you wanna probe, and the set of prompts, and see whether you can find a cool feature.

The important thing is you have to select the whole dataset, and it will by itself( its unsupervised learning) find the important features for each prompt
( The analogy is like training CNN - In CNN, we have dataset, and we want to find the weight matrix, which will decompose each picture into a category, which we have defined. Similarly, we can correlate the images dataset with the prompts dataset here, and like finding CNN weights matrix, we have to find weight matrix of the encoder and decoder of the SAE for each layer. And unlike CNN, we dont have to manually give different classes, as the method is unsupervised, it by itself finds clusters to which a certain activation might belong to)

> One important aspect is the details of the dataset, so the input is SAE is the d_model_dimension size of each token, and then try to reconstruct the d_model_dimension of that vector. So the whole prompt doesn’t have to do much here, except in calculating attention, but through SAE, only individual tokens go in. So we might find a particular feature is getting activated for a certain token, and then correlate it with the entire prompt.
> 

Neuronpedia- https://www.neuronpedia.org/

What it does it, it has already trained for different set of prompts for different models, so you can just input your prompt, and then it will try to find in its input prompts the prompts which are most similar to your query, and output the prompts, and see which layer’s activations that prompt activates, and in which category it falls into.
and then it will run the activations of that prompt of each layer and each token through the SAE’s encoder and decoder weights, and whether it greatly activates some specific activation( and then it goes through its training dataset, and sees which prompt resulted in activating the specific feature’s activation)

SAEs are time taking because of the number of training steps to find sparse solutions using L1.
And you have to do it for every word in the prompt.

I also tried steering with features found through SAEs, feels like magic the first time you do it, but then it makes sense, and you understand what really is going on.

Questions :
1) To find a particular feature, lets say if I input harmful prompts, so some feature( if it exists) should get activated,( a particular layers and a particular neuron).
→ to do this, I have to input harmful prompts, run SAE for every layer and every word( or heuristic take middle layers), and then see the outputs that SAEs reconstructed activation vector generates, and see whether it activates harmful words or not?

2) How exactly the loss function works, we put a restriction of L1 on the hidden layer of the SAEs, and it is still able to run it for every prompt, so the final SAE decoder which is constructed, is it zero for most, or is it full, and each index denoting a particular feature and there are a lot of features, and when I run a prompt, it just multiplies it with encoder and decoder weights and finds a feature. And what all the numbers in neuropedia mean

Okay, a rundown again: of whats going on in Neuropedia-
```
It has SAE_encoder and SAE_decoder for some layers( which it received on training on huge dataset, and making activation sparse).
```
Lets say we have sae weights for gemma-2b for layer 6.

So, when I input a prompt, it reaches till layer 6, and then each individual token in the prompt is multiplied by SAE decoder weights, and we see the coefficient of the hidden dimension( and then we segregate based on whose coefficient is the highest).
And once we identify, lets say index 15392 gets the highest activation ( 96) for this current prompt, we check in the training data - which training data caused it to give the highest activation.
And then we probe into that particular index 15392, and check which all prompts gave it high activation ( and the exact token number), and then we input the prompt and the token number in GPT, and it gives some response of what it might correspond to.

Now steering is different thing entirely, we have to check if we input the same prompt, what is the value of activation that it gives, and the coefficient should be the same
2 things in steering →
1) we are adding that token to any layer, to all positions, and we see that it is forced to produce that token
2) we have to check the effective coefficient number, so for that, check if I input the same prompt into neuropedia, what is the coefficient of the feature we are adding.

Next I am thinking if we can club adding steering vector at each layer to neuropedia’s getting activations’s coefficient out for a specific prompt