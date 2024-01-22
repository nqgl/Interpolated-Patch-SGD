# Gradient Descent Over Interpolated Activation Patches for Circuit Discovery
### Glen Taggart

for gpt-2-small there are 
$h \sum\limits_{l=0}^{n - 1} l h = n (n - 1) / 2 * h ^ 2 = 9504$
 "edges" between attention heads which could be patched. This assigns a learnable coefficient to each of them (as well connections from the corrupted residual stream to the attention heads and from the head to the output), and then does gradient descent over all of them as a method for circuit discovery that I'm exploring. Edges are like: (src-layer, src-head) -> (dest-layer, dest-head) 


The psuedocode (glossing over the output coefficients and residual input coefficient) for what is calculated is like:
activations from normal vs corrupted run: 
```
W : edge weights
forward : normal model forward to get cached values for heads
forward_patched : a patched forward pass with $head_l_h$ values inside the curly braces patched into head $h$ of layer $l$ and values in brackets corresponding to the layer and head index to return (if none is provided, return the logits)

clean <- forward(input) 
dirty <- forward(corrupted input)
for i $\in {0, 1, 2, ..., 12}:
  for j $\in {0, 1, 2, ..., 12}:
    patched_i_j = forward_patched(input)\{
      for l $\in {0, 1, 2, ..., i - 1}
      for h $\in {0, 1, 2, ..., 12}
      head_l_h = clean_i_j * (1 - W_i_j_l_h) + patched_l_h * W_i_j_l_h
    )\[i, j\]

out = forward_patched(input){
  for l $\in {0, 1, 2, ..., 12}:
    for h $\in {0, 1, 2, ..., 12}:
      head_l_h = patched_l_h
}
```

then out is used to calculate a loss and gradient descent can be performed over $W$



$$
O\left(
  h \sum\limits_{l=0}^{n - 1} l h 
\right)
=O\left(
n^2 h^2 
\right)
$$


The naive implementation of this requires O(n^2 h^2) forward passes if I recall correctly. However you can do parallel src-heads within the same layer together by stacking them into a batch dimension, which brings it down to O(n^2 h) forward passes. This is pretty absurd but has been nagging at me to give it a try for a while. 

My plan was to start by implementing this just to clarify some things for myself, then have it require an utterly unworkable amount of vram, after which I would start by modifying the technique to first prune the graph by learn ((head,layer)->(layer)) edges prior to doing full edge processing on the subgraph produced by that process. However, it's actually just fully runnable currently! I still think these improvements would be a good idea, as running a single iteration takes 10-30 seconds and at a batch size of 8 it uses ~15gb of VRAM. It's very slow. As one would expect for something that requires O(n^2 h^2) forward passes of gpt2 per optimization step...

Nevertheless, it works somewhat at finding IOI edges! and manages to fit in vram! Not fast at all though.

It recovers some of the IOI circuit and some false positives, I haven't gotten to hyperparameter tune it enough to fully assess if this is capable of doing a decent job discovering circuits. Current performance is not competitive.




