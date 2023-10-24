---
layout: post
title: A Data-Oriented Approach to Software Careers and Job Search
tags: [bigdata, spark, python, analytics]
date: 2024-12-31
---

At the beginning of this year, I took some extra time to conduct a much more thorough job search than I 
usually do.  I took advice from good friends and used multiple spreadsheets and organizational tools.  The process
was long but really valuable, and I'd like to share it with others in the hope this might be valuable for you as well.  In fact, this process really has nothing to do per se with technical jobs, software, or data engineering, but you might find it easier to relate to my examples and the methodology if you are in the same field. 
(I'd love to get feedback from those of you reading this in non-software fields though!)

## How I usually look for jobs

As I age, even as I pick up a tremendous amount of experience, I realized that the way I approached job searches
had not significantly evolved.  I write this as a very experienced data architect and distributed systems engineer, but the way I would go about looking for jobs went something like this:

- Reach out to a friend or two, let them know I'm looking
- Find out about some potential opportunities
- Create a pros/cons list - actually I can't really find many previous lists, so many of these might have been in my head
- Talk to companies and get a sense of them

As I weighed different options, I found that the pro's and con's approach did not significantly help me as it always seems like I get stuck in "A has X, but B has Y!" type of dilemmas.  In the end it would come down to how I felt about choices - it was like an emotions based decision.

## This Search Had to be Different

At the end of last year my career seemed to be in a holding pattern.  I was in my second early stage startup in a row, where it felt like my gifts weren't really being used anywhere nearly to full capacity.  I felt like I needed to slow down and understand what was going on, and not just blame startups (which was an initial reaction).  

I started talking to friends and got some great advice - to take some time to really think wider, about goals, about previous places and what went well and didn't go well, and to think about what I wanted to do.  What I mean was, not just which company? but more like, what type of work should I do?  Should I keep working for somebody else or try to do consulting?   I had only limited experience with consulting before, but it felt like time to think really carefully.

There is often tremendous pressure, especially if you are the only breadwinner in the family, to get a job as quickly as possible.   On the other hand, I felt like if I made another poor choice, that my career would really get stuck, and I didn't want that to happen.  So, this search HAD to be different.  I had to take a really different approach and think about bigger meta-questions, and come to understand myself better.

## Analysis of Previous Companies

So the first step is to create a spreadsheet/table, where the left side row headers are different factors that might have played an important factor in your job performance and happiness.  On the top column headers are the names of the previous companies.  I would write notes in the cell and color the cell background based on whether I thought this factor was a positive contribution or a negative contribution.
For me, I used green to indicate a really positive factor, and red to indicate a very negative factor, and various shades and colours in between.  

Here is an example:

| Factor  |  Socrata | Tuplejump | Apple  | CompanyC |
| ------- | --------- | --------- | ------ | -------- |
| Language Fit | Good - Scala  | Good - Scala | Good - Scala | Fine - Rust |
| Domain Fit | Ok. Data co., but mostly PostGres | Great - Spark, Cassandra | Great - Spark, Cassandra, Kafka, FiloDB | Ok - mostly small data, some aggregation.  DE - new area. |
| Open Source | A tiny bit | YES! | No, but my main project was | A tiny bit |
| Data Scale and Performance are Immediate Concerns | No | Yes - scalable tech and clients had enough data | YES!  | Not really |

Note that for me, this wasn't the first step, as I started looking up different companies; however as my conversations progressed I realized that I really needed to understand better my past.

I think it's really important to pick whatever factors you feel could be influential, even if industry has taught you that it shouldn't matter.  
For me, for example, I have if my manager was a minority.  I had reason to believe, based on previous experience,
that this might be influential.  This is for you, not for the world, so pick whatever matters to you.

### A List of my Factors

I thought it would be helpful to list all the factors I used on the left hand side of the prev. co. analysis. 
The first group represents technical fit -- the things that for me, I believe makes a difference into how 
much value I add, how much I learn, how much fun I had, how well I did.  The first two should be self 
explanatory.  The last three are based on my background - since I specialize in systems that deal with tons 
of data, as well as architecture and data structures etc., I included those factors.

* Language Fit
* Domain Fit
* Data Scale and Performance are Immediate Concerns
* Big architectural design, solutions building
* Deep dive into algorithms/data structures/details etc

The second group has to do with personal relations.  You won't succeed at any job if you don't get along
with your boss, and for startups, the founders.  Relationships with your team, and especially with management,
are absolutely crucial.  The second item was a theory of mine that I wanted to test out if it made 
a difference, and you should be free to include such factors.  For me, I feel that I'm more comfortable with 
a manager who has a more similar background to myself.

* Know Boss/Founders
* Boss/Founders are minorities or Asian
* Fit in with team
* Fit in with management

The third group are company culture questions.  How supportive are they of open source, if that's important
to you?  How supportive of remote are they?  The last one has to do with the business model - I find that SaaS
products often require very fast release cycles and often requires oncall support, and that has a huge impact
on the kind of work that developers do. 

* Open Source
* Fully remote?
* Long Term Thinking and Investment
* SaaS/Rush to Fix/Oncall 

### What I Learned

I found that I did best in projects/teams/companies that have serious data challenges and allowed me to work at both a large architectural level, designing things, as well as at a very detailed level, working to define and develop new data structures, algorithms, etc.  Of course, the more data scale challenges you have, the more pressing it becomes to get architecture and data structures and algorithms correct.  At the same time, I found out from my history that SaaS teams are too focused on constant maintenance, feature updates, incidence handling, etc. to really focus on longer term improvement.  Such teams do not have the freedom to invest time and effort to significantly improve design, performance, etc. regularly - at least not at a level which really allows me to shine.  Another way to put it is that teams where priorities change too frequently, or have oncall requirements, devalue the parts of myself that allow me to shine the most.

What is really interesting is that the above factors seem to matter much more than say how well the company supports remote, or language/specific skill fit.  Good developers can easily pick up new langauges, whereas the above factors are really about what kind of work and style fits your strengths.  

The above factors are really dependent on the size of company, how they are structured and what phase products are in.  I did not specifically mention startups above, but you may notice that startups and early stage companies are often just fighting for survival.  They are trying to get customers however they can; they are fighting for product-market fit, and they often have to just do whatever is needed and may pivot from even week to week.  Such
an environment may be a great fit for some folks, but as someone who likes to dive deep and work on large projects, it's not as good of a fit.

I did really well at Apple, but perhaps I just got lucky.  Every project at Apple involves huge amounts of data, but I also got to work on greenfield database tech and do a large, very impactful project where I got to do tons of architectural, design, and low level stuff.  At the same time I'm also aware many large companies have really compartmentalized teams.   Big tech experience is really dependent on project and manager.  The good news about big tech is that if you're not happy on one team, it is usually not difficult to move.

### About Playing Politics and Climbing Ladders

A bit of advice that a friend shared was that when you are a senior individual contributor, politics will 
always be important.  If you are coming into a new domain as a senior contributor, then you are essentially 
only playing politics - as you won't be able to take advantage of the natural trust of being a domain expert.
If you know people, like management or founders, then you naturally have a huge advantage in terms of being 
trusted.  However, if you don't know anyone, and are not a domain expert, then what reason do people 
have to trust you?  :)

## Figuring out your Goals

At this point I felt like I had a good understanding of my past, but even as I spoke to companies I felt like
I needed to take a step back.   Was it worth it to work extra hours at this point in my career?  What about
consulting?   I wanted to think about top overall goals from the perspective of work.b  What did I come up
with?

* Given that I had a family and many interests, it was really important for me to be able to live a balanced life, to spend enough time with family, and other hobbies
* Improve people skills in all aspects
* Maintain the ability to design and think about distributed data systems, in detail
* Maintain thought leadership in a couple areas, including high velocity data processing systems
* Keep learning and be around others who I can keep learning with

### On the Importance of Improving the World and Tech

## Analysis of Candidate Companies

I started talking to some people and companies, and along the way started to do the above exercises to understand
the factors involved in my previous successful jobs, as well as examine my life goals.  The process helped
clarify my decision process significantly.

### The Candidate Company spreadsheet

### Eliminating Early-Stage Startups

### Arriving at Good Fit Finalists

## Networking as King

Networking is important, but please do the previous steps first.
Talking to people without a deep understanding of where you've been, and where you want to go, could 
lead to influences in the wrong directions.

## Last Thoughts
