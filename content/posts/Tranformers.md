+++
title = "How transformers work, Encoder Decoder difference, How Inputs gets passed"
weight =5
+++

I was confused for a very long time what the actual difference between encoders and decoders are, and how inputs gets processed by each layer( yes, I know the usuals like decoder only has casual mask, and it can’t look ahead, these all sound nice, but I want to know what exactly is happening over there)

So, a tldr:
1) Encoders take embedding of each word, and does some processing on each word and passes it.
Ex - A 4 sentence word, so 4  - 512 dimensional embedding, after many layers of encoders, we see 4  - 512 dimensional embedding only.
2) Decoder can do 2 things, depending on whether gold output is available or not( inference time vs training time)
If gold output is available( during training time) - it will similarly take take 4 - 512 dimensional embedding, and after various layers, calculate a final 4 - 512 dimensional vectors. The interesting thing is in decoder, attention is calculated twice, once between the gold output upto that index, and then between the query of the current index, and the keys and value of the final output of the encoder. We can do it all at once by using causal masking.
If gold output is not available ( during inference time) - it will take only 1 - 512 dimensional vector and further refine it to arrive at one final 512 dimensional vector( it will calculate attention upto the outputs produced upto that index, and then again calculate attention with it being the query matrix, and the keys and value matrix coming from the final encoder layer)

So, difference is mainly how many vectors get passed at each layer, and how many times attention is being calculated.

And after this, the final 512 dimensional  vector is feed forwaded to tgt_vocab_size, to produce score for each word, and then softmax taken, and various sampling schemes can be applied here, like greedy, beam search, to get the desired result.

Also interesting is words really hold no significance, we can really establish some really beautiful results with a bunch of numbers.