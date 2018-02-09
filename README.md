
We are going to fuzz programs from the RERS 2016 or 2017 challenges. The first step is to download and install AFL:

AFL - http://lcamtuf.coredump.cx/afl/

Simply run make and you are good to go.

Then download and unpack the RERS challenge programs:

The RERS 2016 reachability problems - http://www.rers-challenge.org/2016/problems/Reachability/ReachabilityRERS2016.zip
The RERS 2017 reachability training problems - http://rers-challenge.org/2017/problems/training/RERS17TrainingReachability.zip

The archives contain highly obfuscated c and java code, e.g., :

In ReachabilityRERS2016/Problem10/Problem10.c

```C++
...
void errorCheck() {
	    if(((a90 == 9 && a160 == 8) && a56 == 10)){
	    cf = 0;
	    __VERIFIER_error(0);
	    if(((a179 == 10 && a160 == 2) && a56 == 10)){
	    cf = 0;
	    __VERIFIER_error(1);
	    }
}
...
 void calculate_outputm46(int input) {
    if(((a168 == 7 && ((((input == 4) &&  cf==1 ) && a92 == 4) && a63 == 33)) && (a0 == 10 && (a164 == 33 && a41 == 1)))) {
    	cf = 0;
    	a56 = 12;
    	a82 = 6;
    	a14 = 32 ;
    	a95 = 10;
    	a188 = 5;
    	a39 = 2;
    	a168 = 8;
    	a25 = 32 ;
    	a164 = 32 ;
    	a24 = 34 ;
    	a79 = 9;
    	a111 = 10;
    	a109 = 7;
    	a191 = 32 ;
    	a196 = 32 ;
    	a150 = 32 ;
    	a131 = 33 ;
    	a200 = 32 ;
    	a37 = 11;
    	a7 = 5;
    	a41 = 2;
    	a92 = 5;
    	a63 = 32 ;
    	a0 = 11;
    	a197 = 34 ;
    	a110 = 13; 
    	 printf("%d\n", 25); fflush(stdout); 
    } 
...
void calculate_output(int input) {
        cf = 1;

    if(((a7 == 4 && ((a63 == 33 && (a197 == 33 && ( cf==1  && a56 == 7))) && a86 == 33)) && (a0 == 10 && a150 == 33))) {
    	if(((((a200 == 33 && ( cf==1  && a69 == 32)) && a63 == 33) && a196 == 33) && (a95 == 9 && (a79 == 8 && a63 == 33)))) {
    		calculate_outputm2(input);
    	} 
    } 
    if(((a142 == 6 && (a25 == 32 && (a7 == 5 && a79 == 9))) && (a188 == 4 && (( cf==1  && a56 == 8) && a150 == 32)))) {
    	if(((a164 == 32 && (a86 == 32 && a111 == 10)) && (a111 == 10 && ((a95 == 10 && ( cf==1  && a160 == 2)) && a82 == 6)))) {
    		calculate_outputm6(input);
    	} 
...
int main()
{
    // main i/o-loop
    while(1)
    {
        // read input
        int input;
        scanf("%d", &input);        
        // operate eca engine
        if((input != 2) && (input != 5) && (input != 3) && (input != 1) && (input != 4))
          return -2;
        calculate_output(input);
    }
}
```

Good luck understanding the logic underlying this code... All we know is that it is a simple state machine, which has been obfuscated to make analysis harder. In case you are interested in obfuscation: check out Tigress: http://tigress.cs.arizona.edu. We of course use automated tools in order to try to understand it!

The goal of the RERS challenge is to discover exactly which VERIFIER_error(s) are reachable, some of the if statements in errorCheck() can never occur, while other can occur but only after reasing a very long input. The input to the program is simply a long list of integers, seperated by spaces or line ends. After reading an input, the program returns an integer, or gives an error.

The c code will not compile as given, we need to make a few changes, and some more to run AFL on it.

First, replace:

```C++
    extern void __VERIFIER_error(int);
(line 6)
```

with:
   
```C++
void __VERIFIER_error(int i) {
    fprintf(stderr, "error_%d ", i);
    assert(0);
}
```

The assert causes a crash, which is what we want to find using AFL.

Then, replace:

```C++
int main()
{
    // main i/o-loop
    while(1)
    {
        // read input
        int input;
        scanf("%d", &input);        
        // operate eca engine
        if((input != 2) && (input != 5) && (input != 3) && (input != 1) && (input != 4))
          return -2;
        calculate_output(input);
    }
}
```

with:

```C++
int main()
{
	// main i/o-loop
	while (1) {
		// read input
		int input = 0;
		int ret = scanf("%d", &input);
		if (ret != 1) return 0;
    if((input != 1) && (input != 2) && (input != 3) && (input != 4) && (input != 5)) return 0;
		// operate eca engine
		calculate_output(input);
	}
}
```

This avoids a hang in the scanf function created when the input ends.

Now all we need to run afl is to create two directories: tests and findings. In tests you have to provide AFL with some example inputs, makes sure that you give an integer with a space or new line. For instance, I have the file 1.txt in the test directory, containing:

```C++
"1
"
```

so a 1 with a new line.

Make sure that you create files with all possible input symbols (otherwise AFL has to discover these which might take a long time).

Now we are ready to fuzz.

First compile the file you edited (for example Problem10.c) using the afl-gcc compiler or in my case afl-clang:

path_to_afl/afl-clang Problem10.c

Then run afl on the obtained binary:

path_to_afl/afl-fuzz -i path_to_test_dir -o path_to_findings_dir path_to_binary/a.out

You should see AFL starting, perhaps with some warnings due to useless input files, you can simply ignore these.

Keep on fuzzing.

Then after some time, you can investigate the findings directory. You will find for instance the crashes found in the crashed directory, and interesting test cases (which it uses to trigger new paths) in queue. You can run the crashes found in the fuzzed program in order to answer the reachability problems:

cat id:000000,sig:06.src:000001,op:havoc,rep:2 | path_to_binary/a.out

Gives in my case:

...

error_29 ...

Implying that error 29 is reachable.

In this way, and some extra tricks, we obtained third place in the reachability category of the RERS 2016 challenge: http://rers-challenge.org/2016/index.php?page=results. We are team Radboud (together with PhD students from Radboud university), for some reason they still did not update the names. We wrote a paper describing our approach (including learning which you will learn later in this course), which is available at https://arxiv.org/pdf/1611.02429.pdf. Please take a look if you are interested.


