---
title: "Predicting global box office gross for animated films (regression)"
categories:
  - Blog
tags:
  - linear regression
  - Python
  - regression
  - animated films
  - japan
  - usa
  - challenges
  - mistakes
  - lessons learned
  - web scraping
excerpt: "This project was a failure and a learning experience."
---
## Table of Contents
1. [Introduction](#introduction)
2. [Methodology & Results](#methodology--results)
3. [Challenges, Mistakes, & Lessons Learned](#challenges-mistakes--lessons-learned)
4. [Conclusion](#conclusion)

## Introduction
This project was a failure and a learning experience.

For my first Metis solo project, I had to build a regression model using data scraped from the web. I chose to work with movie data scraped from imdb.com. The U.S. and Japan are the big titans in the animation industry (think Disney and Studio Ghibli), so my goal was to build two linear regression models: one to predict the global box office revenue for American animated films and the other to predict the global box office revenue for Japanese animated films. With the two models side-by-side, I could compare them to learn which revenue drivers differ between American and Japanese animated films—that was the intent at least.

I went into this project with high aspirations since I thought I had set up an interesting problem; instead, I faced challenges and made mistakes that frustrated my goals. Experiencing failure is difficult, but I learned a lot from this project. Sometimes what you learn from failure is much more valuable than what you learn from success.

## Methodology & Results

I scraped 2,846 animated films from imdb.com (about evenly split between American and Japanese animated films) using the package BeautifulSoup as my HTML parser. While cleaning the data, I had to drop ~1,800 films because they were missing my target (global box office gross). Moreover, I had to drop another 80 films because they were listed as being both from Japan and the US, which would confound my separate models. Another data issue was that of the remaining ~500 American and ~500 Japanese animated films, 30% and 90% of the American and Japanese films were missing budget data, respectively. I filled the missing budget data with the median budget, but using the median budget for 90% of Japanese films would cause a problem once I start modeling (I discuss this further in my [Challenges, Mistakes, & Lessons Learned](#challenges-mistakes--lessons-learned) section).

My approach to modeling was to create baseline models with a small number of features and then iteratively add more features from the remaining scraped data or feature engineer additional features. To verify that the added features improved my model without causing overfitting, I used 5 k-fold cross validation to check my R-squared across my training and validation sets. I was particularly focused on feature engineering binary time-based features (e.g. is_summer_release, is_xmas_release, is_golden_week_release) since I had read literature that mentioned that box office revenue is sensitive to release dates. Based on my Japan model, a movie released in the summer would see a 6 mn USD increase in global box office gross, but a movie released during Golden Week, which is a week of holidays in Japan spanning the end of April to early May, would see a 10 mn USD drop in global box office gross, which is a curious finding.

Unfortunately, I couldn’t trust my Japan model because it had a validation R-squared of .29 and a test R-squared of only .06, which meant that on the test set, my model barely performed better than using the target mean for all predictions. My US model did better with a validation R-squared of .69 and a test R-squared of .66. Because of the poor performance of my Japan model, I could not compare the two models to gain insights on the different revenue drivers for American and Japanese animated films.

![animation regression results table](https://user-images.githubusercontent.com/62628676/102293349-eba09e80-3f14-11eb-966d-3ba2eb5b4964.png)

## Challenges, Mistakes, & Lessons Learned
My Japan model didn’t perform well because of challenges I encountered in the scraped data and because of my own mistakes. I like to record and reflect on the mistakes I made so I don’t make them again. Usually, I record them privately in my Trello or in a Google Doc, but I’m glad to share them here. As they say, sharing is caring. :heart:

1. **Data Quality:** Of the ~500 Japanese films in my dataset, 90% didn’t have budget values, which is a crucial feature to model global box office gross. For the American films in my dataset, I did have the majority of their budget values, and budget alone explained ~50% of the variance in my target; in other words, regressing global box office gross on budget alone yielded a test R-squared close to 50% in my American model. imdb.com is an American website, so it’s not surprising that it wouldn’t have great data on foreign films. Perhaps there exists a Japanese version of imdb.com, which would have more complete budget data for Japanese animated films, but then I’d need to pray that Google Translate works well enough or be able to read Japanese—the good thing is that I’m slowly learning Japanese right now: I finished learning hiragana and am in the middle of learning katakana. One day, I’ll find the Japanese version of imdb.com and add in the missing budget data!

2. **Residual Analysis:** Despite missing budget data, I could have done more to improve my Japan model. Instead of analyzing residuals throughout my modeling process, I only analyzed them the night before my project was due. But when I did, it was eye-opening: the movies that had the highest residuals in Japan were movies produced by famous animation studios like Studio Ghibli and CoMix Wave Films, which produces Makoto Shinkai’s (the director of *Your Name*) films. The same was true for my American model where the highest residuals were movies produced by Disney/Pixar and Illumination. Unfortunately, I didn’t scrape animation studio, so I couldn't add it as a feature to my models; also, it was too late to edit and re-run my web scraper.

![japan movies largest residuals](https://user-images.githubusercontent.com/62628676/102674735-ff424400-4164-11eb-84fd-33e0c2fd2151.png)

![u.s. movies largest residuals](https://user-images.githubusercontent.com/62628676/102674724-f5204580-4164-11eb-8fbe-c3db6b5fd95e.png)

3. **Presentation:** I cringe when I look back at my PowerPoint presentation for this project because, without mincing words, it’s bad. I rarely made slide decks at D. E. Shaw, so it’s not a skill I’ve developed much professionally. If you look at the slide below, you can see why my Metis instructors gave me a lot of feedback on my presentation. As I watched my Metis peers present their regression projects, I was amazed by how many of them had incredible PowerPoint/data visualization skills. It inspired me so much that when Metis instructor Alice Zhao gave her lecture on presentations and data visualizations, I sat there like a student in the front row of class (it was a Zoom lecture), taking notes on everything she said. I’m happy to say that I’ve come a long way since this presentation. On Metis graduation day, the instructors took turns singing the praises of each student. When it was my turn, my instructor noted how since my regression project, I aimed to improve my presentation skills. I had come a long way she said and I should be proud of myself. That compliment made my day. :smile:

![bad animation regression slide](https://user-images.githubusercontent.com/62628676/102298524-46d78e80-3f1f-11eb-857c-aa2fd7c3ddc3.png)
<span style="font-size: .8em; font-style: italic; display: block;">Using text to explain numerical relationships is not the way to go!</span>

Below are the lessons I learned above in a tl;dr format:

1. Think carefully about what data I need to scrape beforehand since re-scraping data is time-expensive.
2. Do residual analysis throughout the modeling process, not at the very end.
3. Find additional/better data sources if the data source I'm working with is inadequate.
4. Give myself enough time to work on my PowerPoint presentation and be deliberate in the design, data visualizations, and storytelling.

## Conclusion
Each successive project I’ve done at Metis, I’ve improved at something: my presentation skills, my ability to work with new technologies, my modeling skills, and more. I landed on my face with my first solo Metis project, but learning from that failure set me up for greater success for later projects. I got up, and I’m still running toward my data science dreams.

All code on [GitHub](https://github.com/binh748/animation-regression).
