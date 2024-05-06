# Program Explanation 
The program is quite simple, as the functions used were created with the mindset that it will be modified to incorporate any other data structure/algorithm. Each function has a description above it detailing what they do.
## The Implementation
```c++
/**
 * @file project.cpp
 * @brief The complete file that demonstrates using Treap, Kruskal's algorithm, and Comb sort.
*/

#include <iostream>   // For input and output operations.
#include <fstream>    // For file input and output operations.
#include <vector>     // Standard template library vector for dynamic array functionality.
#include <algorithm>  // For algorithms like sort.
#include <cstdlib>    // Standard General Utilities Library.
#include <ctime>      // For time functions.
#include <set>        // For set data structure which stores unique elements.

using namespace std;

/**
* @brief Struct to represent an edge in a graph.
*/
struct Edge {
    int src, dest, weight;    // Source vertex, destination vertex, and weight of the edge.
    string packageCode;       // Custom identifier for the edge.
};

/**
* @brief Struct to represent a graph.
*/
struct Graph {
    int V, E;                 // V: Number of vertices, E: Number of edges.
    vector<Edge> edges;       // Vector to store edges.

        /**
         * @brief Function to add an edge to the graph.
         * @param src Source vertex of the edge.
         * @param dest Destination vertex of the edge.
         * @param weight Weight of the edge.
         * @param packageCode Identifier for the edge.
        */
        void addEdge(int src, int dest, int weight, string packageCode) {
        edges.push_back({src, dest, weight, packageCode});  // Append new edge to the vector.
    }
};

/**
* @brief Struct for managing subsets for union-find.
*/
struct subset {
    int parent;   // Parent index of the element.
    int rank;     // Rank of the tree rooted at the element.

    // Constructor initializes parent to itself and rank to 0.
    subset() : parent(0), rank(0) {}
};

/**
* @brief Find function with path compression.
* @param subsets Array of subsets.
* @param i Index of the element.
* @return Root of the set to which 'i' belongs.
*/
int find(subset subsets[], int i) {
    // Path compression heuristic.
    if (subsets[i].parent != i)
        subsets[i].parent = find(subsets, subsets[i].parent); // Recursively find the root and make it the parent.
    return subsets[i].parent;  // Return the root of the set to which 'i' belongs.
}


/**
* @brief Union by rank function.
* @param subsets Array of subsets.
* @param x Index of element x.
* @param y Index of element y.
*/
void Union(subset subsets[], int x, int y) {
    int xroot = find(subsets, x); // Find root of element x.
    int yroot = find(subsets, y); // Find root of element y.

    if (subsets[xroot].rank < subsets[yroot].rank)  // If tree with root x has lesser rank, make yroot its parent.
        subsets[xroot].parent = yroot;
    else if (subsets[xroot].rank > subsets[yroot].rank) // If y has lesser rank, make xroot its parent.
        subsets[yroot].parent = xroot;
    else { // If ranks are the same.
        subsets[yroot].parent = xroot; // Make one root the parent of the other.
        subsets[xroot].rank++; // Increment the rank of the new root.
    }
}

/**
* @brief Comb sort algorithm to sort edges by weight.
* @param edges Vector of edges to be sorted.
*/
void combSort(vector<Edge>& edges) {
    int size = edges.size();   // Number of edges.
    int gap = size;            // Initialize gap size.
    bool swapped;              // Flag to check if a swap happened.

    do {
        gap = (gap * 10) / 13; // Decrease gap size using shrink factor.
        if (gap < 1) gap = 1; // Minimum gap is 1.

        swapped = false;
        for (int i = 0; i < size - gap; i++) { // Traverse the array with the current gap.
            if (edges[i].weight > edges[i + gap].weight) { // If elements are out of order.
                swap(edges[i], edges[i + gap]); // Swap them.
                swapped = true; // Set swapped to true.
            }
        }
    } while (gap != 1 || swapped); // Continue until no swaps are needed and gap is 1.
}

/**
* @brief Treap node structure for maintaining a balanced binary search tree.
*/
struct TreapNode {
    Edge key;              // Key on which the treap is sorted.
    int priority;          // Random priority to ensure balance.
    TreapNode *left, *right; // Pointers to left and right children.

    /**
     * @brief Constructor initializes node with an edge and random priority.
     * @param key Edge key.
    */    
    TreapNode(Edge key) : key(key), priority(rand()), left(nullptr), right(nullptr) {}
};

/**
* @brief Right rotation to maintain balanced structure.
* @param y Node to be rotated.
* @return New root after rotation.
*/
TreapNode* rightRotate(TreapNode *y) {
    TreapNode *x = y->left, *T2 = x->right;
    x->right = y;          // x becomes new root.
    y->left = T2;          // T2 becomes left child of y.
    return x;              // Return new root.
}

/**
* @brief Left rotation to maintain balanced structure.
* @param x Node to be rotated.
* @return New root after rotation.
*/
TreapNode* leftRotate(TreapNode *x) {
    TreapNode *y = x->right, *T2 = y->left;
    y->left = x;           // y becomes new root.
    x->right = T2;         // T2 becomes right child of x.
    return y;              // Return new root.
}

/**
* @brief Function to insert a node in the Treap.
* @param node Root of the treap.
* @param key Edge key to be inserted.
* @return Root of the updated treap.
*/
TreapNode* insert(TreapNode* node, Edge key) {
    if (!node) return new TreapNode(key);  // If node is null, create a new node.

    if (key.packageCode < node->key.packageCode) {  // If key is less, insert in left subtree.
        node->left = insert(node->left, key);       // Recursive insert in the left subtree.
        if (node->left->priority > node->priority)  // Perform rotation if necessary.
            node = rightRotate(node);
    } else {  // If key is greater or equal, insert in right subtree.
        node->right = insert(node->right, key);     // Recursive insert in the right subtree.
        if (node->right->priority > node->priority) // Perform rotation if necessary.
            node = leftRotate(node);
    }
    return node;  // Return the potentially new root of the subtree.
}

/**
* @brief Search and display package information based on package code.
* @param root Root of the treap.
* @param packageCode Package code to search for.
* @param allPackageCodes Set of all package codes.
*/
void searchAndDisplay(TreapNode* root, const string& packageCode, const set<string>& allPackageCodes) {
    TreapNode* found = root;
    // Navigate the treap to find the package code.
    while (found && found->key.packageCode != packageCode) {
        if (packageCode < found->key.packageCode) // Move left if the search key is less.
            found = found->left;
        else // Move right otherwise.
            found = found->right;
    }

    if (found) { // If the package is found in the MST.
        cout << "Package " << found->key.packageCode << " found: connects " << found->key.src
             << " to " << found->key.dest << " with weight " << found->key.weight << " (located in the MST)" << endl;
    } else if (allPackageCodes.find(packageCode) != allPackageCodes.end()) { // If package code exists but not in the MST.
        cout << "Package code " << packageCode << " found but not located in the MST." << endl;
    } else { // If package code does not exist.
        cout << "Package code not found." << endl;
    }
}

/**
* @brief Function to construct MST using Kruskal's algorithm.
* @param graph Graph.
* @param treapRoot Root of the treap.
* @param allPackageCodes Set of all package codes.
*/
void KruskalMST(Graph& graph, TreapNode*& treapRoot, set<string>& allPackageCodes) {
    
    vector<Edge> result;    // To store the resultant MST.
    int V = graph.V;        // Number of vertices in the graph.
    combSort(graph.edges);  // Sort edges by weight.

    subset *subsets = new subset[V]; // Allocate memory for creating V subsets.

    for (int v = 0; v < V; ++v) {
        subsets[v].parent = v;  // Initial configuration: each vertex is its own parent.
        subsets[v].rank = 0;    // Initial rank of all vertices is 0.
    }

    int e = 0; // Initial count of edges in MST is 0.
    int i = 0; // Initial index of sorted edges.
    int totalWeight = 0; // Variable to store total weight of the MST.
    set<string> usedPackageCodes; // To track used package codes.

    // Number of edges to be taken is equal to V-1
    while (e < V - 1 && i < graph.E) {
        Edge next_edge = graph.edges[i++]; // Pick the smallest edge.

        int x = find(subsets, next_edge.src); // Find subsets of the vertices of the edge.
        int y = find(subsets, next_edge.dest);

        if (x != y) { // If including this edge does not cause a cycle.
            result.push_back(next_edge); // Add the edge to result.
            Union(subsets, x, y); // Union the two subsets.
            treapRoot = insert(treapRoot, next_edge); // Insert the edge in the Treap.
            totalWeight += next_edge.weight; // Update total weight of the MST.
            usedPackageCodes.insert(next_edge.packageCode); // Record the package code used.
            e++; // Increment count of edges in MST.
        }
    }

    delete[] subsets; // Deallocate memory.

    cout << "The edges in the constructed MST are:" << endl;
    for (const Edge& edge : result) {
        cout << edge.src << " - " << edge.dest << " (Weight: " << edge.weight << ", Package: " << edge.packageCode << ")" << endl;
    }
    cout << "Total weight of MST: " << totalWeight << endl;
}

/**
* @brief Main function to drive the program.
*/
int main() {
    srand(time(nullptr)); // Seed for random number generation.

    ifstream file("graph_data.txt"); // Open the file containing the graph data.
    if (!file) { // Check if file opening failed.
        cerr << "Error opening file." << endl;
        return 1; // Return with error code.
    }

    int V, E; // Variables to store number of vertices and edges.
    file >> V >> E; // Read number of vertices and edges from file.

    Graph graph; // Create a graph instance.
    graph.V = V; // Set number of vertices.
    graph.E = E; // Set number of edges.

    set<string> allPackageCodes; // Set to store all package codes.

    string packageCode; // Variable to store package code.
    int src, dest, weight; // Variables to store the source, destination, and weight of an edge.
    // Read each edge data from file.
    while (file >> src >> dest >> weight >> packageCode) {
        graph.addEdge(src, dest, weight, packageCode); // Add edge to graph.
        allPackageCodes.insert(packageCode); // Insert package code into set.
    }

    file.close(); // Close the file.

    TreapNode* treapRoot = nullptr; // Initialize root of the Treap to null.
    KruskalMST(graph, treapRoot, allPackageCodes); // Construct the MST and populate the Treap.

    cout << "Enter the package code to search: ";
    cin >> packageCode; // Read the package code to search for.

    searchAndDisplay(treapRoot, packageCode, allPackageCodes); // Perform search and display operation.

    return 0; // Return successful execution.
}
``` 
