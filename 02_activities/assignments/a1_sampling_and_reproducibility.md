# ASSIGNMENT: Sampling and Reproducibility in Python

Read the blog post [Contact tracing can give a biased sample of COVID-19 cases](https://andrewwhitby.com/2020/11/24/contact-tracing-biased/) by Andrew Whitby to understand the context and motivation behind the simulation model we will be examining.

Examine the code in `whitby_covid_tracing.py`. Identify all stages at which sampling is occurring in the model. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.

Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?

Modify the number of repetitions in the simulation to 100 (from the original 1000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.

Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times

# Author: Faiyza Alam

```
1. Read the blog post [Contact tracing can give a biased sample of COVID-19 cases](https://andrewwhitby.com/2020/11/24/contact-tracing-biased/) by Andrew Whitby to understand the context and motivation behind the simulation model we will be examining.
--> Blog post has been read. It certainly is interesting to have watched discussions around these statistics for covid tracing play out in real time during the pandemic, and to have seen the impact this had on real decisions to trace or not to trace. Knowing how faulty these reports can be really puts that time into an interesting context...

2a. Examine the code in `whitby_covid_tracing.py`. 
--> Examined. 

2b. Identify all stages at which sampling is occurring in the model. 
--> First, a random sample is being infected. This is RANDOM SAMPLING applied to the WHOLE POPULATION, specifically based on a PROBABILITY of infection rate (in this case, 10%).
--> Then, PRIMARY CONTACT TRACING occurs. This is also RANDOM SAMPLING, and it is based on a 20% PROBABILITY specifically of the INFECTED population. In other words, every infected person has a 20% chance of being traced. 
--> After this, SECONDARY CONTACT TRACING is also a sampling event. However, this one is NOT based on chance. Rather, it is DETERMINISTIC or a NON-PROBABILITY MODEL in a sense, because it is based on some fixed criteria. This is the one that introduces some bias, because it is more likely two people from the weddings are infected (since it is a larger gathering) compared to the brunch, so the weddings are more likely to be 100% contact traced.
--> Finally, the statistics of these sampled groups are collated. This is not a form of sampling
--> The code also runs this whole process to generate 1000 iterations, which is also not explicitly a form of sampling (because we are not taking a subset of a group). Rather, we are simply seeing the different outcomes over 1000 trials and then plotting them. 

2c. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.
--> For INFECTION:
    --> Initially set to FALSE (i.e. no infection) for all individuals in the population.
    --> ATTACK_RATE = 0.10. In other words, a 10% probability for any one individual in the whole population,    across all events, to be infected.
    --> Random selection goverened by np.random.choice, which chooses random elements in an array. Here, it is choosing from ppl.index (the index given to an individual, so it is essentially picking out people from the data frame of the whole population). It is going to randomly select 0.10 times the length of the ppl array. In other words, 10% of the total number of individuals. 
    --> SAMPLING SIZE: 10% of the whole population. In this case, 0.10*1000 people (800 from brunches, 200 from weddings) gives us 100 people. 
    --> DISTRIBUTION: Since this is population-wide and every individual has the same chance of being infected, this is a UNIFORM distribution.
    --> SAMPLING FRAME: Whole population.
    --> From the blog post, this is "Suppose that exactly 10% of people at every event are infected, regardless of the type of event." The initial infecting event. 
--> For PRIMARY TRACING:
    --> Here, we choose from those who are infected and decide via some probability threshold (in this case, 20%) how likely it is that they are traced.
    --> SAMPLING FRAME: Only those who are infected.
    --> SAMPLING SIZE: From those who are infected (100 people) we are tracing 10%, so 20 people.  
    -- We will be selecting these 20 people RANDOMLY, via np.random.rand. This generates a random humber between 0 and 1 for every infected individual (so 100 of them). If that number is less than 0.2, we say they are traced. 
    --> DISTRIBUTION: The underlying randomness of giving each individual a random number from 0 to 1 is UNIFORM. Once we make the distinction of <0.2 is traced, vs >0.2 is untraced, we separate our population into two distributions (this gives us a BINOMIAL distribution)
    --> From the blog post, this is the primary tracing event. "Suppose that contact-tracing is imperfect, and that due to faulty recall of patients and staff shortages, an infection has only a 20% chance of being traced to a source event. Call that “primary contact tracing."
-- For SECONDARY TRACING:
    --> This is not random. This is deterministic sampling, based on some threshold.
    --> First, we look at ONLY those who were PRIMARILY traced. Then, we check what event they were at, and count the total number of people at that one event, all via event_trace_counts = ppl[ppl['traced'] == True]['event'].value_counts()
    --> Then, we apply some threshold where we look ONLY at those events that have at LEAST two people at the SAME event infected (this is place into a list, via events_traced = event_trace_counts[event_trace_counts >= SECONDARY_TRACE_THRESHOLD].index)
    --> Then, we look at everyone at the event. If their status is infected, we change traced to true (because we have now traced everyone at that event): ppl.loc[ppl['event'].isin(events_traced) & ppl['infected'], 'traced'] = True
    --> SAMPLING FRAME: All infected individuals who attended an event where ≥ 2 people were already traced.
    --> SAMPLING SIZE: Not a fixed number. We will have sampled everyone at the events that were flagged in the previous step. 
    --> DISTRIBUTION: Nothing specific...because there is no randomness here. It is based on some fixed threshold. 
    ---> From the article: "If two infections are independently traced to the same source event, a special effort is made to test every person who attended that event, with the result that 100% of infections associated with that event are identified."

3. Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?
--> It does not...the wedding ones overlay around the true proportion of infections from weddings, rather than being overestimated. It does have a similar kind of lower tail closer to 0, but the overarching distribution is otherwise more or less on top of the true distribution, not much higher. 

4. Modify the number of repetitions in the simulation to 100 (from the original 1000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.
--> Seems reproducible to me....Not in the sense that it is EXACTLY the same, because there are definitely variations (and this one has a bit more variation than the 1000 case), but it is reproducible in the sense that I'm still getting a distribution of traced over the real distribution for weddings, with a low tail close to 0 as well.

5. Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times.
--> Since there is some randomness to this code, the way to make this reproducible is to add a seed. 
--> I did np.random.seed(36). The "36" is arbitrary (it is my favourite number)
--> I ran this a few times, and it is indeed exactly reproducible. 

```


## Criteria

|Criteria|Complete|Incomplete|
|--------|----|----|
|Altercation of the code|The code changes made, made it reproducible.|The code is still not reproducible.|
|Description of changes|The author explained the reasonings for the changes made well.|The author did not explain the reasonings for the changes made well.|

## Submission Information

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

### Submission Parameters:
* Submission Due Date: `23:59 - 09/04/2025`
* The branch name for your repo should be: `assignment-1`
* What to submit for this assignment:
    * This markdown file (a1_sampling_and_reproducibility.md) should be populated.
    * The `whitby_covid_tracing.py` should be changed.
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sampling/pull/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-1`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via the help channel in Slack. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.
