# Fuzz testing 

https://github.com/antonio-morales/Fuzzing101?tab=readme-ov-file

Fuzz testing (or fuzzing) is an automated software testing technique that is based on feeding the program with random/mutated input values and monitoring it for exceptions/crashes.

AFL, libFuzzer and HonggFuzz are three of the most successful fuzzers when it comes to real world applications. All three are examples of Coverage-guided evolutionary fuzzers.

## Coverage-guided evolutionary fuzzer

* **Evolutionary**: is a metaheuristic approach inspired by evolutionary algorithms, which basically consists in the evolution and mutation of the initial subset (seeds) over time, by using a selection criteria (ex. coverage).

* **Coverage-guided**: To increase the chance of finding new crashes, coverage-guided fuzzers gather and compare code coverage data between different inputs (usually through instrumentation) and pick those inputs which lead to new execution paths.


## Install AFL

1. Install dependencies

```
sudo apt-get update
sudo apt-get install -y build-essential python3-dev automake git flex bison libglib2.0-dev libpixman-1-dev python3-setuptools
sudo apt-get install -y lld-11 llvm-11 llvm-11-dev clang-11 || sudo apt-get install -y lld llvm llvm-dev clang 
sudo apt-get install -y gcc-$(gcc --version|head -n1|sed 's/.* //'|sed 's/\..*//')-plugin-dev libstdc++-$(gcc --version|head -n1|sed 's/.* //'|sed 's/\..*//')-dev
```

2. Checkout and build afl

```
cd $HOME
git clone https://github.com/AFLplusplus/AFLplusplus && cd AFLplusplus
export LLVM_CONFIG="llvm-config-11"
make distrib
sudo make install
```

3. run afl 

```
afl-fuzz
```

## Create new binaries with afl-compiler

AFL is a coverage-guided fuzzer, which means that it gathers coverage information for each mutated input in order to discover new execution paths and potential bugs. When source code is available AFL can use instrumentation, inserting function calls at the beginning of each basic block (functions, loops, etc.)

To enable instrumentation for our target application, we need to compile the code with AFL's compilers.

1. Download sources from repo you will test

2. In the test-repo run make with the afl-compiler

Example here with xpdf.

```
export LLVM_CONFIG="llvm-config-11"
CC=$HOME/AFLplusplus/afl-clang-fast CXX=$HOME/AFLplusplus/afl-clang-fast++ ./configure --prefix="$HOME/fuzzing_xpdf/install/"
make
make install
```

3. run the fuzzer

```
afl-fuzz -i $HOME/fuzzing_xpdf/pdf_examples/ -o $HOME/fuzzing_xpdf/out/ -s 123 -- $HOME/fuzzing_xpdf/install/bin/pdftotext @@ $HOME/fuzzing_xpdf/output
```

A brief explanation of each option:

* -i indicates the directory where we have to put the input cases (a.k.a file examples)

* -o indicates the directory where AFL++ will store the mutated files

* -s indicates the static random seed to use

* @@ is the placeholder target's command line that AFL will substitute with each input file name

If you receive a message like "Hmm, your system is configured to send core dump notifications to an external utility...", just do:

```
sudo su
echo core >/proc/sys/kernel/core_pattern
exit
```

You can see the "uniq. crashes" value in red, showing the number of unique crashes found. You can find these crashes files in the $HOME/fuzzing_xpdf/out/ directory. You can stop the fuzzer once the first crash is found, this is the one we'll work on. It can take up to one or two hours depending on your machine performance, before you get a crash.


## I found a core file

1. Locate the file corresponding to the crash in the $HOME/fuzzing_xpdf/out/ directory. The filename is something like id:000000,sig:11,src:001504+000002,time:924544,op:splice,rep:16

2. Use gdb

First recompile test-binarie without afl-compilter 

```
rm -r $HOME/fuzzing_xpdf/install
cd $HOME/fuzzing_xpdf/xpdf-3.02/
make clean
CFLAGS="-g -O0" CXXFLAGS="-g -O0" ./configure --prefix="$HOME/fuzzing_xpdf/install/"
make
make install
```

run gdb 

`gdb --args $HOME/fuzzing_xpdf/install/bin/pdftotext $HOME/fuzzing_xpdf/out/default/crashes/<your_filename> $HOME/fuzzing_xpdf/output`

and typ inside GDB:

```
>> run

>> bt  (backtrace)
```

