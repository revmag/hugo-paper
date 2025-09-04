+++
title = "State of Mechanistic Interpretability"
weight =1
+++


So I deep dived into what is going in the field of mechanistic interpretability( it seems really cool, to be able to know how attention is calculating all these amazing similarities from so far, and how it is able to make so much sense).

What I think is happening -
It seems to me, LLMs mimic a lazy person, who is very good at pattern patching, so it learns rules/pattern of some kind implicitly, and its not entirely perfect knowledge like code, but it wants to converge to that. I got this idea by looking at the chess paper, where they analyzed how it predicts the next move of chess game, without being fed the rules explicitly, or what each piece does, but after training for several games, it was able to predict the next move, and also tell some of the rules, Some for calculation, there is this paper, which analyzed how LLMs are able to do 5-6 digit addition, it was amazing, it tried to show they are learning the rules of addition somehow( like addition at ones place, and then carry on if sum exceeds 10), by differentiating the dataset into different catagories (like whether carry sum is needed or not).
```
`So its a pattern matching algo, which implicitly learns the privileged basis for the problem at hand and tries to solve similar things`
```

<aside>
💡 Goal is to find the basis function of any task, and use that to train the model ( I saw a post where someone as OpenAI described their day to day job, it was to solve puzzles, and then tell model how they solved it, explicitly)

</aside>

Here’s an overview of what I read- 
1) There are various methods being used, and different methods work for different cases. Some things which are quite accepted now are - 
Induction heads- only found in 2L models, 
Features being present which tell the output came to be( and we can insert different features in different results, PS -Hello Golden Gate Claude) ,
Zero ablation of heads - we make the attention score of heads 0 and see whether this significantly changes the output or not,
LogitLens - this is nothing new - but it is remarkable in the sense that we have some idea that transformers are able to get back at identity after some layers 
TranformerLens - we study each individual head of a toy model - and try to find any relations between the different heads, induction head, prev token head came from this

And there is a whole lot of interesting things happening like 
Catastrophic forgetting - when model is fine tuned on new data, it seems to forget some of the old data
The methods for diving deep are causal  tracing, ablation, looking at KV cache, looking at weights, feature weights, steering weights, existence of privileged basis vs superposition basis.
Double descent( grokking), when model performance improves, gets bad and then improves again with the increment of training data - it was good till the point improving and getting bad, as this explains overfitting on training data, but then again it suddenly getting better.
( The graph below)

<div class="home-hero">
  <img src="/images/interpretability.png" alt="Me" width="1800">
  <div class="home-quote">
</div>
</div>

Then there are some cases of whether the LLM is actually just memorizing things from its training data( a paper showed it gave input the first sentence of harry potter, and it produced the entire page verbatim)

Some further questions like are more layers better( seems true with the analysis of LogitLens, at the final layers, the model somehow seems to be getting idea of what the actual output can be)
Whether phase change is inherent part of how models learn?

And further analyse Edge cases where
linearising LayerNorm breaks, activation patching breaks, causal scrubbing breaks, ablation breaks, composition scores break, eigenvalue copying score break

Most of the research is doing inference on models and see what the weights are, maybe doing fine tuning on these things can give us further answers, and inserting our own feature vectors( Hello, Claude’s Golden Gate bridge)

Links:
1) https://www.lesswrong.com/posts/AcKRB8wDpdaN6v6ru/interpreting-gpt-the-logit-lens
2) https://adamkarvonen.github.io/machine_learning/2024/01/03/chess-world-models.html
3) https://arxiv.org/abs/2310.15213 - function vectors
4) https://www.alignmentforum.org/posts/jGuXSZgv6qfdhMCuJ/refusal-in-llms-is-mediated-by-a-single-direction - refusal
5) https://philipquirke.github.io/transformer-maths/2023/10/14/Understanding-Addition.html - this explains addition by privileged basis
6) I got many of my ideas and some possible research directions by looking at the problems people are trying to solve( from 200 open problems in Mech Interpretability)  - https://docs.google.com/spreadsheets/d/1oOdrQ80jDK-aGn-EVdDt3dg65GhmzrvBWzJ6MUZB8n4/edit#gid=0
7) https://colab.research.google.com/drive/1NYjR3tjOiDJ2v8nv3mhrph-_IM4p9goS?usp=sharing- was doing the exercises in Neel Nanda’s guide to mech interpretability on Function vectors and model steering