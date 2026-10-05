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


The dot product is the **numerator** of the cosine similarity equation. It's calculated by multiplying each pair of corresponding elements and summing the results:

![[Dot Product.png]]

So this means that if we use or example earlier of "Hello" and "Hello World" that would give us a dot product of 1.

The dot product has a geometric interpretation of **|A| × |B| × cos(θ)** — it's the cosine of the angle **multiplied by the lengths of both vectors**. This means the dot product considers **both direction and magnitude**, unlike cosine similarity which only cares about direction.

So if we compare "Hello World" (1,1) with "Hello Hello Hello" (3,0):

**Cosine similarity**: Both point along the X-axis → angle = 0° → similarity = 1.0 (they're "equally similar" directionally)

**Dot product**: (1×3) + (1×0) = 3 vs (1×1) + (1×0) = 1 → "Hello Hello Hello" scores **3× higher** because it's a longer vector


Euclidean distance is the **straight-line distance** between two points in the vector space.

![[Euclidean Distance.png]]

L2 **penalizes magnitude differences** — even though the two vectors point in the same direction, one is much longer, so L2 says they're far apart. This is the opposite of what cosine does.

"Hello World" (1,1) vs "Hello Hello Hello" (3,0):

**Cosine similarity**: Both point along the X-axis → angle = 0° → similarity = 1.0 (identical direction)

**L2 distance**: √((1−3)² + (1−0)²) = √(4+1) = √5 ≈ **2.24** (very far apart)


Cosine Similarity should be used in most vector search use cases, for example doing semantic search / RAG with text embeddings, the vector length is arbitrary, or when the embedding model was trained with a cosine loss.

Dot Product should be used mainly when magnitude is a feaure, for example if the vectors are already L2-normalized (then it's identical to cosine, just faster because no division is needed). It can also be used when the vector length encodes something meaningful (confidence, popularity, user activity level in a recommendation system) and also if our model was trained with a dot product / inner-product objective.

Euclidean Distance should be used mainly when absolute position matters for example if we're doing clustering (k-means, DBSCAN are defined in L2 space), when vectors represent spatial/physical measurements (GPS, sensor readings, pixel values), if our model was trained with a contrastive/Euclidean loss, and if we need a true distance (satisfies triangle inequality) for downstream math like bounding boxes or outlier thresholds.

If your vectors are unit-normalized, all three metrics produce identical rankings. So it's not about which metric is best, it's about wheteher or not the model normalizes and whether or not magnitude carries meaning.