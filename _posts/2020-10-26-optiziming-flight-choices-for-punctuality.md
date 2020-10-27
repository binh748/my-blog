---
title: "Optimizing flight choices for punctuality"
categories:
  - Blog
tags:
  - logistic regression
  - flights
  - SQL
  - PostgreSQL
  - API
  - class imbalance
  - travel tips
---
One of my greatest joys in life is traveling (and I certainly miss it during the pandemic). My mom tells me in Vietnamese that I was born with “running feet”. Unfortunately, I can’t remember what the Vietnamese expression she used was, so I’ll have to ask her again one day. The thing is I like traveling when my feet are on the ground, not when they’re in the air. I think flying is a pain (maybe I’ll change my mind when I fly business class, but that’s not happening anytime soon), and it becomes even more of a pain if my flight is delayed or cancelled. I tremble when I think back to when my partner and I discovered that our flights back to New York City from Puerto Vallarta, Mexico were cancelled because of Hurricane Harvey. When I found a 2015 U.S. flights dataset on Kaggle that’s used to classify whether a flight is delayed or not, I thought this would be the perfect classification project for me and maybe I can learn a thing or two about how to optimize my flight choices for punctuality.

![plane in the sky](https://images.unsplash.com/photo-1547382806-7c4a0346ee4f?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1500&q=80)

## Summary
I constructed a logistic regression model to predict whether a flight would be delayed. Delay here means that the flight was delayed, diverted, or cancelled. The data for my model came from a sample of 100,000 flights from the 5.8 million flights Kaggle dataset and a weather dataset that I pulled from Visual Crossing Weather’s API.  To evaluate my model, I chose the F1 metric, which is the harmonic mean of precision and recall since I am optimizing for both: I want my model to classify as many of the delayed flights as possible (recall), but to also be accurate in how it's classifying the positive class (precision).

I chose logistic regression because I wanted high model interpretability so that I can use the model’s coefficients to learn how certain factors affect flight delay probabilities. I did run a random forest and gradient boosted models alongside logistic regression for comparison, but did not achieve better performance.

I had a class imbalance problem since only 20% of the flights in my sample were delayed. To combat the class imbalance, I set my logistic regression class weights to balanced so that the flights that were delayed (i.e. the minority class) had a higher contribution to the cost function; moreover, I used a grid-search type method to find the probability threshold for my logistic regression model that maximized my F1 score: that threshold was 47% instead of the default 50%.

When I evaluated my logistic regression model on the test data, I achieved decent recall, but poor precision (.28). One suggestion I got from my Metis instructor to improve my F1 score was to see if flight delays earlier in the day or week could increase the chances of nearby future flights being delayed as a sort of domino effect. This is a good suggestion to explore and makes sense since one reason for delayed flights is waiting for the aircraft to arrive, which is more likely to happen if there are delays from previous flights.

What’s important in data science is not to just model data, but to use the model in a practical way, for example, to support decision making. In my project, I used the logistic regression coefficients to help me understand some of the major factors that lead to flight delays. First, I bucketed my features into three groups to analyze them: airline categorical features, weather features, and datetime features. I then looked at the largest coefficients within each group and wrote travel tips based on those coefficients. For example, looking at the coefficients for my weather features, I saw that visibility was the most important feature since its coefficient had the largest magnitude, so I wrote my tip “If visibility is poor, download more Netflix shows than usual.” You can see all my tips in the graphics below. It was fun when I presented these tips to my Metis classmates and instructors since they typed into the Zoom chat anecdotal evidence that validated my tips.

![tip-1](https://user-images.githubusercontent.com/62628676/93727297-90b31180-fb88-11ea-890d-13b7356c9d17.png)

![tip-2](https://user-images.githubusercontent.com/62628676/97218602-46ccd500-179f-11eb-8e5d-6e1b0d766298.png)

![tip-3/4](https://user-images.githubusercontent.com/62628676/97218657-56e4b480-179f-11eb-8c61-24ed09370a91.png)

## Challenges
To store the Kaggle flights data, which came in three separate .csv files (flights.csv, airlines.csv, airports.csv), I used PostgreSQL. I created the SQL database from scratch and defined the database schemas. That wasn’t too difficult. What was difficult was writing my queries with multiple table joins so I could query the data I needed for analysis and store it in a pandas DataFrame. I initially tried writing the queries late at night in one big swoop, but I somehow got duplicated records that didn’t make sense. Frustrated, I went to bed. The next morning, I woke up fresh and wrote the queries one step at a time, making sure each additional step of my query produced the results I was expecting before building on.

That worked! I learned a couple of lessons from that experience:

1. Coding late at night is not wise.
2. Start SQL queries small and then build up. My Metis instructors had told me this before, but I didn’t listen and paid a price for it.

```sql
SELECT
       f.*,
       a.airline AS airline_name,
       o.airport AS origin_airport_name,
       o.city AS origin_airport_city,
       o.state AS origin_airport_state,
       o.latitude AS origin_airport_latitude,
       o.longitude AS origin_airport_longitude,
       d.airport AS destination_airport_name,
       d.city AS destination_airport_city,
       d.state AS destination_airport_state,
       d.latitude AS destination_airport_latitude,
       d.longitude AS destination_airport_longitude
  FROM flights AS f
  LEFT JOIN airlines AS a
    ON f.airline = a.iata_code
  LEFT JOIN airports AS o
    ON f.origin_airport = o.iata_code
  LEFT JOIN airports AS d
    ON f.destination_airport = d.iata_code
 ORDER BY RANDOM()
 LIMIT 100000;
```
<span style="font-size: .8em; font-style: italic;">This is the query I used to join the tables together and pull a random sample of 100,000 flights to model in Python.</span>

The other challenge I had was finding a trustworthy weather dataset to add to my model. There are a bunch of weather APIs out there, and I was almost going to use a free one until I saw that the site was not secure. My partner is a cybersecurity engineer and has rightfully scared the bejeezus out of me regarding unsecure websites.

After doing more research, I found Visual Crossing Weather’s API to be secure and simple enough for my needs, although I had to pay for it since the free version only gave me 500 API requests a day. I was afraid of running into a lot of difficulties and bugs figuring out the API, but it wasn’t too bad, and I was able to get all the data through the API quickly. I remember working at D. E. Shaw & Co. and hearing about “API this and API that” from the developers I worked with, and I was always curious how to access data from one, but now I know. Through the Metis bootcamp, a lot of technology which I had only understood at a high level has been demystified as I learn how to use it myself.

![Guy raising his fist in the air on a cliff with a beautiful morning](https://images.unsplash.com/photo-1519834785169-98be25ec3f84?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=80)

## Conclusion
I hope to revisit this project someday since there are a number of things I could do to expand on it: I can scale up my model to incorporate the entire 2015 flights dataset (will probably use PySpark or Dask to do the large-scale data processing), improve my F1 score by doing additional feature engineering like seeing if earlier flight delays affect later flights, and make a simple Flask to predict flight delay probabilities based on a user’s flight information.

Because of the pandemic, I won’t be on a plane anytime soon. When the pandemic does end, I’ll be gleefully traveling again and now know a number of tips to keep in mind if I want to choose flights that have a higher probability of being on-time. :wink:

All code on [GitHub](https://github.com/binh748/flight-classification).
