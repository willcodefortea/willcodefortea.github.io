---
title:  "Bloom Filter"
categories: what-is
layout: single

header:
  overlay_image: /assets/images/what-is/bloom-filter/lilly.jpeg
  # overlay_filter: 0.6
excerpt: A bloom filter is a probabilistic data structure used to check set membership in constant time and space. They're pretty cool.
---

Often when we consider the limiting behaviour of a function, usually represented in $O()$ (Big O) notation, we are talking about the time complexity. However, the space that an algorithm takes to perform is also an important consideration and can have practical implications. Your code will never return if it requires more memory than is available for it to execute.

{% include figure image_path="/assets/images/what-is/bloom-filter/Comparison_computational_complexity.svg.png" alt="Plots of various functions on a graph." caption="Graphs of functions commonly used in algorithm analysis. By Cmglee - Own work, CC BY-SA 4.0, https://commons.wikimedia.org/w/index.php?curid=50321072" %}{: .img-zoom}

One very simple algorithm is that of membership, that is to say if an item is present, or more formally:

$$\exists \ x \in X $$

> This is a fancy way of saying "does element x belong to set X" and a way of demonstrating that I'm still learning LaTex. 👀

One might expect this to be quite trivial but it's a great example of an algorithm where the data structure that's used dictates its performance.

## An array

The first solution that comes to mind may be to use an array to store your set and perform membership tests against it. Computers are pretty fast these days, so even if your data set contains a few thousand items (and you're not doing this operation very often) you probably won't notice any issue. However, membership tests on array-like data structures occur in $O(n)$ time. That is to say, the more stuff you put in them, the slower it'll get.

## Enter the Set

You might be thinking "well I wouldn't do that, I'd just use a Set", and you dear reader would be showing your smarts. Or that you've been bitten by this before. If you require to do this operation frequently, or your dataset is sufficiently large, then $O(n)$ will no longer be sufficient. You need something you can rely on, you need something _constant_.

Sets usually do not preserve the order of items that they contain, but share the property that membership tests are done in $O(1)$ time. (How they do this is quite cool; in python a Set is basically a hashmap where the key and value are the same thing + some extra operations.)

Okay great, we've got the best possible solution. What's next?

Well, what about space complexity. What if the thing you want to check doesn't neatly fit into the allocated space your algorithm must run in? Difficult to imagine I know, but there are several fairly common cases where this can become a problem:

1. You're in a resource starved environment (like serverless)
2. This will run in the browser on a user's device, where it's hard to predict what they'll have available
3. You have a highly concurrent system; this may not be a problem for one thread but won't work across all of them
4. You're convinced there must be a better way.


## Bloom Filters

A bloom filter is a probabilistic data structure that does not store the item, cannot guarantee that an element is in a set, but can guarantee when it isn't. This trade off in what it can do allows it to be extremely efficient and be constant in both time AND space. So how does it work?

To begin, we need to know the size of the filter we want to implement and a number of hash functions we want to use within it. Each hash function should take a given input and map it to a bit in our filter. When we add an item to filter, we pass it to each hash function and set the corresponding bit to 1.

{% include figure image_path="/assets/images/what-is/bloom-filter/bloom-filter-population.jpg" alt="Bloom filter population" caption="A bloom filter of length 16 with three hash functions marked red, green and blue. After adding items $x$, $y$, and $z$, we notice that $red(x) == green(z)$!" %}{: .img-zoom}

To test for membership, we pass a value through the hash functions and check the corresponding bits, if any of the bits are 0 then the item is definitely not in the set, if it were then those bits would all have been set to 1.

But, if they're all 1 then the item is only _probably_ in the set. Why only probably? Well, a bloom filter _relies_ on hash collisions to reduce the space required to track item membership. In the example above we see that $x$ and $z$ both share a bit. What if we added more items? What if we kept going until all our 16 bits were 1? Wouldn't that indicate that _anything_ was in the set? Yes, yes it would.

The _size_ of the bloom filter and the number of hashers that are used are crucial, and dictate the overall rate of false positives (often referred to as an error rate). As a rule of thumb, ~10 bits per element will supply a 1% false positive rate, and the optimal number of hashes is given as:

$$ {m \over n} \ln2$$

This is great to be aware of, but usually not something we as software engineers would expect to implement ourselves.

> The eagle-eyed among you may have noticed that if we rely on collisions, then once an item is added to the filter there is no way to remove it. (If required however, there are more advanced bloom filters that do allow this.)

So, if ever you're required to check if an email already exists in your system at sign up time for your super popular new webapp, or you happen to be building a popular database engine and want to avoid expensive disk lookups, consider using a bloom filter! Not only will they do the job, but they're pretty fuckin' cool.


<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$'], ['\\(','\\)']],
    processEscapes: true
  }
});
</script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
