# Burrows-Wheeler-Transform
### Project Description
The aim of this project was to implement a bwt encoder and a bwt backward search program. The provided program can encode, search and decode any text file with ASCII characters in the range of 32 - 126, tab (ASCII 9) and newline (ASCII 10). This implementation is suitable for files up to at least 50MB in size.

----
### Demo
To demo the project simply run the following commands:

To compile the program use:

`$ make` 

The project contains two different programs (bwtencode and bwtsearch):
#### Encode
To encode a file use the follwoing command:

`$ ./bwtencode [delimiter] [index_src_path] [input_src_path] [bwt_src_path]`

#### Search
To search a file use the follwoing command:

`$ ./bwtsearch [delimiter] [bwt_src_path] [index_src_path] [option] [query/search_term]`

Note all file arguments enclosed in square brackets above will need to be replaced with a valid file path. The argument delimiter can be replaced with any single 7-bit ASCII character (eg '$' or '\n'). The argument query can be replaced with any 7-bit ASCII search term (eg 'demo'). The argument option can be replaced with one of four options:
* -m for the number of matching substrings (count duplicates).
* -n for the number of unique matching records.
* -a for listing the identifiers of all the matching records (no duplicates in ascending order).
* -i for displaying the content of the records with ids provided in the search term.

----
### Output
Provided below is an example interaction:

Run the encode program.
```
$ ./bwtencode '$' /tmp/index test.txt bwt.txt
```
After running the encode program the Burrows Wheeler Transform will be stored in the file 'bwt.txt'. The contents of the resulting bwt and original file 'test.txt' are shown below.

test.txt: 'first$second$third$forth$'

bwt.txt: 'tddhenrs$$tthfocfiio$rsr$'

The contents of the auxiliary file 'bwt.txt.aux' and index file is explained in detail below.

Run the search program.

Use -m option (count the number of occurrences of 'th')
```
$ ./bwtsearch '$' bwt.txt /tmp/index -m "th"
2
```

Use -n option (count the number of records containing 'th')
```
$ ./bwtsearch '$' bwt.txt /tmp/index -n "th"
2
```

Use -a option (print the record numbers that contain occurrence of 'th')
```
$ ./bwtsearch '$' bwt.txt /tmp/index -a "th"
3
4
```

Use -i option (decode and print records in the range of record 2 - record 4)
```
$ ./bwtsearch '$' bwt.txt /tmp/index -i "2 4"
second
third
forth
```

Note that the search term will include overlaps (ie there are 2 occurrences of 'abcabc' in 'abcabcabc') and will differ to the output using grep.

----
### Encode
The encoding section of the project is contained within the file bwtencode.cpp and is designed to output the bwt of a file up to 50MB in size. The encoder also creates an auxiliary file which contains positional information for searching and decoding the bwt.

#### Creating the bwt
The suffix array which is used to generate the bwt is found through the combined use of radix sort and the library function qsort. The radix sort operates through recursive bucketing of the suffix array one character at a time. Once a bucket at a certain depth d contains less than a certain threshold number of elements, sorting switches to qsort which completes the sorting from depth d. After creating the suffix array, the bwt is formed by taking the position value (p) from the suffix array at index (i) and writing the character from the original file at position (p - 1) to the bwt at index (i).

In order to facilitate the transfer of positional information in the bwt, the delimiter has been ordered in terms of its positional value. As such, the delimiter at position (i) will be considered before the delimiter at position (j) in the ordering of the suffix array provided i < j. This has the characteristic that once the bwt is formed, we can find the last character of record (r) by looking at the character in the bwt at position (r). The example below illustrates this idea.

Original text: `first$second$third$forth$`

bwt: `tddhenrs$$tthfocfiio$rsr$`

We can now retrieve the last character 't' from the first record 'first' by selecting the first element of the bwt. The same principle can be followed to retieve the final character of the other n records.

#### Creating the auxiliary file
The auxiliary file is used to store positional information which is later used for searching and decoding the bwt. The format of the auxiliary file is as follows. The first n * 4 bytes of the auxiliary file (called the delimiter map) will contain a mapping of the occurrence of the bwt delimiter to its record number from the original file. As such, the integer value at byte location i * 4 in the auxiliary file will contain the record number of the ith found delimiter in the bwt file. In the example provided above we would have the following auxiliary file.

bwt.aux:`[1][4][2][3]` (where each value [v] in the bwt.aux file is an integer)

Original text: `first$_1second$_2third$_3forth$_4` (with labels from bwt.aux file)

bwt: `tddhenrs$_1$_4tthfocfiio$_2rsr$_3` (with labels from bwt.aux file)

We can now map each delimiter in the bwt file by using this postional information. In this case, we can see that the 3rd delimiter found in the bwt at position 21 belongs at the end of the second record in the original file, since 2 is the 3rd integer value in the auxiliary file.

The auxiliary file has been extended in the case where very long records require more frequent mapping from an element in the bwt to its record number in the original file. The extra_map array found in the bwtencode.cpp file is responsible for these extra labelled characters and is referred to as the extra map. The extra map will contain a mapping for every MAP_FREQ character found in the bwt and will be stored inside the auxiliary file after the delimiter map. This extra map is only used if there is sufficient space in the auxiliary file. Below is a sample auxiliary file for the example above based on MAP_FREQ = 4 (ie every forth character found in the bwt starting from the first is mapped in extra_map).

bwt.aux: `[1][4][2][3] [1][2][-1][3][4][-1]` (where each value [v] in the bwt.aux file is an integer)

Original text: `first_1$_1se_2cond$_2th_3ird$_3f_4orth$_4` (with labels from bwt.aux file)

bwt: `t_1ddhe_2nrs$_1$_4tth_3focf_4iio$_2rsr$_3` (with labels from bwt.aux file)

The first four integer values contain the delimiter map as presented before, however the extra seven integer values contains the extra map. By reading the extra map from file we can see that the the character at bwt index 0 lies in record 1. Similarly the character at index 4 lies in record 2. The [-1] stored at the third integer value in the extra map simply means that a delimiter lies at bwt index 8 and the extra map won't be required as the delimiter is already labelled. The extra map provides the advantage of reducing backward search time since only an average of MAP_FREQ characters need to be decoded in order to find a bwt position which has been mapped to a record label in the auxiliary file.

### Search and Decode
The search and decoding section of this project is contained within the file bwtsearch.cpp and is designed to perform four different search functions. The first of these search functions returns the total number of matches of a given query and is specified with the argument '-m'. The second search function specified by '-a' returns all records that contain a match. The search type '-n' returns the number of records that contain a match. Finally the search type -i simply decodes the records specified within the boundaries of the query 'start finish' where the records at position start and end are included. In order to speed up the search and decoding speed, an index file is used which stores a set of occurrence tables containing information on the frequency count of each character at regular block size intervals.

#### Search type '-m'
To return the number of matches for a given query, the backward search algorithm has been used. After completion of the backward search algorithm, the total number of matches can be calculated by taking the value of Last - First + 1. The backward search algorithm pseudocode used in this implementation is shown below for reference (please note that there are minor modifications within the implementation version).

```
backward_search(P[1,p])
|  i = p, c = P[p], First = C[c] + 1, Last = C[c + 1];
|  while ((First <= Last) and (i >= 2)) do
|  |  c = P[i - 1];
|  |  First = C[c] + Occ(c, First - 1) + 1;
|  |  Last = C[c] + Occ(c, Last);
|  |  i = i - 1
|  return (First, Last)
```

The Occ(c, i) function here returns the number of occurrences of character c up to index i within the bwt.

#### Search type '-a'
In order to find the records in which the matches occur, the auxiliary file is needed to map a given bwt character back to the record in which it belongs. As mentioned in the previous section, an auxiliary file is created during the encoding of the bwt which consists of a delimiter map and an extra map (for long record files only). After the program uses the backward search algorithm to find all positions which contain a match, these match locations are passed to a function called find_labelled_postion(). This function uses a slight variation of the backward search algorithm which continues until a labelled postion is reached (this is generally a delimiter or a position that is divisible by MAP_FREQ and present in extra map). This labelled position is then passed to the function record_label() which returns the record number it belongs to. A boolean array tracks the presence of a match for each of the records and is used for printing the sorted order of record matches in the final result.

#### Search type '-n'
This search type follows the same steps as in search type '-a' but instead of printing the records that contain a match, it prints the number of records that contain a match.

#### Search type '-i'
To decode all records within the boundaries 'start end', the positional information mentioned in the encoding section is utilised. Since the bwt has been modified such that the delimiter has been ordered in terms of its positional value, we can easily find the final character of each record at the start of the bwt. For example we can find the final character of record (r) by looking at the character at index (r - 1) within the bwt. Once we have the final character of a record, we can simply perform backward search until the next delimiter is found and print out the full record.

#### Index file
As mentioned previously, the search program does create an index file ('index/occ.txt') to keep track of occurrence tables at regular block size intervals. These occurrence tables contain the frequency count of each character occurring up to block (B). The get_block() function is responsible for retrieveing the occurrence table as well as the next block from the bwt. For example if we call the function get_block() at position i = 5000 with block size set to 4096, the function will retrieve the occurrence table from the index file containing all character frequency counts up to position 4096. It will also retrieve the next block from the bwt containing characters between position 4096 and 8192. The Occ() function can now quickly calculate the number of occurrences of character c at position 5000 by taking occ_table[c] and adding the frequency count of character c between postions 4096 and 5000.

