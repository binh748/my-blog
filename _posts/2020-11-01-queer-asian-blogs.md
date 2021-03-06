---
title: "Using NLP to analyze queer and trans Asian blog posts"
categories:
  - Blog
tags:
  - queer
  - python
  - trans
  - asian
  - blog
  - natural language processing
  - unsupervised learning
  - plotly
  - VADER
  - spacy
  - NLTK
excerpt: "As part of my community organizing work with GAPIMNY, I'm an editor of a tumblr called the Gaysian Diaries."
---
## Table of Contents
1. [Introduction](#introduction)
2. [Summary](#summary)
3. [Challenges](#challenges)
4. [Conclusion](#conclusion)

## Introduction
As part of my community organizing work with [GAPIMNY](http://www.gapimny.org), I’m an editor of a tumblr called the [Gaysian Diaries](https://gaysiandiaries.com). The Gaysian Diaries posts monthly stories from queer and trans Asian Pacific Islanders (API): by sharing these stories, others in the community can connect, learn, and feel less alone. I have also contributed to the Gaysian Diaries, having written a couple posts. There’s a similar tumblr called the [Gaysian Third Space](https://gaysianthirdspace.com), and I’m friends with the creators.

These blogs made a unique dataset for my Metis NLP/unsupervised learning project. I was curious to see what insights I could discover considering I already know the blogs well. This was my most personal project yet.

## Summary
I had considered various avenues of analyses such as using Google Analytics or creating a word embedding model, but in the end, I chose to do topic modeling and sentiment analysis. I scraped close to 400 blog posts from tumblr using BeautifulSoup and stored the data in MongoDB. I then used NLTK and Spacy to do the text pre-processing and tf-idf vectorizer to vectorize my documents. To perform topic modeling, I used non-negative matrix factorization (NMF) to get eight topics such as body image, family, and racism and assigned one topic to each blog post based on which topic carried the largest weight. For sentiment analysis, I used the VADER package, which calculates sentiment scores in the range of -1 to 1 where -1 is the most negative sentiment and 1 is the most positive.

I discovered a surprising result from my analysis: the sentiment scores for each topic were slightly positive even for difficult topics like body image and racism. This was counterintuitive, so I dug deeper by doing a sectional sentiment analysis. I divided the blog posts into thirds (beginning, middle, end) and calculated the average sentiment score for each section. I saw that there was an upward trend in sentiment as you traverse the sections in order, with the last section having the most positive average sentiment score. This means that the blog post writers tended to concentrate their negative feelings in the beginning and middle sections before ending on a hopeful, optimistic tone. This trend holds true at the aggregate level (i.e. across all blog posts) and at the topic level.

![All sentiment trend graph](https://user-images.githubusercontent.com/62628676/100012330-16c62100-2da1-11eb-97e1-1fb9143319d2.png)
![sentiment trend by topic](https://user-images.githubusercontent.com/62628676/100010363-27c16300-2d9e-11eb-8919-455b522ac64b.png)
<span class="caption">You can see the upward trend in sentiment scores, with the last section being the most positive. This trend is more pronounced in certain topics than others: for example, in the identity topic, you can see a sharp increase in the sentiment score at the end.</span>

I think this finding points to the resiliency of the writers and perhaps to the community as a whole, that despite the hardships they faced, many hold hope for a better tomorrow.

![From despair to hope](https://user-images.githubusercontent.com/62628676/95486383-ecd7ad00-0960-11eb-8a22-4f27ebb1f846.png)

<span class="photo-credit">Photo by <a href="https://unsplash.com/@roadtripwithraj?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Road Trip with Raj</a> and <a href="https://unsplash.com/@lephunghia?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Nghia Le</a> on <a href="https://unsplash.com/@roadtripwithraj?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

## Challenges
I love clean, informative data visualizations. It’s one of the reasons I got hooked into data science in the first place—my eyes light up every time I come across a New York Times’ D3 visualization.

For this project, I wanted to up my data visualization game by making an interactive plot using the Plotly visualization package. I used t-SNE, a dimensionality reduction technique to visualize higher dimensional data, to prepare my eight document-topic clusters for visualization; for the actual visualization, I built a scatter plot on Plotly with a custom, interactive tooltip.

![t-sne doc-topic clusters with custom tooltip](https://user-images.githubusercontent.com/62628676/95398790-b6545080-08d4-11eb-9a2b-de9472541208.png)

Figuring out how to customize the tooltip took some time as I had to pour through the documentation and get familiar with Plotly’s syntax. (A lot of coding is reading through documentation!). A nice thing is that as I read through more and more documentation of various packages, I get faster at finding the information I need. :smile:

One small frustration I had with Plotly was that it doesn’t allow you to change the location of the x-axis and y-axis labels unlike Seaborn and Matplotlib. I like my x-axis and y-axis labels to look like the below figure since I learned from Metis instructor Alice Zhao that for English-readers, we first read the left-most text; moreover, vertical text, which is how y-axis labels are usually displayed, is difficult to read. I totally agree. No one wants to turn their head to read text, lest they want a stiff neck. Maybe one day Plotly will add such functionality, but until then, I’ll just cry a little inside whenever I see my vertical y-axis labels on Plotly.

![alice-zhao-data-viz-example](https://user-images.githubusercontent.com/62628676/104978710-7ff89800-59d0-11eb-8b82-e7f6fc449aab.png)
<span class="caption">This is how x-axis and y-axis labels should be positioned and oriented for easy reading. Visualization from <a href="https://www.thisismetis.com/">Metis data science bootcamp</a>.</span>

My other visualization challenge was figuring out how to create a sentiment map where color-coded squares represent the sentiment trend of a blog post. The color-coded squares would be ordered top-down, left-right to follow a blog post's sentence order. Some quick searching showed me that no native plot exists, so I had to get creative. I read before that a data scientist is supposed to be a hacker: they should have good enough programming skills to put together whatever they need, be it an app, visualization, or website, even if it may not be production quality.

Hack I did! I commandeered Seaborn’s heatmap plot, which naturally displays squares of data, to create my sentiment map. It required that I preprocess my sentences into a matrix of sentiment scores and then run that matrix through Seaborn’s heatmap plot. I also discovered that I could create thick horizontal lines that run through the sentiment map to divide my map into the three sections I was analyzing (beginning, middle, and end). I felt awesome being able to hack away and produce a custom visualization. Imagine trying to do that on Excel—it would be painful.

![Sentiment map](https://user-images.githubusercontent.com/62628676/95486449-024cd700-0961-11eb-9d5f-9cec5d3554f9.png)

## Conclusion
When I finished this project, I shared my findings with my queer and trans API friends and the editors of the Gaysian Diaries and Gaysian Third Space tumblr. It generated a lot of discussion and questions. For example, they were also surprised by my finding that the topic sentiment scores were positive across all topics, but came to understand why when I showed them my sectional sentiment analysis.

That’s the neat thing about doing NLP/unsupervised learning—even if you’re working on texts that you’re familiar with, you’re still bound to learn something new. Data science opens up new ways of analyzing texts beyond standard textual analysis. As someone who majored in humanities, it makes me happy knowing that the two disciplines, humanities and data science, can be used together to further human knowledge. That’s why Stanford has a whole field of study called the [digital humanities](https://shc.stanford.edu/digital-humanities). Maybe my next NLP data science project will be analyzing some of the literature I read in college. :grin:

All code on [GitHub](https://github.com/binh748/queer-asian-stories).
