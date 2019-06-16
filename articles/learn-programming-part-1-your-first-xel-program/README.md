# Part 1: Your First XEL Program
## Quick Introduction
This tutorial is designed for software programmers with a need to understand the ePL programming language starting from scratch. You will gain enough knowledge from where you can take yourself to a higher level of expertise. Let us give you a brief overview of what ePL is: ePL is a programming language which was designed explicitly for coding algorithms to be executed on the XEL computation node network. Its syntax is very similar to the C programming languages. However, since nodes on the XEL network download and execute code from potentially dangerous sources a few adaptations were necessary to ensure that code written in ePL can cause no harm to the system it is executed on. Furthermore, since task distribution and verification is done in a decentralized fashion (all nodes which meet a minimum system requirement in terms of memory and CPU power must be able to verify the algorithms’ results in a similar amount of time) is it designed to run in a resource-limited VM. Furthermore, ePL has a slightly modified memory management (and instruction set) which ensures that every node can assess how much memory the program will consume and when it will terminate before execution. This avoids unpredictable behavior such as out of memory errors and the halting problem. However, before we get too deep into the little details, you should make sure you have installed all components as described in the "Installation" section.

## A Simple Example: Finding Prime Numbers

The example we will be looking at is a simple ePL program which is meant to find prime numbers between 10000 – 20000. While it has no real relevance for the real world, it certainly is small enough to help in understanding how ePL works. First of all, let us take a look at the entire code before we start breaking down each line of it:

```
array_uint 1000;

function main {
    verify();
} 

function primetest {
    u[1] = 0;
    if (u[2] <= 1)  u[1]=0;
    else if (u[2] <= 3) u[1]=0;
    else if (u[2]%2 == 0 || u[2]%3 == 0) u[1]=0;
    else {
        u[3]=5;
        u[1]=1;
        repeat(u[99],20000,20000){
        if (u[2]%u[3] == 0 || u[2]%(u[3]+2) == 0)
        { u[1]=0; break; }
        if(u[3]*u[3] > u[2]) break;
            u[3]+=6;
        }
    }
}
function verify {
    // make prime test
    u[2] = m[0];
    u[1]=0;
    if(m[0]>10000 && m[0]<20000) primetest();
    verify_bty ((m[0]>10000 && m[0]<20000) && (u[1]==1));

    // some randomness for POW function
    u[57] = m[1];
    u[56] = m[2];
    u[55] = m[3];
    u[54] = m[4];

    verify_pow (u[57], u[56], u[55], u[54]);
}
```

An ePL program usually starts with memory allocations which basically describe how the following six arrays are being allocated:

```
int i[] : signed ints (32 bit)
int u[] : unsigned ints (32 bit)
int l[] : signed longs (64 bit)
int ul[] : unsigned longs (64 bit)
int d[] : doubles
int f[] : floats
```

Your ePL program should have these 6 array initializers at the beginning, each one telling the VM how many elements it should allocate in which array. In this example it is:

```
array_int 0;
array_uint 1000;
array_long 0;
array_ulong 0;
array_double 0;
array_float 0;
```

Lines that allocate 0 elements to an array can be left out – this has been done in our example above. The arrays themselves do not need (and must not be) allocated by the programmer. The command array_int 1000; will already make sure that I[] is created with 1000 elements in it (indices 0 – 999). Please keep in mind, that there is no such concept as variable scopes, all arrays are global. Later, you will learn that this does not necessarily make it harder to code in ePL – be a little patient 🙂 In the next part, the body of your ePL program, you can define multiple functions. Functions have the following syntax:

```
function name {
    ...
}
```

Functions neither have arguments nor return values. This is not a big issue though since arguments and return values can be mimicked by merely using the global arrays. Again, later on, you will see how you still can easily and comfortably code in the way you are used to right now.

In our example we have defined three functions: main, verify and test_prime. While you can define as many functions are you like (as long as you stay within the maximum allowed complexity and size allowed for your program), you must have the main and the verify function.

The functions main and verify are needed both because ePL allows for an asymmetric calculation verification. To understand this a bit better, it is crucial to understand how working nodes are searching for solutions and how the rest of the network verifies these result to make sure that a scientist, who purchases computational power on XEL, actually gets what he has paid for: a correct result. Workers, those nodes who are searching for solutions to other tasks, are executing the main function whereas the verification of a solution to the algorithm is done by executing verify().

This has a very good reason: XEL allows you more complex programs with a reasonably longer execution time inside the main function that it does for the verification routine to ensure a smooth and lightweight verification on the nodes without stalling the network for too long. A good example where this asymmetrical verification could come in handy is an SAT solver. The main() function would carry a complex SAT solving logic, which is more expensive to execute, whereas the verification routine would assign the result to the boolean formula and check if it evaluates to true. If you want to see an example of how this can be done, feel free to take a look at our SAT solving demo.

Urgent warning: It is the developer's responsibility to code the logic in main and verify correctly. Mainly, it is essential to make sure that the logic (in particular, the verify_bty logic which we will read more about later on) in verify evaluates to true whenever the one in main evaluates to true. An incorrect implementation by the developer might cause results to be accepted, that does not reflect your desired outcome. Even more importantly, it can cause the Blockchain rejects valid results that have been found in main because they do not pass verify. Admittedly, this sounds still very abstract to you, but it will make more sense as we dig deeper into this tutorial.

For now, things are straightforward: in our particular case, there is no possibility to do an asymmetrical verification as finding primes is as complex as verifying them. To keep the code clean, we, therefore, add the entire logic into verify(), and let the main function call verify():

```
function main {
    verify();
}
```

Let’s start taking a look at the verify function:

```
function verify {
    // make prime test
    u[2] = m[0];
    u[1]=0;
    if(m[0]>10000 && m[0]<20000) primetest();
    verify_bty ((m[0]>10000 && m[0]<20000) && (u[1]==1));

    // some randomness for POW function
    u[57] = m[1];
    u[56] = m[2];
    u[55] = m[3];
    u[54] = m[4];

    verify_pow (u[57], u[56], u[55], u[54]);
}
```

Before you can understand what is going on here, it is essential that you understand how randomness is introduced into the execution of your code. ePL has one more array, the so-called uint m[12]; array – an array which consists of 12 unsigned integers. The first 10 unsigned integers are pseudo-random and provide you with 320 bits of entropy in total. m[10], the 11th unsigned integer, contains the round number and m[11], the 12th unsigned integer, contains the iteration that we are currently in. The concept of iterations will be explained at a later point in time. To better understand what is going on here, please take a look at the internal C logic that fills this uint m[12]; array:

```
// hash32[] is a random hash value provided by the protocol
// mult32[] is other information such as round and iteration 

// Randomize Inputs m[0]-m[9]
for (i = 0; i < 10; i++) {
  work->vm_input[i] = swap32(hash32[i % 4]);
  if (i > 4)
    work->vm_input[i] =
             work->vm_input[i] ^ work->vm_input[i - 3];
}

// Set m[10] To Round Number
work->vm_input[10] = mult32[1];
// Set Inputs m[11] To Iteration Number
work->vm_input[11] = mult32[2];
```

In our particular example, we will use the first integer m[0] to pick our prime candidate. Of course, you can use the entire 320 bit to create potential solution candidates to your algorithm. If you need more than 320 bit, you can even derive an infinite pseudo-random number stream from it:

```
u[2] = m[0];
u[1]=0;
```

In this part, we take over the first random value in m[0] and store it in one of our unsigned int array slots u[0] – remember, we have defined 1000 of them at the top of our program. Additionally, we initialize u[1] with the value 0: this slot will later hold the result of the prime-check.

```
if(m[0]>10000 && m[0]<20000) primetest();
```

In this part, we want to call the function conducting the prime test. Since we are only interested in primes between 10000 and 20000, we only need to call that function if the value in m[0] meets this requirement.

```
verify_bty ((m[0]>10000 && m[0]<20000) && (u[1]==1));
```

This part tells the backend system what to be considered a valid solution to your task/challenge – or a “bounty” how it is called in XEL’s terminology. This calls the function verify_bty () which takes a boolean expression as an argument which should evaluate to true for every valid solution that you want to have reported back. In this case, it is straightforward: if the number we have stored in m[0] is between 10000 and 20000, and the value in m[1] equals 1, then consider this a valid solution.

In a next step, we need to specify a condition for “proof-of-work” payments. XEL’s Blockchain works in a way that every task has two types of payments: the proof-of-work payments and the bounty payments. This has a very simple reason: very often, bounties are very hard to find and only limited to a hand full per job. Imagine you want to find an input to a CNF formula that evaluates it to true: if you have a bijective relation, there will be only one solution which, depending on the complexity of your formula, is very hard to find. While the discovery of such a solution should be awarded an attractive amount of coins (to keep workers motivated) you also want to keep them motivated by frequent, smaller payments. XEL introduces the concept of proof-of-work solutions to achieve that. These are solutions that do not solve your problem but which can be submitted by the workers to claim a tiny amount of coins as a reward for ongoing commitment. As you can probably imagine, payments per bounty should be set significantly higher than payments per proof-of-work. However, if you set this value too small, other tasks on the network might seem more lucrative and attract the processing power away from you. Proof-of-work packages, by the way, work on top of a re-targeting mechanism comparable to the block difficulty re-target in Bitcoin: The more processing power the network has, the harder it becomes to find such proof-of-work solutions so that, on average, the total number of proof-of-work solutions in the entire network converges to 10 per minute.

It is important to point out that it is the developer’s responsibility to provide the verify_pow() function with enough “evidence” that the worker has been actually working on your problem in the form of four unsigned integers. In our example here, we use this:

```
// some randomness for POW function
u[57] = m[1];
u[56] = m[2];
u[55] = m[3];
u[54] = m[4];

verify_pow (u[57], u[56], u[55], u[54]);
```

Now, while it is more than fine for our little demo, it is an example of the worst way to do it for productive use. Here, we take four integers from our random input entropy and use them to specify the proof-of-work reward condition. In this case, workers who are not interested in getting the bounty payments but want to work towards the proof-of-work solutions can omit the entire program logic and only check input integers m[1] ... m[4] to see if they have found a proof-of-work solution. This gives them a massive advantage over the more honest workers and allows them to get paid without really contributing anything useful to your task. It is always a best practice to use four values which depend “somehow” on the execution on as most of the program’s logic as possible. Then, only people who run the entire code can find those bounties. While this still does not guarantee that they will submit any bounty, there is no real reason why they shouldn’t since they have already performed all the work required for a bounty check. In the last step, we are going to take a look at the actual prime check:

```
function primetest {
    u[1] = 0;
    if (u[2] <= 1)  u[1]=0;
    else if (u[2] <= 3) u[1]=0;
    else if (u[2]%2 == 0 || u[2]%3 == 0) u[1]=0;
    else {
        u[3]=5;
        u[1]=1;
        repeat(u[99],20000,20000){
        if (u[2]%u[3] == 0 || u[2]%(u[3]+2) == 0)
        { u[1]=0; break; }
        if(u[3]*u[3] > u[2]) break;
            u[3]+=6;
        }
    }
}
```

What you see here is a simple trial-division based primacy check which works on m[2] (we have stored there our prime candidate before) as the input and sets m[1] = 1 if the candidate is a prime number, and m[1] = 0 otherwise. This value can be considered the “return value” of our function. One novelty you have probably noticed is the strange loop construct that you have probably never seen before:

```
repeat(u[99],20000,20000){
...
}
```

Before digging deeper into the characteristics of ePL’s loops, we need to take a look at what makes loops so evil. Remember, we want to have a programming language where it is known beforehand when it will terminate and how much memory it will use to avoid all sorts of DOS attacks where tasks are submitted that never terminate. However, why is it so hard to assess this with traditional loops? Well, traditional loops suffer from the so-called halting problem. The halting problem mainly is the problem of determining whether a program will terminate or whether it will run forever. While it is possible to do it for a C loop like this

```
for(int i=0; i<1000: ++i){
   ...
}
```

it already becomes impossible for simple constructs like this

```
int entropy = rand();
for(int i=0; i<1000: ++i){
   if (entropy < 10000) --i; if (entropy > 50000) ++i;
   ...
}
```

Here, the termination of the for loop is depending on the value of entropy. Of course, in this simple example, we can see with our own eyes what is going to happen. If entropy < 10000 then the loop runs forever. If 10000 <= entropy <= 50000, then the loop iterates exactly 1000 times, and for entropy > 50000 only 500 times. For the general case, however, there is no efficient way to algorithmically tell whether a C program will terminate, and when.

To eliminate this issue, ePL introduces a loop in the form of:

```
repeat(index,iterations,max_iterations){
...
}
```

Where the index is an array slot, the iteration is an expression which may be constant but which also may be an array slot of your choice, and the max_iterations is a constant value giving a maximum amount of iterations after which the loop exits. Internally, when translated back to C, our loop

```
repeat(u[99],20000,20000){
...
}
```

translates to

```
int counter = 0; // immutable by user's code
u[99]=0;
for(int i=0; i<20000; ++i){
    ...
    counter++;
    u[99] = counter;
    if(counter==20000) break;
}
```

The counter variable cannot be changed by the ePL code itself and will count to the maximum iteration count before it causes the loop to exit – regardless of what your code does. When parsing your code and checking if the complexity of your code is within certain bounds, it will always be assumed that your loop will run for the maximum amount of iterations that you have specified. Make sure to specify a sufficiently large number here, or your loops will exit prematurely.

## First Compilation
Now, that you have written your first program in ePL. Now, it’s time to test it. To test your program for syntactical correctness, you can use xel_miner. Assuming your program is called find_prime.epl, you can use this command to do the first run of your code:

```
./xel_miner --validate --test-avoidcache
                       --test-vm find_prime.epl
```

. The --validate flag tells xel_miner to look for the use of uninitialized variables which are not guaranteed to be zero at all times. You want to avoid that behavior. Furthermore, programs that do not pass this check will not be accepted on the Blockchain. The --test-vm flag tells xel_miner not to start mining, but to run (and check it for correctness) a local ePL file on your computer. If everything goes well, you will be presented with an output like this:

```
** Elastic Compute Engine **
   Miner Version: 0.9.6
   ElasticPL Version: 0.9.4
[16:31:29] DEBUG: Skipping recompilation to be blazing fast
[16:31:29] DEBUG: TestVM: block id '123456789'
[16:31:29] DEBUG: TestVM: work id '987654321'
[16:31:29] DEBUG: WCET main tester status: OFF
[16:31:29] DEBUG: WCET verify tester status: OFF
[16:31:29] DEBUG: TestVM: multiplicator '0000......0000'
[16:31:29] DEBUG: TestVM: pubkey '0000......0000'
[16:31:29] DEBUG: TestVM: target '0FFF......FFFF'
[16:31:29] DEBUG: storage size: 0
[16:31:29] DEBUG: Library 'job_987654321' Loaded
[16:31:29] DEBUG: Bounty Found: false
[16:31:29] DEBUG: POW Found: false
[16:31:29] DEBUG: POW Hash: 4E95CBA872172C0596BE7CB482703EE4
[16:31:29] DEBUG: Compiler Test Complete
[16:31:29] Exiting xel_miner
[16:31:29] Cleaning up
```

If everything went well, and you didn’t receive any compilation errors, it means that your code is syntactically correct – does not yet tell us much about whether it is doing what you want to do; this is something we will test later. However, first of all, you might be interested in how your program looks like when compiled into C. You can take a look by running:

```
cat work/job_987654321.h
```

The output will look like this:

```
void main_987654321(uint32_t * , uint32_t, uint32_t * ,
    uint32_t * , uint32_t * );
void primetest_987654321();
void verify_987654321(uint32_t * , uint32_t, uint32_t * ,
    uint32_t * , uint32_t * );

void main_987654321(uint32_t * bounty_found,
    uint32_t verify_pow, uint32_t * pow_found,
    uint32_t * target, uint32_t * hash) {
    verify_987654321(bounty_found, verify_pow,
        pow_found, target, hash);
}

void primetest_987654321() {
    u[1] = 0;
    if ((u[2]) <= (1)) {
        u[1] = 0;
    } else {

        if ((u[2]) <= (3)) {
            u[1] = 0;
        } else {
            if ((((((2) != 0) ? (u[2]) % (2) : 0)) == (0)) ||
                (((((3) != 0) ? (u[2]) % (3) : 0)) == (0))) {
                u[1] = 0;
            } else {
                u[3] = 5;
                u[1] = 1;
                int loop87;
                for (loop87 = 0; loop87 < (20000); loop87++) {
                    if (loop87 >= 20000) break;
                    u[99] = loop87;
                    if ((((((u[3]) != 0) ? (u[2]) % (u[3]) : 0))
                        == (0)) ||
                        ((((((u[3]) + (2)) != 0) ?
                                (u[2]) % ((u[3]) + (2)) : 0)) ==
                            (0))) {
                        u[1] = 0;
                        break;
                    }
                    if (((u[3]) * (u[3])) > (u[2])) {
                        break;
                    }
                    u[3] += 6;
                }
            }
        }
    }
}

void verify_987654321(uint32_t* bounty_found,uint32_t verify_pow,
    uint32_t* pow_found, uint32_t* target, uint32_t* hash) {
    u[2] = m[0];
    u[1] = 0;
    if (((m[0]) > (10000)) && ((m[0]) < (20000))) {
        primetest_987654321();
    }
    * bounty_found = (uint32_t)((((m[0]) > (10000)) &&
            ((m[0]) < (20000))) && ((u[1]) == (1)) != 0 ? 1 :
        0);
    u[57] = m[1];
    u[56] = m[2];
    u[55] = m[3];
    u[54] = m[4];
    if (verify_pow == 1)
        *
        pow_found =
        check_pow(u[57], u[56], u[55], u[54], &
            m[0], & target[0], & hash[0]);
    else
        *pow_found = 0;
}
```

We won’t go into detail and go through every line of it. Rather, we wanted to show you how you can compile your ePL program into C to understand how your ePL will be translated in the end and help you debug problems that you might face during development. For now, we have just verified that your code is syntactically correct but not if it produces the correct results. You can try a test-mining run by calling xel_miner like this (important: always use –test-avoidcache to disable caching, this may cause problems during testing):

```
./xel_miner --test-avoidcache --test-cont-bounty
                              --test-vm prime.epl
```

Depending on the complexity of your problem, you will have to wait some time until you get your first result. For testing purpose, it might be wise to lessen the requirements inside your verify_bty(...) call during your local testing activities to get your result quicker. In our case, the result should be displayed very quickly:

```
** Elastic Compute Engine **
   Miner Version: 0.9.6
   ElasticPL Version: 0.9.4
[20:08:09] DEBUG: Skipping recompilation to be blazing fast
[20:08:09] DEBUG: TestVM: block id '123456789'
[20:08:09] DEBUG: TestVM: work id '987654321'
[20:08:09] DEBUG: WCET main tester status: OFF
[20:08:09] DEBUG: WCET verify tester status: OFF
[20:08:09] DEBUG: TestVM: multiplicator '0000....0000'
[20:08:09] DEBUG: TestVM: pubkey '0000....0000'
[20:08:09] DEBUG: TestVM: target '0FFF....FFFF'
[20:08:09] DEBUG: storage size: 0
[20:08:09] DEBUG: Library 'job_987654321' Loaded
[20:08:09] >> STARTING CONTINUOUS TEST
[20:08:09] >> THIS MAY TAKE VERY LONG DEPENDING ON YOUR PROBLEM
[20:08:17] ran 5000000 iterations, still working ...
[20:08:18] ********************************
[20:08:18] FOUND A SOLUTION TO YOUR PROBLEM
[20:08:18]  we will dump all 12 input ints
[20:08:18] ********************************
[20:08:18] m[0]    =    18503
[20:08:18] m[1]    =    3294169169
[20:08:18] m[2]    =    2082860469
[20:08:18] m[3]    =    2676999779
[20:08:18] m[4]    =    18503
[20:08:18] m[5]    =    3095193060
[20:08:18] m[6]    =    3819580374
[20:08:18] m[7]    =    2676985380
[20:08:18] m[8]    =    3095178659
[20:08:18] m[9]    =    670245767
[20:08:18] m[10]    =    0
[20:08:18] m[11]    =    1
[20:08:18] DEBUG: Bounty Found: true
[20:08:18] DEBUG: POW Found: false
[20:08:18] DEBUG: POW Hash: 6F71E050C9A3B5A4B8A5686CB47EB17E
[20:08:18] DEBUG: Compiler Test Complete
[20:08:18] Exiting xel_miner
[20:08:18] Cleaning up
```

From this line here

```
[20:08:18] DEBUG: Bounty Found: true
```

You can see, that indeed a bounty has been found. Moreover, since we have used a straightforward way to define solution candidates, you can see the result right away:

```
[20:08:18] m[0]    =    18503
```

Indeed, it is a prime number between 10000 and 20000. Are you ready to push your first ePL program to the Blockchain?