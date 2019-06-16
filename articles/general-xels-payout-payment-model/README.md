# XEL's Payout/Payment Model

XEL has two types of payments. To understand how XEL's payments work, we have to shed some light on the entire workflow - beginning with the broadcast of a task to the network, to the way miners work on tasks and broadcast the solutions.

#### The Workflow

The entire workflow typically starts with scientists composing jobs (or task) which contain all the logic to solve for the actual problem as well as information regarding the rewards being offered to incentivize participation in solving the problem. Then, this job is broadcast to the Blockchain for others to work on.

Once these jobs are available on the Blockchain, these jobs are downloaded by those nodes which contribute their computational resources to the network - let's call them "workers" for now. Workers that find actual solutions – which we also call "bounties" – are rewarded with a "**bounty reward**" by the scientist paid in units of the underlying Crypto-currency. These bounty rewards are set by the scientist when the job is created and should be ideally set to an amount that attracts participation in running their job. As the network grows, competition among scientists to attract participation in running their job increases; therefore, scientists will want to calibrate their bounty rewards to a fair market value to ensure enough interest in their job.

While "bounties" are the primary incentive to attract computation nodes to work on a job, there is a risk that nodes could be working on jobs where no bounty solution even exists. This could be due to an intentional malicious act by the scientist, or simply a bug in the job’s code. To mitigate this risk, scientists will be required to additionally provide "**PoW rewards**". These PoW are defined as moderately hard tasks which are easily verifiable and which are randomly found by the workers at a certain rate, regardless of how hard or easy the underlying task is, in order ensure workers stay motivated. PoW rewards should be calibrated to roughly match the average electricity cost of running a computational node to alleviate any concerns of participants that they could potentially lose money by running the job. Currently, the network adapts itself so that, on average, 10 PoW payments are made each minute - for all jobs in total, not individually. While the PoW price can be set by the scientist arbitrarily, and will vary among different tasks, it is worth to mention that several tasks might require a higher PoW payment due to a more complex program which needs significantly more time to generate one bounty in relation to other, potentially easier tasks, that are currently alive.

#### A Simple Calculation Example

Imagine there are two jobs active at the moment. Job 1 pays a "bounty" of 100 XEL, and a "PoW reward" of 0.01 XEL. Job 2 pays a "bounty" of 200 XEL, and a "PoW reward" of 0.02 XEL. Also, imagine that one "run" of Job 1 takes 1 second, and one "run" of Job 2 takes 2 seconds (because it is more complex).

On average, every minute there should be a total of 10 PoW rewards in the system. Since you can run Job 1 at twice the speed of Job 2, you will get around 6,66 PoW from Job 1, and 3,33 PoW from Job 2 every minute - that is, roughly, 0.0666 XEL from each of the jobs. Now, if the scientist behind Job 2 had put in a lower PoW reward, you would have no incentive of running his task at all, since sticking to Job 1 would be far more profitable.

In the end, if you solve the actual task and submit the solution, you get 100 XEL from Job 1 and 200 XEL from Job 2. However, there is no guarantee that a solution even exists. Without the periodic payouts of the "PoW bounties," the incentive to participate would be too low.
