# 📌 Nearest Neighbor Search (NNS)

Nearest neighbor search (NSS) is an optimization problem that aims to find the point in a given dataset that is closest to a specific data point by measuring the distance using a metric function. This project consists of two KNNS methods, **Brute-Force** and **NN-Descent**.

## K-Nearest Neighbors Search Methods

The **KNN Brute-Force** method computes the K-nearest neighbors for every data point within the dataset by calculating the distance between itself and every other data point. The assumption is that datasets consist of floating-point numbers.

The **NN-Descent** method is a graph-based algorithm for approximate k-Nearest Neighbor Search (KNN). It constructs a K-nearest neighbors graph (KNNG) to find K-nearest neighbors for each data point in a dataset. The algorithm begins by creating a random graph where each vertex is connected to K random neighbors. It then updates the graph by computing potential neighbors based on their distance and adding them to the graph, ensuring that at the end of the process, each vertex has its K-nearest neighbors.

The `calculatePotentialNewNeighbors` method computes potential neighbors for each vertex by considering both its neighbors and reverse neighbors based on their distances. For each pair of potential neighbors, the algorithm calculates the distance between two vertices using the specified metric. If the calculated distance is greater than the distance to the farthest neighbor, the comparison is skipped since the new potential neighbor is not closer than the current farthest neighbor. If the new neighbor is found to be closer, it is added to the set of potential neighbors, and `updateGraph` is called. The `updateGraph` method removes the farthest neighbor, inserts the new one, and updates the reverse neighbors accordingly. This process repeats for each vertex to ensure that the entire graph remains updated.

## Classes

The `KNNDescent` class stores the number of neighbors to be found (K), the size and dimensions of the dataset, and the distance metric between data points. It also contains an array of pointers to `Vertex` objects, where each `Vertex` represents a data point in the dataset.

The `Vertex` class maintains a pointer to the actual data (datapoint) and three sets: Nearest Neighbors (NN), Reverse Nearest Neighbors (RNN), and potential neighbors (PN) that might replace other neighbors of this vertex in the future. Additionally, it keeps a map that stores distances to other vertices.

The `Neighbor` class contains a pointer to an integer representing the ID of the neighbor vertex and a pointer to a double representing the distance between the current vertex and this neighbor.

## Dataset and Distance Metric

Some datasets are available in this repository under the "datasets" directory. The user's distance metric function must be present in the `main.cpp` file. An implementation of Euclidean distance for an arbitrary number of dimensions is already provided in `main.cpp`. The similarity between the brute-force calculated graph and the NN-Descent graph is computed in the `compare_results` function. This function compares the edges of the two graphs based on the IDs of the vertices and prints the similarity percentage. The vertex ID "i" refers to the pointer `data[i]` or the i-th data point in the file reading order.

In the main function, two calculations take place. First, the brute-force graph is computed, followed by the NN-Descent graph. Finally, their similarity is calculated.

## Optimizations

### Sampling

Due to the high computational cost of local joins with a large K, a sampling strategy is introduced. This involves marking nodes for comparison and constructing sampled lists of neighbors. The user can adjust `p` (the second argument) to balance between precision and time.

### Graph Storing

An additional optimization in the k-nearest neighbors algorithm is implemented for cases where the dataset size (N) is below or equal to 5000. In such cases, the program uses a precomputed graph stored in a file, which is retrieved and used to avoid redundant calculations.

### Distance Calculation

The Euclidean distance calculation has been optimized by focusing on squared distances rather than absolute distances. The absolute value of the distance was found to be less significant in this use case, emphasizing its comparison with other distances. This optimization is achieved by precomputing and storing vector norms using the binomial theorem, reducing the computational workload during distance calculations.

### Squares Precomputing

Squares of vector norms are precomputed and stored for later use in distance calculations. This optimization saves considerable computation time, as squares are calculated once at the beginning, reducing the workload in subsequent distance calculations. Additionally, the high-performance `cblas_sdot` function from the CBLAS library is utilized to ensure efficient dot product calculations.

### Parallelization

Four methods have been parallelized to improve performance: `calculatePotentialNewNeighbors`, `updateGraph`, `calculateSquares`, and `updateRPGraphOpt`.

The `calculatePotentialNewNeighborsOpt` method takes advantage of hardware concurrency by dynamically creating a number of threads (`num_threads`) based on the system's hardware capabilities. Each thread is assigned a specific portion of the workload by making calls to the function `parallelCalculatePotentialNewNeighbors`. This method is a parallelized version of `calculatePotentialNewNeighbors`. A mutex is assigned to each data point, blocking access to shared data when necessary and preventing race conditions.

The `updateGraph2` method follows a similar threading pattern but involves a more complex locking mechanism due to the amount of shared resources.

The `calculateSquares` method efficiently computes the squares of data points in a multi-dimensional space using parallelization for better performance. Multiple threads work simultaneously on different subsets of data points. The method dynamically decides the number of threads based on hardware capabilities (`num_threads`). Each thread handles a specific range of data points, while the `parallelSquares` function is executed concurrently to calculate the squares. Within `parallelSquares`, a mutex (`squareMutex`) ensures that only one thread accesses the shared squares array at a time, preventing conflicts between threads and ensuring safe computation.

The parallelization of `updateRPGraphOpt` is relatively simple. It does not involve mutexes, as shared resources are not at risk of concurrent access. Instead, it divides the workload among multiple threads to reduce execution time.

## Disclaimer

The `ADTSet` is based on the open-source code of the k08 class, which can be found at [GitHub Repository](https://github.com/chatziko-k08/lecture-code).  
The `cblas_sdot` function is included in the CBLAS library.


- **HOW TO RUN:**

make run ARG='100 0.5 datasets/00001000-1.bin 0.01' : Runs the main function for K = 100, SamplingFactor = 0.5, DatasetPath = datasets/00001000-1.bin, Delta = 0.01
make test : Runs the tests

