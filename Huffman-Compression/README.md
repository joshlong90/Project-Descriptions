# Huffman-Compression
### Project Description
The aim of this project was to implement the static huffman compression algorithm. The provided program can encode / decode any ASCII or binary file and search an ASCII 7-bit encoded file.

----
### Demo
To demo the project simply run the following commands:

To compile the program use:

`$ make` 

To run the program there are 3 different functions:
#### Encode
To encode a file use the follwoing command:

`$ ./huffman -e [input_src_path] [compressed_src_path]`

#### Decode
To decode a file use the follwoing command:

`$ ./huffman -d [compressed_src_path] [output_src_path]`

#### Search
To search an encoded file use the follwoing command:

`$ ./huffman -s [query] [compressed_src_path]`

Note all arguments enclosed in square bracket above will need to be replaced with a valid file path. The argument query can be replaced with any 7-bit ASCII search term (eg 'demo').

----
### Output
Provided below is an example interaction:

Create a 1MB text file consisting of the characters 'a', 'b', 'c' and 'd'.
```
$ python3 file_generator.py 
Enter the file size in bytes: 1000000
```

Run the encode, search and decode programs.
```
$ ./huffman -e input.txt compressed.txt
$ ./huffman -s 'abcabc' compressed.txt
257
$ ./huffman -d compressed.txt output.txt
```

Check the file sizes and validate that the decoded output file matches the original input file.
```
$ stat -f%z input.txt
1000000
$ stat -f%z compressed.txt
219755
$ stat -f%z output.txt
1000000
$ diff input.txt output.txt
$ 
```
Note that the search term will include overlaps (ie there are 2 occurrences of 'abcabc' in 'abcabcabc') and will differ to the output using grep.

----
### Huffman Tree Design
The huffman tree has been designed using a custom abstract data type named in this project as a priority queue binary tree (pqbtree). The pqbtree is essentially a sorted list that maintains order in terms of increasing frequency however each node in this list also acts as a binary tree node (with left and right child). Initially all characters with their frequency are pushed onto the pqbtree using pqbtreePush(). Once all characters are pushed onto the pqbtree, the merge function pqbtreeMerge() is invoked. This function essentially pops the first two nodes of the pqbtree (which will have the lowest frequencies) and then creates another parent node which is pushed back onto the pqbtree and shuffled into order of priority. The new node that is pushed will contain the removed nodes as its right and left child nodes. This process is continued until there is only one root node at the top of the tree.

After the tree is constructed, the bit_table is created, which provides a mapping of each character to its compressed bit representation. The bit_table is created from the tree by invoking the function pqbtreeFormBitTable(). This function performs a depth first search through the tree to find each leaf recording the path as a series of 0s or 1s at each left or right move downwards. Once the leaf is reached a mapping is created in the bit_table from the character found at the leaf node to the path (bit representation) required to get there.

### Header Design
The 1024 byte header of the compressed file has been broken into 256 divisions. Each of these divisions contains 4 bytes (the size of an integer value) and has been used to store the frequency value for each bit representation. For example, the frequency of the character 'A' (with bit representation 01000001) in the original file is stored as an integer value in the 65th division of the header. As such, during decompression and search, the frequency of each character in the original file has been retrieved from the compressed file header by using getw(compressed_file) which retrieves an integer value from file.

### Search
The search algorithm has incorporated the KMP algorithm. This choice was made because it has the benefit of always moving forward through the file which limits complications of finding matches across separate consecutive buffers. Before sending the file contents to the KMP algorithm, part of the file is decompressed into a buffer. This avoids complications associated with non fixed length character representations. The failure function and KMP algorithm have remained similar to a normal implementation. When a match is found, the match count is incremented and the search continues forward. The value of j which tracks the current position within the query pattern is maintained inbetween buffers. The i value which checks the current position within the buffer is set to 0 as per usual everytime a new buffer is checked for matches.
