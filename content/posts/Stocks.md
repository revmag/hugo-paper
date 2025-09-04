+++
title = "Stock Market Prediction with RNN"
weight = 7
+++


In trying to predict the closing price of a stock, if we have opening, high, and closing price for several days, I encountered a few questions.

1. How should my input be, and what should my output be?

—>  I knew all the backprop steps in calculating RNN, but I was not able to figure out, what it means to apply sequence to sequence modeling here, how many days should determine the output of the next days( like should a sequence of 7 days determine the output of the 8th day, or lesser or more, or should only 1 day determine the output of next day, and i should calculate my output prediction of all the days, and how should my hidden layer be, do i calculate output at every day, and Hoping to pass hidden state value at every step)

1. What does sequence to sequence modeling mean, like let's say the output of 43rd day, in calculating that, should the hidden layer incorporate details of the 1st day, or should it just be a function of weights which got updated

 Some of the answers which i got ( i mostly arrived at these by reading about how karpathy said RNNs are ridiculously easy, and you just have to decide number of features, sequence length, and no of cycles, i will be telling what these all mean).

So the basic theory is this:

You have to decide how many days should be taken into account to calculate output of the next day( let's say 7 days should decide the output of the 8th day).

And let's say 3 features are there in the input ( opening price, closing price, highest price) and output is just the closing price.

For this, the input will be 3*1. And the length of sequence is 7 days to calculate 8th day( what this means is, backpropagation for updation of weights will happen after 7 days, that's it).

So day 1…7 will predict 8th day, and weight updation happens. 

Now, hidden layer vector resets to zero, and weights get updated for W_IH( input to hidden layer), W_HH( hidden to hidden layer), W_HO( hidden to output layer).

And this updated weight matrix is used to take input for days 2…8, and calculate 9th day and so on.

Tldr: we have to determine the length of sequence( for backprop), and inputs are 1d vector ( number of features), hidden state vector can be anything, output can be anything. Sequence length is number of days after which backprop happens, hidden state vector resets after 7th day( but the weights get updated).

Code : https://github.com/revmag/Deep_Learning/tree/main