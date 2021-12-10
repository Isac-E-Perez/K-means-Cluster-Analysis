# K-means Cluster Analysis

### About:
Clustering is a broad set of techniques for finding subgroups of observations within a data set. When we cluster observations, we want observations in the same group to be similar and observations in the different groups to be dissimilar. This is because there isn't a response variable, this is an unsupervised method, which implies that it seeks to find relationships between the *n* obserations without being trained by a response variable. Clustering allows us ot identify which obervations are alike, and potentially categorize them therein. K-means clustering is the simplest and most commonly used clustering method for splitting a dataset into a set of k groups. 

### Note: 
There will be five points to the results portion. 
1. **Libraries Needed**: The libraries needed to start the project. 
2. **Data Preparation**: Preparing the data for cluster analysis.
3. **Clustering Distance Measures**: Understanding how to measure differences in observations.
4. **K-Mean Clustering**: Calculations and methods for creating K subgroups of the data. 
5. **Determining Optimal Clusters**: Identifying the right number of clusters to group the data.

### Results:

**Libraries Needed**

The following packages were used in this project:
```python
library(tidyverse) # data manipulation
library(cluster) # clustering algorithms
library(factoextra) # clustering algorithms & visualization
```

**Data Preparation**

To perform cluster analysis in R, generally, we have three conditions on how the data should be prepared.
1. Rows are observations (individuals) and columns are variables.
2. Any missing value in the data must be removed or estimated.
3. The data must be standardized (i.e. scaled) to make the variables comparable. Note that standardization consists of transforming the variables such that they have man zero and standard deviation of one.

In this project, I will be using a built-in R data set `USArrests`, which contains statistics in arrests per 100,000 residents for assault, murder, and rape in each of the 50 US states in 1973. The data also includes the percent of hte population living in urban areas. 


```python
df <- USArrests 
df <- na.omit(df) # remove any missing values that might be present in the data.
df <- scale(df) # I don't want the clustering algorithm to depend on an arbitrary variable unit, therefore I started by scaling/standardizing the data using the R function scale.
head(df)
```

**Clustering Distance Measures**

The classification of observations into groups requires some methods for computing the distance or the (dis)similarity between each pair of observations. The result of this computation is known as a **dissimilarity** or **distance matrix**. There are many methods to calculate this distance information; the choice of distance measuring is a critical step in clustering. It defines how the similarity of two elements ( x , y ) is calculated and it will influence the shape of the clusters.

The classical methods for distance measurements are *Euclidean* and *Manhattan distance*. The one I will be using is *Euclidean*, which is defined as followed:

![1](https://user-images.githubusercontent.com/89553126/145494761-0ccf7f64-94a3-414c-8d1a-dcbd22af8dea.png) [^1]
 
For common clustering software, the default distance measurement is the *Euclidean distance*. However, I should note that depending on the type of data and the research questions, other dissimilarity measures might be preferred, such as *Manhattan*, *Pearson*, *Spearman*, and *Kendall correlation distance*.

Within R we can compute and visualize the distance matrix using the functions `get_dist` and `fviz_dist` and the `factoextra` R package. This begins to illustrate which states have large dissimilarities (red) versus those that appear to be fairly similar (teal).

```python
distance <- get_dist(df) # for computing a distance matrix between the rows of a data matrix.
fviz_dist(distance, gradient = list(low = "#00AFBB", mid = "white", high = "#FC4E07")) # for visualizing the distance matrix
```
![Plot1](https://user-images.githubusercontent.com/89553126/143398927-e2cc283d-bb3f-45ca-bed2-72d8b8a516d0.png)

**K-Mean Clustering**

K-mean clustering is the most commonly used unsupervised machine learning algorithm for partitioning a given data set into a set of *k* groups (i.e. *k* clusters), where *k* represents the number of groups pre-specified by the analyst. It classifies objects in multiple groups (i.e. clusters), such that objects within the same cluster are as similar as possible (i.e. high instra-class similarity), whereas objects from different clusters are as dissimilar as possible (i.e. low inter-class similarity). In k-means clustering, each cluster is represented by its center (i.e. centroid) which corresponds to the mean of points assigned to the cluster.

The basic idea to k-means clustering consists of defining clusters so that the total intra-cluster variation (known as total within-cluster variation)[^2] is minimized[^3].

The k-means algorithm can be summarized as followed:
1. Specify the number of clusters (k) to be created (by the analyst).
2. Select randomly k objects from the data set as the initial cluster centers or means.
3. Assigns each observation to their closest centroid, based on the Euclidean distance between the object and the centroid.
4. For each of hte k clusters, update the cluster centroid by calculating the new mean values of all the data points in the cluster. 
5. Iteratively minimize the total within sum of square. That is, iterate steps 3 and 4 until the cluster assignments stop changing or the maximum number of iterations is reached.

We can compute k-means in R iwht the `kmeans` function. In the code we will group the data into two clusters (`center = 2`). The `kmeans` function also has an `nstart` option that attempts multiple initial configurations and reports on the best one. In my case, adding `nstart = 25` will generate 25 initial configurations. 

```python
k2 <- kmeans(df, centers = 2, nstart = 25)
```
I could view the results by using `fviz_cluster`. This provies an illustration of the clusters. If there are more than two dimensions (variables) `fviz_cluster` will perform principal component analyis (PCA) and plot the data points according to the first two principal components that explain the majority of the variance.

```python
fviz_cluster(k2, data = df)
```

![Plot2](https://user-images.githubusercontent.com/89553126/143398929-d3cbc6d4-952a-49b6-a1b1-6747cf48a4f5.png)

Alternatively, standard pairwise scatter plots can be used to illustrate the clusters compared to the original variables.  

```python
df %>%
  as_tibble() %>%
  mutate(cluster = k2$cluster,
         state = row.names(USArrests)) %>%
  ggplot(aes(UrbanPop, Murder, color = factor(cluster), label = state)) +
  geom_text()
```

![Plot3](https://user-images.githubusercontent.com/89553126/143398931-10e82959-29f8-425d-99ae-49bc503ba805.png)

We can also use several different values of k and examine the differences in the results. We can execute hte same process for 3, 4 and 5 clusters.

```python
k3 <- kmeans(df, centers = 3, nstart = 25)
k4 <- kmeans(df, centers = 4, nstart = 25)
k5 <- kmeans(df, centers = 5, nstart = 25)

# plots to compare
p1 <- fviz_cluster(k2, geom = "point", data = df) + ggtitle("k = 2")
p2 <- fviz_cluster(k3, geom = "point",  data = df) + ggtitle("k = 3")
p3 <- fviz_cluster(k4, geom = "point",  data = df) + ggtitle("k = 4")
p4 <- fviz_cluster(k5, geom = "point",  data = df) + ggtitle("k = 5")


library(gridExtra)
grid.arrange(p1, p2, p3, p4, nrow = 2)
```

![Plot4](https://user-images.githubusercontent.com/89553126/143398932-e4448312-4d4d-4f08-8b90-3f310eaf276e.png)

Although the visual assessment tells us where the true dilineations occur (or do not occur) between clusters, it does not inform us what the optimal number of clusters is.

**Determining Optimal Clusters**

Ideally I would want to use the most optimal number of clusters. In order to figure this out, I enacted the three most popular methods for determining the optimal clusters, which are the **Elbow**, and **Silhouette** method and **Gap statistic**. 

**Elbow method**

The basic idea of this method is that we use the folowing equation:

![2](https://user-images.githubusercontent.com/89553126/145503650-4aa075bf-37d8-49e5-8de9-c0f8910343db.png) [^4]

The total within-cluster sum of square (wss) measures the compactness of the clustering and we want tit ot be as small as possible. Therefore, we can use the following algorithm to define the optimal clusters:
1. Compute clustering algorithm (e.g. k-means clustering) for different values of *k*. 
2. For each *k*, calculate the total within-cluster sum of square (wss)
3. Plot the curve of wss according to the number of clusters *k*.
4. The location of a bend (knee) in the plot is generally considered as an indicator of the appropriate number of clusters.

I can now implement this in R with the following code.

```python
set.seed(123)

# function to compute total within-cluster sum of square 
wss <- function(k) {
  kmeans(df, k, nstart = 10 )$tot.withinss
}

# Compute and plot wss for k = 1 to k = 15
k.values <- 1:15

# extract wss for 2-15 clusters
wss_values <- map_dbl(k.values, wss)

plot(k.values, wss_values,
     type="b", pch = 19, frame = FALSE, 
     xlab="Number of clusters K",
     ylab="Total within-clusters sum of squares")
```

![Plot5](https://user-images.githubusercontent.com/89553126/143398933-c641214b-e0a2-4c0e-a1b0-6e5525839140.png)

This process can be accomplished with a single function `fviz_nbclust`:

```python
set.seed(123)

fviz_nbclust(df, kmeans, method = "wss")
```

![Plot6](https://user-images.githubusercontent.com/89553126/143398935-38a8ab4d-becd-4c3d-8a9d-11ad936874c6.png) [^5]

**Average Silhouette Method**

![Plot7](https://user-images.githubusercontent.com/89553126/143398937-6b4057df-1751-42e6-b66e-9fc519c73285.png)
![Plot8](https://user-images.githubusercontent.com/89553126/143398946-80d360ba-60ae-4ab6-ac46-2cfcbf6c0721.png)
![Plot9](https://user-images.githubusercontent.com/89553126/143398948-e71ceb58-3dff-4838-9446-f8b07ebae6e5.png)
![Plot10](https://user-images.githubusercontent.com/89553126/143398951-fdf8eff1-fe31-4258-b79e-38dc2e2ce32b.png)

[^1]:  Where the x and y are two vectors of length *n*.
[^2]:  The *total within-cluster variation* measures the compactness (i.e. goodness) of the clustering and we want it to be as small as possible.
[^3]:  There are several k-means algorithms available. The standard algorithm is the Hartigan-Wong algorithm, which defines the total within-cluster variation as the sum of squared distances Euclidean distances between items and the corresponding centroid.
[^4]: C<sub>k</sub> is the k<sup>th</sup> cluster and W (C<sub>k</sub> is the within-cluster variation. 
[^5]: Currenlty I am not sure why the function and the formula differ. They should be the same and I am currenlty looking into it. It should be noted that I am writing the code on Mac and have had problems in the past regarding the use of certain function and packages with the operating system. 
