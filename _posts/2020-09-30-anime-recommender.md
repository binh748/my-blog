---
title: "Anime Recommender"
categories:
  - Blog
tags:
  - anime
  - recommender system
  - flask
  - cloud
  - docker
  - collaborative filtering
  - content-based filtering
---
I’ve been watching anime since I was a kid, but for most of my life, I had been cautious about letting people know I love anime. It was only until I moved to New York City after college that I began to shed my insecurity and embrace my anime-loving self.

To get into the Metis data science bootcamp, I interviewed with a Metis alum who asked me what I wanted to do for my passion project, which is the last project that a student does to complete their 5-project portfolio. I answered confidently and without shame that I wanted to build an anime recommender system.

![Ookiku Furikabutte funny picture](https://wallpapercave.com/wp/wp5109395.jpg)
<span style="font-size: .8em; font-style: italic;">One of my favorite anime about a high school baseball team with big hearts and big dreams: Ookiku Furikabutte.</span>

## Summary
My anime recommender combines content-based and collaborative filtering. The data is scraped from [myanimelist.net](https://myanimelist.net/). For content-based filtering, I hand-selected the anime features from the data I scraped, and the user features are averages of the anime features for the anime the user has watched. For collaborative filtering, I used non-negative matrix factorization on a user-ratings matrix to learn six latent features for my user and anime embeddings. For both filtering systems, I used cosine similarity to find the nearest anime embeddings for every user embedding. For a more in-depth discussion on content-based versus collaborative filtering, see [My Take on Recommenders](#my-take-on-recommenders).

I deployed my recommender on a Flask app. The app allows users to choose how adventurous they want their recommendations to be. Setting the recommendation type to "more adventurous" will show the user more collaborative filtering recommendations whereas setting the recommendation type to “less adventurous” will show the user more content-based filtering recommendations. By allowing users to select their recommendation type, they have optionality in finding recommendations that fit their preferences.

![Flask home page](https://user-images.githubusercontent.com/62628676/93408976-d78cc880-f863-11ea-9978-c921b2a56945.png)

![Flask recommender page](https://user-images.githubusercontent.com/62628676/93409135-33efe800-f864-11ea-9c10-0396cda3428d.png)
<span style="font-size: .8em; font-style: italic;">Screenshots of my Flask app.</span>

## Challenges
#### Web Scraping

I did web scraping for my regression and NLP/unsupervised learning Metis projects, but the datasets for both those projects were quite small, at most a few thousand data points. This time, I wanted to scrape anime ratings from 120,000 users for my collaborative filtering system. What made matters worse was that myanimelist.net was a prohibitive site to scrape from since after a certain number of requests, it would temporarily block me as a bot. This forced me to only scrape anime ratings from 100 users at a time and then pause for 3 minutes before scraping again. I parallelized my web scraping function with a package called joblib so I could scrape as quickly as possible before pausing, but even then, I calculated that it would take 3.75 days of running my web scraper non-stop on my Macbook Pro to get all the data. I don’t have time for that!

To speed things up, I learned how to deploy my web scraper across four Google Cloud Compute Engine instances in addition to my laptop. Learning how to do that was no easy task. First, I had to learn how to build a Docker container that would install all the Python/package dependencies for my web scraping script and run the script. I toiled away for a few days figuring out Docker, and then once I got my container up and running in the cloud, my web scraping script would run for a bit and then crash…

![Defeated pusheen](https://www.clipartkey.com/mpngs/m/34-344284_pusheen-cat-clipart-lazy-lazy-pusheen.png)
<span style="font-size: .8em; font-style: italic;">What I felt like when my Docker container wasn't working.</span>

After some frantic Googling, I found out that because I was using Selenium and a headless Chrome driver to web scrape, I needed to allocate more shared memory to my Docker container than the default 64 mb. That did the trick (I ended up upping the shared memory to 12 gb—maybe a bit overkill, but I wanted to be extra safe).

So what happened to the 3.75 days? Well, if you only count the time that the web scraper was running, I was able to do it all in one day. However, learning how to use Docker took a few days in itself, so in total, it still took about 3.75 days to get all the data. Womp (a favorite expression of mine). I came full circle, but now I know how to use Docker and the cloud, so I’m smiling. :smile:

![Gon Freecs smiling](https://carboncostume.com/wordpress/wp-content/uploads/2017/08/gon-freecs.jpg)
<span style="font-size: .8em; font-style: italic;">Gon is better at smiling than me.</span>

#### Flask

Making a simple and beautiful Flask app to show off my anime recommender was incredibly rewarding. I also learned a lot getting my hands dirty with HTML and CSS. But the next time I need to deploy a model, I don’t plan on using Flask because it’s time-consuming and tedious: just figuring out how to align HTML elements could take me an hour.

Next time, I plan on using Streamlit. Streamlit lets you create an app with just Python code and markdown, dramatically reducing the complexity of app-building since then you don’t have to mess around with HTML/CSS/JS to build a nice-looking app. Flask is more powerful and customizable than Streamlit, though, so I’m glad to have started with Flask first. The more tools in my belt, the better.

![Streamlit or Flask?](https://user-images.githubusercontent.com/62628676/94727114-3c4c2680-032c-11eb-8a73-0c87c9114fe2.png)

## My Take on Recommenders
Two well-known ways to build a recommender system are content-based and collaborative filtering. In content-based filtering, you recommend similar anime to anime that the user has watched. In collaborative filtering, you recommend anime that fit the user’s preferences based on how that user’s anime ratings compare with other user’s anime ratings—there are technical details that I’m leaving out here, but the main point is that collaborative filtering recommendations tend to yield more diverse and interesting recommendations than content-based filtering.

When I first learned about these two filtering techniques during the bootcamp, I wanted to figure out how I could combine the two in my anime recommender. The way I did it was I got recommendations independently from both filtering techniques and then scored those recommendations based on user input on how adventurous they want their recommendations to be. If the user wants more adventurous (i.e. more diverse) recommendations, the content-based filtering recommendations will be weighed higher and vice versa.

I think letting users choose their recommendation type can provide a better user experience. It definitely made me wonder, though, why the streaming platforms I have subscriptions to (e.g. Netflix, Crunchyroll) don’t have such a feature.

![Crunchyroll’s recommendations for me](https://user-images.githubusercontent.com/62628676/94749921-f2c30200-0352-11eb-8da4-911a25323f79.png)
<span style="font-size: .8em; font-style: italic;">These are the anime recommendations I get on Crunchyroll, but there’s no option to change the recommendation type.</span>

Maybe it’s because my assumption is wrong, and most users don’t care about being able to choose their recommendation preferences (I wouldn’t be one of those users, though!). Regardless, I built this recommender for my own enjoyment and with my own vision for how a recommender should work—if I were building this recommender for a company, I definitely would do testing to validate my assumptions.

## Conclusion
I learned a lot from this project: how to build a Docker container, leverage Google Cloud instances, create a recommender using both content-based and collaborative-filtering, and build a Flask app. Though there were definitely some days where I was banging my head, I kept motivated because I did the project on something I love: anime.

Unfortunately, it’s not possible for me to test my recommender full-scale to see if the recommendations are meaningful for people, but for fun, I did test it on myself and my friend. My recommender helped me discover two new anime: [Tengen Toppa Gurren Lagann](https://myanimelist.net/anime/2001/Tengen_Toppa_Gurren_Lagann) and [Hyouka](https://myanimelist.net/anime/12189/Hyouka), and I’m watching Tengen Toppa Gurren Lagann right now and enjoying it. For my friend, his top anime recommendation was [Tokyo Ghoul](https://myanimelist.net/anime/22319/Tokyo_Ghoul), which my friend said he’s been meaning to watch for a while now. My friend takes his anime very seriously, so I’m glad my recommender didn’t disappoint him.

Maybe one day I’ll be a data scientist at Crunchyroll helping them improve their recommender system. If my favorite anime characters can accomplish their dreams, so can I.

All code on [GitHub](https://github.com/binh748/anime-recommender).
