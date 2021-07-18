# Backtest Methodology for the DOMINUS strategy
## Preliminary
### Goal
The goal is to find the most profitable settings for the DOMINUS strategy.
### Scope
The current DOMINUS version (v2) has 10 different variables that affect entries and exits (not including the 9 variables for the ADX and the BB). We built an automation that iterates though the permutations of those settings and save the results in a `.csv` file for data analysis.
## Too many info
### The permutational problem
Those of you that have experience with permutations will already have spotted the first big hurdle of this approach. For the rest, here is what we face:
* The `Heiken Ashi` setting can take 2 values (true or false).
* The `Length` setting is a positive integer that can take infinite values, so we have to limit is. Let's limit it to 18 options for now (3-20).
* The `PcntChange` is a positive real number that pose tha same problem. Let's limit it to 3 options (1, 2, and 3).
* The `Rating for Entry` setting is a real number that ranges from 0 to 1. We will limit it to 13 option (0.2 to 0.5 in intervals of 0.05).
* The `Rating for Exit` setting is a real number that ranges from 0 to 1. We will limit it to 13 option (0.2 to 0.5 in intervals of 0.05).
* The `TP % for Short Entries` setting is a real number that ranges from 0 to 100. We will limit it to 31 option (0.2 to 0.5 in intervals of 0.05).
* The `SL % for Short Entries` setting is a real number that ranges from 0 to 100. We will limit it to 31 option (0.2 to 0.5 in intervals of 0.05).
* The `Entry_numbers` setting is an integer that ranges from 0 to 100. We will limit it to 4 options (1, 2, 3, and 100).
* The `LongEntry` setting can take 2 values (true or false).
* The `ShortEntry` setting can take 2 values (true or false).

To calculate the total number of iterations (given the limits we set above) we have to multiply the number of different options per setting together. That means `Total iterations = 2 * 18 * 3 * 13 * 13 * 31 * 31 * 4 * 2 * 2 = 280,642,752`. We can currently run about 48 tests per minutes. If we decide to test all of those iterations we will need `Total_time =  280,642,752 / 48 = 5,846,724 mins or 97,445 hours or 4,060 days or 11 years`. 
### See you in 11 years
Of course you want the result in the next few days and not in a decade.

One solution is to fire up multiple instances of our backtester, but we will need 4,060 of those to have the result in a day, and computational power is not free. 

So another solution must be found.
### What is a heuristic
> A heuristic or heuristic technique (/hjʊəˈrɪstɪk/; Ancient Greek: εὑρίσκω, heurískō, 'I find, discover'), is any approach to problem solving or self-discovery that employs a practical method that is not guaranteed to be optimal, perfect, or rational, but is nevertheless sufficient for reaching an immediate, short-term goal or approximation. Where finding an optimal solution is impossible or impractical, heuristic methods can be used to speed up the process of finding a satisfactory solution. Heuristics can be mental shortcuts that ease the cognitive load of making a decision.
### Divine and conquer
If you've already played with the DOMINUS settings yourself, and especially if you have disabled the `ShortEntry`, you might have noticed that it is not affecting the number or timing of the long entries. Given that the DOMINUS strategy is based on the One Tool to Rule them All, this might not come as a surprise to you.

What that means though for our case is that we can split the problem in two.
* Find the best settings for long entries
* Take the best result from above and try to find the best short entries only for this setting.
### Why does this make any difference?
As we saw on the ["The permutational problem"](#The-permutational-problem), each extra setting multiplies the total iterations by it's options. So if we remove a setting we can divide the total.

So, setting the `ShortEntry` to false, the total drops to 140,321,376.
Then, setting the `LongEntry` to true, the total is 70,160,688.  
Then, by not iterating through the `SL % for Short Entries` (since it does not affect the longs), we can divide by 31 to get 2,263,248.  
Doing the same for the `TP % for Short Entries`, we divide again by 31, for a total of 73,008.  
Finally, keeping the `Entry_numbers` (another setting that only affects short entries) a constant, we divide by 4 for a final total of 18,252.

With our current speed, we can iterate through 18,252 options in **6 hours and 20 minutes**.
## Running the tests
### Did you run any tests or are you all talk
We did run tests on the `BINANCE:ETHUST` pair on the 1 hour chart.
### Were are your results for that?
You can find them [here](data/results_1626389047259.csv).
### So, did you found the best setting for long, right?
Not yet. A good testing requires both backwards and forwards testings. Given the limits imposed by Tradingview, we could only test from 01/01/2019 on the 1 hour chart. So, we decided to run a backwards test from 01/01/2019 to 31/12/2020, take the best 100 results and then run forward tests from 01/01/2020 to 17/07/2021 on those.

We then used a weighted average of the two results and came to a conclusion.
### Finally. Now gimme.
You can find the forward tests [here](data/results_1626429053825.csv) and the final results [here](data/final.csv).
## Time to short
### But I want to short as well
We do as well. For that, we have to keep the previous settings a constant and iterate through the settings that affect only the short entries. That is `Total_iterations = 31 * 31 *4 = 3,844 or about 80 minutes`. We did this for a few of the top "long" settings that we found.
### Are you going to give us the results or should we hunt you down?
We are finally on the home stretch. Here are the results for the [`true, 13, 2, 0.30, 0.40`](data/results_1626487483582.csv), the [`true, 14, 1, 0.35, 0.45`](data/results_1626527514686.csv), and the [`true, 7, 1, 0.35, 0.45`](data/results_1626535826398.csv).
## Conclusion
### What are the best settings?
The "best"? We do not know. Do you only care about net profit? Do you care about drawdown? Profit Ratio? Profitability %? Everyone has a different definition about "best".
### You are useless and this whole document was a waste of my time.
We know.
