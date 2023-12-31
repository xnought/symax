# `symax`

Symmetric selection of values in attention that is not shift invariant. (this is an experiment) If I'm not using softmax for probabilties, why would I care about shift invariance? That's one assumption. The other came from this Evan Miller dude which is that the limits don't go to one for extreme values for softmax which creates issues.

Inspired this one dudes tweet on why softmax is giving weirdness in attention

$$\text{Attention}\left(Q, K, V\right) = \text{softmax}\left(\frac{QK^T}{\sqrt{d}}\right)V$$

In summary, the fact that you need to choose between discrete entities into a probability forces you to weigh stuff high even if it isn't pertinent.

In the case where the query and key shouldn't weigh any values, what do you do? Apparently it was found that certain tokens have extreme spikes (like space tokens) which mess things up.

The dude thinks a +1 fixes a lot of this in softmax.

https://www.evanmiller.org/attention-is-off-by-one.html

## Why not just remove e at this point

Intuitively, why not make this symmetric by taking the `abs` instead of exponentiating the vector for softmax?

Then, the attention mechanism would be sensitive to values in either direction and still have a limit that is favorable to extremities

$$\text{symax}(x, \eta)_i = \frac{|x_i|}{\eta + {\sum | x_j |}}$$

-   Where |s| represents the absolute value of a number s. I guess you could interpret this as the $||x_i||_1^1$ (1-norm) so you could extend this to other norms probably.
-   Where $\eta$ is a scalar number that defaults to 1, (so the limit at infinities goes to 0) or learnable parameter (haven't tested this).
-   Note this is not a probability distribution since sum < 1

(also begs the question if other norms would be more favorable and what type of behavior you might get)

```python
def symax(x: tensor, eta=1, dim=0):
    sizes = torch.abs(x)
    return sizes / (eta + sizes.sum(dim=dim))
```

## Limits

For resonable computable large values, if all the $x$s are extreme, will tend towards 0. For example all values in $x$ are very very negative, will tend, symax will tend towards 0.

## Sym Attention

With a default of $\eta=1$ (not shown)

$$\text{SymAttention}\left(Q, K, V\right) = \text{symax}\left(\frac{QK^T}{\sqrt{d}}\right)V$$

## TODO

-   Train a model with `SymAttention` and see what kind of values I get!
-   Compare with regular `Attention` and with a modified $\text{softmax}_1$ version too

Test performance too!
