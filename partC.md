# Q1: Conceptual Question

**K-Means is often called a 'greedy' algorithm. What does this mean? Can it get stuck in local minima? How does K-Means++ initialization help?**

K-Means is considered greedy because it makes locally optimal choices at each iteration without considering the global optimum. During the assignment step, each point is assigned to the nearest centroid, and during the update step, centroids are recalculated as the mean of assigned points. This myopic approach can indeed get stuck in local minima.

Yes, K-Means can get stuck in local minima. The final clustering depends heavily on the initial centroid positions, and poor initialization can lead to suboptimal convergence where the algorithm finds a local rather than global optimum.

K-Means++ initialization helps by:

- Spreading initial centroids more intelligently across the data space
- Selecting the first centroid randomly, then subsequent centroids with probability proportional to squared distance from existing centroids
- Reducing the likelihood of poor initialization and improving convergence quality
- Typically achieving better clustering results and faster convergence compared to random initialization

## Q2: Coding Question

**Implement K-Means from scratch using only NumPy. Your function kmeans(X,k,max_iter=100) should return cluster labels and centroids.**

```python
import numpy as np

def kmeans(X, k, max_iter=100, tol=1e-4):
    """
    K-Means clustering implementation from scratch.
  
    Parameters:
    X : array-like, shape (n_samples, n_features)
        Input data
    k : int
        Number of clusters
    max_iter : int
        Maximum number of iterations
    tol : float
        Convergence tolerance
  
    Returns:
    labels : array, shape (n_samples,)
        Cluster labels for each data point
    centroids : array, shape (k, n_features)
        Final cluster centroids
    """
    n_samples, n_features = X.shape
  
    centroids = kmeans_plus_plus_init(X, k)
  
    for iteration in range(max_iter):
        distances = np.linalg.norm(X[:, np.newaxis] - centroids, axis=2)
        labels = np.argmin(distances, axis=1)
  
        new_centroids = np.zeros((k, n_features))
        for i in range(k):
            if np.sum(labels == i) > 0:
                new_centroids[i] = X[labels == i].mean(axis=0)
            else:
                # Handle empty clusters by reinitializing randomly
                new_centroids[i] = X[np.random.choice(n_samples)]
  
        if np.linalg.norm(new_centroids - centroids) < tol:
            break
  
        centroids = new_centroids
  
    return labels, centroids

def kmeans_plus_plus_init(X, k):
    """K-Means++ initialization algorithm."""
    n_samples = X.shape[0]
    centroids = np.zeros((k, X.shape[1]))
  
    centroids[0] = X[np.random.choice(n_samples)]
  
    for i in range(1, k):
        distances = np.array([np.min(np.sum((X - centroids[j])**2, axis=1)) 
                            for j in range(i)])
        probabilities = distances / distances.sum()
  
        centroids[i] = X[np.random.choice(n_samples, p=probabilities)]
  
    return centroids
```

## Q3: Analyze Question

**Your K-Means with K=5 gives silhouette score 0.25. Is this good? What would you investigate next?**

A silhouette score of 0.25 is moderate to poor. Silhouette scores range from -1 to 1:

- 0.71-1.0: Strong structure
- 0.51-0.70: Reasonable structure
- 0.26-0.50: Weak structure
- ≤0.25: No substantial structure

A score of 0.25 suggests weak cluster structure, indicating the clustering may not be meaningful.

**Next steps to investigate:**

1. **Optimal K determination**: Use elbow method, gap statistic, or evaluate silhouette scores for different K values (2-10) to find the optimal number of clusters.
2. **Data preprocessing**:

   - Check for feature scaling needs (standardize/normalize features)
   - Examine feature correlations and consider dimensionality reduction
   - Remove outliers that may distort cluster boundaries
3. **Algorithm alternatives**:

   - Try different initializations (multiple runs with different seeds)
   - Consider hierarchical clustering for comparison
   - Evaluate DBSCAN for density-based clustering if clusters have irregular shapes
4. **Cluster validation**:

   - Visualize clusters using PCA/t-SNE for 2D/3D representation
   - Analyze cluster characteristics and separation
   - Consider domain knowledge to validate cluster meaningfulness
5. **Data quality assessment**: Check if the data actually has distinct cluster patterns or if it's more naturally continuous.
