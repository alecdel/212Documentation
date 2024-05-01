# How to set up the file
### Note: the files can be renamed to whatever you want, make sure to change the file names when compiling and running
## How to run
- Make sure g++ is installed on your machine
- Download the two required files: project.cpp and graph_data.txt
- In your terminal and if you are in the directory that the two files are downloaded into, to compile: g++ project.cpp -o test (test can be anything)
- To run the compiled test program in your terminal: ./test graph_data.txt

### Here is a step by step of how to run the file
```
g++ project.cpp -o test
./ test graph_data.txt
```
### Once ran this will be the output with the current graph_data.txt file
```
The edges in the constructed MST are:
0 - 1 (Weight: 10, Package: PKG100)
1 - 2 (Weight: 15, Package: PKG200)
2 - 3 (Weight: 20, Package: PKG300)
3 - 4 (Weight: 25, Package: PKG400)
Total weight of MST: 70
Enter the package code to search:
```
In this, the weight is the distance from the previous neighborhood
### It then asks for an input, currently inputing either PKG100, PKG200, PKG300, or PKG400 will produce this with the selected package
```
Enter the package code to search: PKG100
Package PKG100 found: connects 0 to 1 with weight 10 (located in the MST)
```
### If no package is located in the MST, this will be the output
```
Enter the package code to search: PKG500
Package code not found.
```
Currently there is no way of inserting packages but that can easily be implemented using the insert function with the input, but in the scope of this project, we purposefully allowed limitations
