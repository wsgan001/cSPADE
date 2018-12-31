# Quick Start

Needs utils from the tposedb repo: https://github.com/zakimjz/tposedb

Here are the steps to follow for the example file test.ascii

1) makebin test.ascii test.data
The *.data extension for binary is required

2) getconf -i test -o test

3) exttpose -i test -o test -s 0 -l -x

4) now you can run spade 

    spade -e 1 -r -i test -s 0.2 -u 2 -o

this means max gap between events is 2, and minsup is 20% or 2/10

an example output pattern:

22 80 -> 72 -> 42 -> 22 -- 2 2

gives you the frequent sequence followed by support (the last two
		numbers, which will be the same in this application).

This means that the itemset (22 80) is followed by 72 followed by
42 followed by 22.

Other flags of interest include:

    -w max_window for the whole sequence
    -z max_itemset length
    -Z max_seq length (in terms of the number of itemsets it can have)

# Details

## Your own sequence data in ASCII format

Your data must conform to the following format in ASCII
    
    cid tid numitems itemlist

cid is sequence ID (int)
tid is event ID/time (int)
numitems is number of items in event (int)
itemlist is sorted item IDs (int) for that event

See test.ascii and test2.ascii for examples

If you are using a real DB then it must still be in the same
format. If it is initially in ASCII format, you can convert it to
binary using:

    makebin XXX.asc XXX.data, where XXX.asc is the ascii db input.

Next run 
    
    ./getconf -i XXX -o XXX to obtain XXX.conf file.

Next, run: 
    exttpose -i XXX -o XXX -l -s LMINSUP
        e.g. exttpose -i XXX -o XXX -l -s 0 -x

note: this produces the files XXX.tpose, and XXX.idx

The XXX.tpose file is the DB in vertical format, and
XXX.idx is an index file specifying where the tid-list for each item
begins.

You can specify a value of LMINSUP to be the same as the one you will use to 
run spade below, in which case you will have to rerun exttpose each time you 
use a new lower MINSUP. Alternatively, you can use a small value for LMINSUP, 
and it will continue to work for all values of MINSUP >= LMINSUP when you
run spade.

The time for inverting is stored in summary.out. The format is:

    TPOSE DB_FILENAME X NUMITEMS TOTAL_TIME

(see note one TOTAL_TIME below)

## Running cSPADE
Now you can run cSPADE as follows:

        spade -e 1 -r -i XXX -s MINSUP

MINSUP is in fractions, i.e., specify 0.5 if you want 50% minsup or
0.01 if you want 1% support.

        some other options:
        -o output mined sequences (-2 separates the sequence from the
                                support)
        -l min_gap between itemsets in a seq
        -u max_gap between itemsets in a seq        
           note: you can specify both max and min gap simultaneously.
           for instance if you want to find all sequences with
           itemsets exactly g apart you can use -l <g> -u <g>
        -w max_window for the whole sequence
        -z max_itemset length
        -Z max_seq length (in terms of the number of itemsets it can have)
        -M use for weighted support (multiple counts/seq)

  For example here are a few exmaple runs:
    
    spade -e 1 -r -i XXX -s MINSUP -l 2
    
    spade -e 1 -r -i XXX -s MINSUP -u 2
    
    spade -e 1 -r -i XXX -s MINSUP -l 2 -u 2
    
    spade -e 1 -r -i XXX -s MINSUP -l 2 -u 4
    
    spade -e 1 -r -i XXX -s MINSUP -w 4
    
    spade -e 1 -r -i XXX -s MINSUP -l 2 -u 4 -w 10
    
    spade -e 1 -r -i XXX -s MINSUP -z 3
    
    spade -e 1 -r -i XXX -s MINSUP -Z 3
    
    spade -e 1 -r -i XXX -s MINSUP -z 3 -Z 4
  


note that the summary of the run is stored in the summary.out
file. The format of this file is as follows:

SPADE DB_FILENAME MINSUP ACTUAL_SUPPORT NUMBER_OF_JOINS JOIN_TIME_FOR_F2
      . . . :  F1 F2 F3 .... F_k : F1_TIME F2_TIME FK_TIME TOTAL_TIME

The values of most interest to you are F1, F2, F3, ... F_k, ... which
give the number of frequent k-sequences and TOTAL_TIME which is the
total Wall-Clock or Elapsed time (and NOT just the CPU time).

Note1: You can use the -o option to print out the actual sequences (to
stdout).  

Note2: -r option does a DFS generation of sequences, which consumes less 
memory, and thus can run even when there are long sequences.
You can try to omit the flag to do BFS generation if you want.

Note3: -e 1 option is a flag indicating spade to compute the support
of 2-sequences from scratch. The number 1 says there is only one DB
partition that will be inverted entirely in main memory. If the
original DB is large then this inversion will obviously take too much
time. So in this case I recommend dividing the DB into chunks of size
roughly 5MB (assuming there is 32MB available to the process). The
exttpose program is equiped to handle this case. If you specify a <-p
NUMPART> flag to exttpose it will divide the DB into NUMPART
chunks. Now you can run spade with -e NUMPART option. You must do this
if the DB is large otherwise the timings for spade will be
skewed. Generally, the more the partitions the better the running time
for spade. For example:        
        exttpose -i XXX -o XXX -x -l -s LMINSUP -p 10
        spade -i XXX -s MINSUP -e 10 -r

Note4: You can use the -m MEMSIZE option to increase the memory
available to the program. MEMSIZE is given in MB. For example if you
have 64 MB available, then use -m 64 (the default is 32MB).

## Generating synthetic data
1) Generate a data file using the IBM data generator program,
gen. This is the same as the public version from Almaden, but I have
enhanced it to generate a configuration file, and a modified ascii
database file.

For cSPADE make sure the DB is in binary format.  The output file
should be named XXX.data (note the 'data' extension). The gen program
also produces a XXX.conf file automatically.

The format of the binary file should be 
cid tid numitem itemlist

run gen seq -help for data generation options.


