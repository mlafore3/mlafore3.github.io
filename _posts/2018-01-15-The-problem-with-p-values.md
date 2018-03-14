---
layout: post
title:  "The problem with p-values"
date:   2018-01-15
excerpt: ""
tag:
- blogpost
---
Like most people, I used to consider statistics a plague that I should try to avoid. However, my time as a medical researcher transformed my opinion because I realized how paramount accurate statistics were in making robust and reproducible conclusions. Luckily, I had come into contact with a professor that transformed my take on the p-value. He did this by showing me just how misleading it can be, which I hope to share with you.

Let’s consider the student’s t-test, which is a procedure that determines if two groups of data are different. For example, perhaps a company wants to know if their product is bought more by men or women, a doctor wants to know if their patients taking a treatment has improved their symptoms, or a group of friends are planning their next vacation and want to know if hotels cost more in New York City or San Fransisco.

Let’s consider our last example. Our hypothesis is that there is a relationship between hotel price and the city we visit. Our null hypothesis is that there is no relationship between the price of the hotel and the city that it is in. Assuming that this data would follow a normal distribution, let’s simulate some data. We’ll pretend that four best friends compiled the price per night of 100 hotels in San Fransisco and found that the average price was $100 with variation of +/- $25 (n=100, u=100, sd=25).

![jekyll Image]({{ site.url }}/assets/img/p-values/random_samples.jpeg)

These four friends plotted the resulting distributions in a histogram with a density plot overlaid. Note that each friend has observed a different distribution of hotel prices, determined by which hotels they looked at. Perhaps friend 2 has more expensive taste whereas friend 4 is more thrifty. Nevertheless, it is important to consider what this means for p-values when we compare two distributions?

Let’s say that these four best friends repeated this for hotels in New York City and tried to determine which city had more expensive hotels by completing a t-test. One of two things can happen. Case 1: The prices of the hotels in the two cities are different or Case 2: there is no difference.

For Case 1, let’s say that the friends collected data from 100 NYC hotels and found that they cost on average $80 a night with standard deviation of $25. Completing the students t-test and calculating the p-value, the friends compare their findings.

![jekyll Image]({{ site.url }}/assets/img/p-values/hotel_diff.jpeg)

Given their extremely small p-values, all four friends conclude that hotels in NYC are cheaper than San Fransisco. Importantly, all four friends got different p-values depending on the data that they collected. Let’s say that these friends are really popular and have 500 additional friends that are planning to join them on the trip.

![jekyll Image]({{ site.url }}/assets/img/p-values/diff_pvalues.jpeg)

If they all completed this analysis and recorded their p-values, we could expect a distribution of p-values seen on the left. Although we mostly get a p-value below our 0.05 threshold, there are times where a friend gets a p-value that is above 0.05. In these instances, that friend would have concluded that there was no difference between hotel prices, even though there is. ALARMING!

Even more alarming, let’s consider Case 2, where the data suggests that there is no difference between hotel prices in NYC and San Fran.

Here, three out of the four friends would have concluded that there is no difference between hotel prices. However, friend number four received a p-value that is below 0.05 and would have thought that there is a difference, even though there isn’t. Who knows? Maybe this would have caused a huge fight and they wouldn’t be friends anymore…that’s just sad.

![jekyll Image]({{ site.url }}/assets/img/p-values/hotel_same.jpeg)

Before the three friends unfriend the fourth friend, they decide to consult with their 500 additional friends to compare their p-values.

![jekyll Image]({{ site.url }}/assets/img/p-values/same_pvalues.jpeg)

They are shocked to discover that when there is no difference between the prices of hotels, the p-value is effectively random! Although most of the friends would have reached the same conclusion, there is a group that would have concluded that there is a difference… even though there wasn’t one.

Recall that the correct interpretation of a p-value is: 
	Assuming that your null is true, a p-value quantifies the probability of seeing a data point at or beyond your current observation.

Basically, you calculate a 95% confidence interval of where you expect your data to be. If you observe a value that is outside these bounds, it is unexpected as it is within the most extreme 5% of your data and supports your null hypothesis. If you want to be more lenient to variation, you might calculate a 97.5% confidence interval and only allow observations with p-values of 0.025 to disapprove your hypothesis. The best way to combat the fickle p-value is to increase the number of replicates that you do and see if your p-value jumps around or if it stays consistently low. Here I have coded an interactive visualization that will allow you to change the parameters of the two distributions and see the resulting p-value distribution. If you want to read more, I would recommend this paper. Code to regenerate this can be found in this repository.