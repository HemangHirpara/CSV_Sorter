# CSV_Sorter
Multithreaded CSV Sorter

# Objective

The objective of this program is to be able to sort multiple csv files in a given directory and subdirectories using threads.

inputs: A start directory (optional), an output directory (optional),

the name of a column to sort by output: create final new file and put in in the location found if no output directory specified. Output file contains all data from valid csv files

# Usage

1. The program goes through the starting dir, and creates a thread for every file and directory it finds and sorts the file if it is a valid csv
2. Use the command line to run the program:

```
./multiThreadSorter - c ‘column_name’ -d ‘start_dir’ -o ‘output_dir’
```
# Design
```
- c parameter is used to denote sorting by column

- d parameter to denote starting directory

- o parameter to denote the output directory
```
File created:

1. multiThreadSorter_thread.c
2. multiThreadSorter_thread.h
3. mergesort.c

# Data Structures
Structs:
```
typedef struct rowNode { char * row; // store each record (row)
char * sort; // store the column to sort CSV file from row
char * col[28]; // store the columns from the CSV file row in correct spot
struct rowNode * next ; // pointer to the next struct (row) } rowNode;
```
Linked List rowNode:

Used to simulate the CSV file where each rowNode structure is a row in the CSV file; later fed into merge sort to get the output.

Linked List tNode:

Used to keep track of thread id’s that have been created by
finding directories and files.

# Functions:
```
1. Int checkFlags(int argc, char *argv[])
    a. Uses the argc and argv values along with getopts to check validity of the arguments passed in. Returns a - 1 on error
    b. Checks to see if column to sort by is a numeric column and sets isNum accordingly
    c. Checks if specificed starting directory and output directory are valid
2. Int find_loc(char *col_name)
    a. Returns the correct spot to put the col_value in an array of 28 entries. One for each column heading for final file
    b. Returns -1 on error, invalid col_name
3. Int count_rows(char *line)
    a. Returns the number of comma’s found within the given line or row. Accounts for comma’s found within quotes
4. Char *trim(char *str)
    a. Returns the input string, which is a column values, that has leading and trailing whitespaces removed. Done to ensure proper comparisons in merge sort
5. Char *get_filename_ext(char *filename)
    a. Returns the extension of a given filename. Used to check if a file found is a CSV file, otherwise an invalid file
6. Void *dirHandler(void *dir_path)
    a. Recursively finds directories and files from a given starting path.
    b. If directory is found, a thread is created and used to call dirHandler() again.
    c. If file is found, a thread is created and used to call fileHandler(). Once fileHandler returns the head of the list, this list is sorted using merge sort and then added to the end of the main linked list that is collecting all valid data.
    d. Creates a thread id for each thread created and stores it in a Linked List of tNodes to print out once each directory is finished
7. Void *fileHandler(void *path)
    a. Checks whether the file found is a valid CSV file 
    b. If invalid, prints a descriptive error message and the thread returns.
    c. If valid, the file is tokenized row by row and then column by column and stored in a Linked List of rowNodes.
    d. Makes use of tokenize function on each row, and places the column value in the proper spot in array
    e. The array is inside the rowNode, size of 28, used to print out the final row for each row found from each file
    f. The thread returns the head of the linked list is created with the data from the valid CSV file.
```
In main, a thread is created to initially call dirHandler. Mutexes have been placed around code segments that require the threads to access shared data such as thread count, rowNode linked list and tNode linked list. Metadata is printed out after each thread has finished its execution. After all metadata is printed out, merge sort is called on the linked list of rowNodes to sort the list. Then the list is outputted to a final file named according to the structure of “AllFiles-sorted-<column name>.csv”. This file is placed in the directory specified by the –o flag.

# Header File: simpleCSVsorter.h

The header file used for the program consisted of function definitions for the entirety of the program. It also included the variable int isNum, which was a flag that denoted whether the column we were sorting by was a numeric column or not. This allows us to the proper merge function. We also included the structs for our program, a rowNode, and a tNode, which is later used to make a linked list of rowNodes and tNodes.

# Difficulties & Assumptions

Difficulties: We had difficulties in working out how to use mutexes for the shared data. We had to rewrite how we isolated the data from each line which required rethinking our whole program. Using threads instead of forking process was a little challenging as we had to use mutexes and at certain places return a value
from a closed thread. The majority of the difficulties arose from creating a final file with all data found from valid CSV files. This required us to rewrite out tokenize function which does a bulk of the work besides merge sort. We had difficulties figuring out how to store all the data from each CSV files and then create one final file. We ended up using a linked list to keep track per file.

# Debugging & Testing

Testing this program required creating smaller CSV files that are easier to read and easy to manipulate. For testing and debugging any program, the general approach we use is to figure out ways to essentially break the program. 

First, this means find errors in input:
```
1. Invalid arguments, such as wrong/misspelled column to sort by name.
2. Invalid directories such as a directory that does not exist
3. Missing arguments, such as not including the ‘-c’ , argument
4. CSV files that were a mix between valid and invalid
5. The proper number of threads are outputted and unique thread ids
```
We had a function called checkFlags which checks if the input arguments on the command line are correct and valid to run the program. This is where we also checked if the column to sort by value was valid. If an invalid column name was specified, it was a fatal error. To check CSV files, we created smaller sample.csv and sample2.csv files which had a mix between proper headings and invalid headings and a mix between the string and numeric values. This allowed us to check if we correctly checked for an invalid CSV heading, and we sorted properly based on the type of the column.

We also checked edge cases such as one column CSV file, sort by the last column of a CSV file, sort on movie_title which contains quotes and commas within. We also ran the program on all three test files, movie_metadata.csv, sample.csv, and sample2.csv to make sure the exclusion of a majority of the columns did not affect the output and that the output was in the correct format.

We also used the movie_metadata.csv file to test the program because of its large size and a large number of rows. We used multiple copies of the movie_metadata.csv file and subsets of that file to test the time it takes for the program to run.

# Testing nested directories:

We tested empty directories, directories with 16 subdirectories with the final directory containing 10 empty CSV files and 10 invalid text files. TEST/foo/bar/zoo/moo/lar/poo/A/B/C/D/E/F/G/H/I/J. Directory J contained 20 files. This was tested to make sure the program could search for all files and directories. We also had a test structure. TestStructure contained inner and inner 2 and movie.csv file. inner had directory emptyDir, which was empty and 3 invalid text files. inner2 had three empty CSV files.
```
Sample.csv

color,duration,gross,country,movie_title
Blue,1,434,USA,hemang
Black,3,147,CHINA,mira
red,5,555,INDIA,poojan
white,7,11,FRANCE,diana

sample2.csv

color,duration,gross,country,movie_title
orange,2,542,C,broccoli
,4,086,B,”mush,room”
gold,6,765,E,”tree,head”
purple,8,411,A,potato
```
# Time analysis

Multiprocessing program:
```
1. Long directory chain: average time with 1 file at end of 16 directories is 0.208 seconds
2. A large number of files in a directory: average time with 256 files is 7.897 seconds
```

Multithreading program:
```
1. Long directory chain: average time with 1 file at end of 16 directories is 0.0977 seconds
2. A large number of files in a directory: average time with 256 files is 24.7 seconds
```
1. The comparison between run times is not a fair one because the run times will vary based on how the code was written. Because one program involved processes and writing an output file for each valid CSV while the other involved threads and writing a single output file with all valid CSV data. Writing to a file and using I/O is very slow. Also, it should be noted that both programs might be looking for valid CSV files and sorting them but they are not necessarily doing the same thing. One required us to write to a final file with the other requiring us to writing to multiple output files.
2. The program which involved processes as opposed to threads was slower. This discrepancy was due to several factors, mainly that our program with processes was making use of multiple while loops, we were inserting to the end of a linked list that could’ve been very long, some of our functions were inefficient in general, and writing an output file for each valid file found which is a lot of I/O usage. The program which involved threads as opposed to processes was much faster. This was because we had rewritten some of the code such as tokenize that was inefficient and mainly we were only writing to a file once at the end which limits the I/O usage.
3. To make the program which involves processes faster would require limiting the number of times an output file is created. We noticed that when we wrote to an output file at the end for out thread program, there was a significant delay depending on the size of the accumulated file data, specifically a delay when writing an output file. Also, the processes are slower than threads because the OS can change the context for threads faster than for processes. When writing code that involves processes, we had to wait until a process was done to continue execution while the threads did not have to wait until they reached shared data.
4. In our case, merge sort seemed like the best sorting algorithm because our main structures holding the data was a linked list. Merge sort is the fastest for a linked list with time complexity of O(nlogn). It seemed like the best algorithm because it was less complicated to sort given the way we had stored our data. However, generally speaking, mergesort would not be the right option for a multithreaded program if you were to use threads to do the sorting. Threads can access shared data and open files. So, it would seem faster to use threads to sort a multithreaded program as we are writing to one final output file and accessing data from multiple different files. It would be faster for each thread to start writing to the output file at the same which would increase the speed of the program.
