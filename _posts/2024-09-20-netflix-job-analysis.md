---
layout: post
title: Netflix Job Analysis
date: 2024-09-21
future: true
---


# Background

In 2020, during the COVID pandemic I had moved into a leadership role at Braintree and it was the first time I really thought about trying to define "my leadership style".  My entire career, I've focused on "doing the right thing" and doubling down on whatever existing intuition I had built from watching others.  But for this exercise, I tried to divorce my current style from one that I'd want to lead with.  Simply, I started from zero.  I did the zero-to-leadership book list that probably everyone has gone through at some point, you know:

1. [The 7 Habits of Highly Effective People](https://www.amazon.com/Habits-Highly-Effective-People-Powerful/dp/0743269519)
2. [How to Win Friends and Influence People](https://www.amazon.com/How-Win-Friends-Influence-People/dp/0671027034)
3. [The Five Dysfunctions of a Team](https://www.amazon.com/Five-Dysfunctions-Team-Leadership-Fable/dp/0787960756)
4. [Leaders Eat Last](https://www.amazon.com/Leaders-Eat-Last-Together-Others/dp/1591848016)
5. [The Lean Startup](https://www.amazon.com/Lean-Startup-Entrepreneurs-Continuous-Innovation/dp/0307887898)
6. [The Phoenix Project](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/1942788290)
7. [Mythical Man Month](https://www.amazon.com/gp/product/B00B8USS14)
8. [Deep Work](https://www.amazon.com/Deep-Work-Focused-Success-Distracted/dp/1455586692)
9. [The Innovators Dilemma](https://www.amazon.com/Innovators-Dilemma-Revolutionary-Change-Business/dp/0062060244)
10. [Working Backwards](https://www.amazon.com/Working-Backwards-Insights-Stories-Warriors/dp/1250236505)
11. [Extreme Ownership](https://www.amazon.com/Extreme-Ownership-U-S-Navy-SEALs/dp/1250067057)
12. [Good to Great](https://www.amazon.com/Good-Great-Some-Companies-Others/dp/0066620996)


There were more but you get the idea. The most comical moment for me is that I read [Managing Up by Rosanne Badowski](https://amzn.to/4dv2rUd) when in reality what was recommended to me was [Managing Up (by Mary Abbajay)](https://amzn.to/3Zx26wJ).  But by reading Roasanne Badowski's book, I learned of Jack Welch and that started me down a whole different path, centered on a guy I really knew nothing about.  And I couldn't stop pulling at this thread - I read [Winning](https://amzn.to/47BwjwP) and [Straight from the Gut](https://amzn.to/3BezMFq) and I watched these interviews ([vid1](https://www.c-span.org/video/?186828-1/winning), [vid2](https://www.youtube.com/watch?v=PxU6Z0BgyWM)) a dozen times.  Jack Welch and I had similar childhoods in suburban Massachusetts, he had a paper route (so did I), he caddied during the summers (so did I) but somehow after high school he went on to become the youngest CEO ever at GE (at age 30) and I, well, didn't.  Fascinating stuff, highly recommend those books if you're interested in one of the most popular (and sometimes controversial) CEO's of the 20th century.

So, like any good engineer I am in an information gathering mode and I'm going full bore. In case you're not familiar with Jack Welch, he wrote the playbook on how a company should set values and then hold people accountable to them.  So that leads me to [Netflix's Culture Doc](https://jobs.netflix.com/culture) which is exactly what Jack Welch would have wanted, it's filled with behaviors and values that are easy to understand but set the culture in a big way.  I was convinced that I would fit in at Netflix, that those values were what I wanted too, and after browsing their job board for a few months, I realized something...

# Netflix's Job Board

[Netflix's job board](https://jobs.netflix.com) is basically set up with a FE displaying a search bar.  Open your network inspector, type a search term and hit Go and you'll see the entire page is powered by a simple GET request to

```bash
https://jobs.netflix.com/api/search?q={search_term}
```

The response is JSON.  Note that as of 2024, this API still works but I can see now that the website is using a new endpoint via `explore.jobs.netflix.net`.

So I thought, well maybe the best way to get a highly coveted job at Netflix was to be the first to know about it a new role and then be the first to apply!


# Job Search Lambda

I built a very simple [serverless](https://www.serverless.com) ruby application that runs a few times a day, looking for anything matching for an "engineering manager" and if it finds something, it will send me an email that looks like this:

![netflix_email](/public/imgs/netflix_email.png)

You can find all the [code in the repo](https://www.github.com/allcentury/netflix-jobs). Including an Easter egg in my [User-Agent](https://github.com/allcentury/netflix-jobs/blob/main/handler.rb#L66) string that I thought someone at Netflix might see and get a chuckle out of.

# Results

So this has been running now since May 27th, 2021 and while I have moved on with my life and really found my leadership style (separate post coming) I still get the emails.  Not only do I still get emails but I've been storing the results of the API the entire time because I wanted the lambda to only email me if there were new jobs that I hadn't seen before.

So with my S3 bucket results, I thought  maybe I should do something with this data.

## Netflix EM Job Trends

First, I think it's worth calling out that job postings over time are on a continuum but are mostly trending up, especially in the earlier part of the year.  Here's what that looks like:

![netflix_em_postings](/public/imgs/netflix_em_postings.png)

## Netflix Org Trends

Looking at trends over time, gives us a view into what Netflix is investing in and how that translates into jobs.  Not only that, but my search has been focused on "engineering manager" roles so one can assume that these roles, especially multiplied, are a leading indicator into org changes and growth (each EM likely having 5-10 direct reports).  Sure some might be backfills because of attrition or performance issues but I think the data is still interesting.

The top 3 teams/organizations for the last 3 years are:

1. Core Engineering - 117+ EM roles
2. Data Science & Engineering - 29+ EM roles
3. Client and UI Engineering - 24+ EM roles

Here's what that looks like over time:

![netflix_orgs](/public/imgs/netflix_orgs.png)


## Netflix Location Trends

Next, I thought it would be interesting to see where these jobs are located. Here's what that looks like:

1. Los Gatos, CA - 166+ EM roles
2. Remote - 51+ EM roles
3. Warsaw, Poland - 16+ EM roles

We can see a small bump in remote roles over time but Los Gatos is still where the majority of hiring is taking place.

![netflix_locations](/public/imgs/netflix_location.png)
