# Intra-Layer Attention along with Inter-Layer Attention

This is a similar seq-2-seq model as [this](https://github.com/JRC1995/Abstractive-Summarization) ,used on [Amazon-Fine-Food-Dataset](https://www.kaggle.com/snap/amazon-fine-food-reviews/data) for abstractive summarization
(and should be usable on translation data for machine translation). 

The main difference is in the encoder and decoder units. This model is inspired from the idea of [recurrent residual attention](https://arxiv.org/abs/1709.03714). 

The RRA model adds a weighted summation of K previous hidden states with the standard recurrent function. 
However, unlike the standard attention where the attention weights are computed from the context by a scoring function, here (in the RRA model), the weights are randomly initialized.

So, I implemented the same standard layer-wise attention mechanism function to calculate the attention weights for previous
hidden states. This can be said to be <b>intra-layer attention</b>.

Note: There may have been previous works on intra-layer-attention, but I didn't refer to any such work while making this implementation.

# Intra-layer-attention for Encoder

I used [global attention](https://nlp.stanford.edu/pubs/emnlp15_attn.pdf) for the basic attention mechanism. 

At each time step of the encoder, a <i>candidate hidden state</i> is created by the standard RNN method (but with elu activation).

>candidate_hidden = elu(input * wxh + previous_hidden_state * whh + Bias)

Now for this intra-layer-attention, all the previous hidden states can be considered as analogous to the encoder hidden states for the layerwise-attention (inter-layer attention - the one that is commonly used), and the candidate_hidden_state can be considered as analogous to the current decoder hidden state for the same (inter-layer attention). 

All the previous hidden states are then scored by the scoring function taking the current candidate hidden state into account.

The scoring function, I used is: 
> hs * Transpose(ht)

ht = traget hidden, and hs = source hidden. 
In this context, ht will be the current candidate hidden state, and hs will be the list of all previous hidden states.

Similar to the standard inter-layer global attention mechanism, a context vector is created as a result of the weighted summation of all the previous hidden states, and then an 'attended hidden state' is created by taking both the context vector and the candidate hidden state into account (following the formula for creating ht_dash as given in [here](https://nlp.stanford.edu/pubs/emnlp15_attn.pdf))

This intra-decoder-layer attended hidden state is considered as the final hidden state for that timestep. 

Finally, we store the computed hidden state in the list of previous hidden states for computing the next hidden state in the next time step.

<b>Possible Improvement</b>: Slightly change (using some proper function) previous hidden states that were attended taking the current context into account. This is based on the intuition that while reading a text, previously read words can take on new meaning from context of the current words. Often we may look back previous words to make sense of the current word. That can be somewhat analogous to the intra-layer-attention - we attend to certain previous words in the text (or in our memory) to make sense of the current words. But, as we attend to previous words, the previous words can make more sense too, from the context of the present words. Which is why, updating previous hidden states based on current hidden states may be something to be explored (if not already explored). We already have bi-directional RNNs (which is what I used here, too) which are based on similar intuitions - that future contexts matter. But, this proposed improvement may eliminate the need for backward RNN, require less parameters (and computational power), and may learn to update only the relevant hidden state (previous context). However, I am unsure, if this truly will make too much of a difference.  

# Intra-layer-attention for Decoder

At each time step of the decoder, a <i>candidate hidden state</i> is created by the standard RNN method (but with elu activation).

>candidate_hidden = elu(input * wxh + previous_hidden_state * whh + Bias)

The initial decoder hidden state is the first bi-directional encoder hidden state and the initial input for the decoder is a word vector representing <SOS> - start of sentence for decoder. 

The previous decoder hidden states are scored in context of the current candidate hidden satte using a similar attention scoring function. With the weighted (wieghts = scores) summation of the previous decoder hidden states, the decoder intra-layer context vector is created. 

By linearly transforming the concatination of the context vector and the current decoder hidden state, the intra-layer-attended hidden state is created. 

The intra-layer-attended hidden state is chosen as the final hidden state of the current timestep.

By linearly transforming this hidden state a probability distribution for the output token is created. 
The GloVe embedded word vector in the vocabulary, which represents the most probabilistic prediction for the output token
becomes the decoder input for the next timestep.

And so the loop continues until all the output tokens are predicted. 

# Is it any better?

Can't say. For now, I don't currently have access to much (computational) resource for training and testing this under a reasonable time.

Ultimately, this is pretty much a toy implementation, that will work only for batch_size = 1. No regularization is implemented either. 

Example output:

>Iteration: 3
>Training input sequence length: 32
>Training target outputs sequence length: 2
>
>TEXT:
>great taffy at a great price. there was a wide assortment of yummy taffy. delivery was very quick. if your a taffy lover, >this is a deal.
>
>
>PREDICTED SUMMARY:
>
>cough cottage
>
>ACTUAL SUMMARY:
>
>great taffy
>
>loss=10.4148

Nothing to expect here. This is just the result of the 3rd iteration (i.e trained only on 3 data pairs).
I haven't run this model for more than a couple of iterations.  

I haven't used any evaluation metric (like BLEU) either, because there's not much point in evaluating these low quality predictions. These are on my TO-DO list if I later find oppurtunity for proper training and testing.

Interesting paper to try out later: https://arxiv.org/pdf/1705.04304.pdf

