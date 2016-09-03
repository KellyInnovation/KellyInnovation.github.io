---
layout: post
title: Struggle Leads to Deep Learning, but Only After Asking for Help
---

I am learning more every day to stop and ask for help.  Often, in bootcamp, if I am struggling with a new concept, so are others. 

I find that I can learn more deeply when I program an aspect on my own.  This can be with assistance from online coding lessons or examples, forums, or existing code.  Alternatively, trial and error with an understanding of key elements results in greater retention once learned.  I believe it is the active process of arriving at the answer through hard work that plants certain concepts in my brain concretely.  

While I love the challenge to solve the problem, there are times when I do not possess the knowledge, and cannot find online help to create a solution.  This was one of those weeks: …enter JSON.

My struggle started with an afternoon lab.  At first glance, the tasks seemed reasonable exercises in data manipulation:
- Write a script that will
    - Show the Most Expensive Item
    - Show the Least Expensive Item
    - Show the Total Revenue
    - Show the Total Profit
    - Show the 10 Best Sellers
    - Show the Number Sold by Each Department

What first seemed an intriguing puzzle using Python3 coding, quickly became another lesson in asking for help.  

Merely importing the JSON data into Python took several minutes.  Once there, …what to do?  
-Confirm the data is there by printing - check.
-Learn that a JSON file opened through Python behaves like a dictionary - check.
-Learn that dictionaries are not iterable - hmm, that one didn’t quite make sense.
-Learn that you cannot use more than one index to describe a position in a dictionary - see above.

How to get at the data - big problem #1.

Firstly, one of my classmates with more experience tried to explain - didn't quite get it.
Looked at online examples, but didn’t find enough to draw conclusions.

I asked for help.  

One answer, with one example, on the problem I had been staring at for 45 minutes was what I needed. 

An empty list and a for loop.  In the for loop, include the key name for the value of interest: 
prices = []
def accessing_values():
	for item in json_data:
		prices.append(item[‘price’])

Asking for help was the key to solving this problem.

I got it.  Include the key name to access that value….great!

Wait, then came Friday’s morning warm-up: dictionaries again.

I thought I understood it, but not well enough.  This time, the magic trick (at least it seemed like magic the first time I understood) of using the key name did not apply.  The key I needed to name were numbers 1 to 31 (for days of the month).  Surely, doing 31 for loops was not the best route…

Nothing.  I understood all of the online dictionary examples in which they used the key name to access the values.  With this exercise, there was not a easily distinguishable key name.  Back to square one.  How do I get at a value in a dictionary?

Again, the instructors were the only way I could learn this concept.  Matt Smith did a great job of explaining the process.  He took us through several examples, starting with real-world perception, then mimicking the concepts from the previous JSON in Python3 experience (i.e. using key names to call values), then to pulling values without known key names.  

It all finally made sense at an even deeper level than before.  This just teaches me once again, if I have a question, it is always best to ask.  A first hand explanation from someone with more experience, can solve the problem and help me learn more quickly.  The slow frustration and final relief of an explanation does make a concept stick, once learned.  

If not, there are always my notes, or asking for help again.
