2026-10-02 20:39

Tags: 

### Vectors

Vector databases store high-dimensional vector embeddings which are unstructured data like text, images, audio files, and video files. Each vector has floating point values as and the amount of values a vector has is known as its number of dimensions. For example we have a vector with 10 values that would mean that we have a vector of 10 dimensions. These vectors define the similarity between objects in the vector space.

For example if we store a bunch of images and we take two images that are similar, their vectors will be close to each other in the vector space.

### Cosine Similarity, Dot Product, Euclidean Distance (L2)

Cosine Similarity is relatively easy to calculate metric that tells us how similar or different things are. Let's take an example of how similar the phrase "Hello World" is to "Hello". We see the word "Hello" and the word "World" once in "Hello World" and we se the word "Hello" once and see the word "World" zero times. This can be plot as a 2d graph where the X axis is "Hello" and the Y axis is "World". So "Hello World" will be plotted at the point (1,1) and "Hello" will be plotted at the point (1,0).

If we draw lines from the origin of the graph to the two points we see that there is a 45 degree angle between the two graphs, cos(45) = 0.71 which is our cosine similarity here.

| Phrase        | Hello | World |
| ------------- | ----- | ----- |
| "Hello World" | 1     | 1     |
| "Hello"       | 1     | 0     |

![[Database Systems/Images/Cosine Similarity.png]]

However even if the second word was something different like "Hello Hello Hello", the length changes but the angle is still the same so the cosine similarity is the same. So we can see here that cosine similarity only cares about the angle between the lines and not the length of the lines. The cosine similarity considers direction but does not consider magnitude. If there is no overlap of phrases like if one phrase is "Hello" and the other is "World" then we'd say that the angle between them is 90 degrees and that would be a cosine similarity of 0. When they exactly overlap it will be 0 degrees and a cosine similarity of 1. The equation that can easily calculate the cosine similarity is 

![[Cosine Similarity 1.png]]

This equation is great because we can't always graph things out, especially when we get to higher numbers of dimensions. 