Stocks, Stonks and NLP
Project 3: Web APIs & NLP

### Overview

This project seeks to put my skills in building classification models, webscraping, API's, and Natural Language Processing to the test. The general overview of my worflow was as follows: scraping posts from subreddits, cleaning and processing the text data into a useable format, building and testing many different classification models on the data, and evaluating my model's performance against my problem statement. 

### Introduction

The focus of my work was two massively poular subreddits, r/stocks and r/wallstreetbets. r/stocks is home to 3.5M followers and the topics found in the subreddit range from investment advice, market or political analysis, commentary, discussion over trading strategies and a fair amount of interesting and technical analysis. r/wallstreetbets has risesn to a level of infamy in recent years and has a following of over 11M. Where amateur investors in the r/stocks community would be asking for a "rate my portfolio" r/wsb "apes" would be sharing "loss porn" or robinhood captures of steep losses taken in leveraged derivative positions to the public as a rite of passage of sorts. Where r/stocks poses general investment advice or a specific company analysis r/wsb shills "stonks" without the need for as much in-depth analysis when you can type out "HODL" next to a GME/AMC meme. Despite all the noise and memes, the tightly correlated subject matter of the two subreddits I thought would make for an interesting challenge when it came to fitting a classification model on the corporus of text compiled from each page.

### Problem Statement

My initial problem statement was therefore a challenge; could I accurately predict with a classification model whether a investing related post belonged to r/stocks versus r/wallstreetbets?

My problem statement evolved in the early steps of my process and I added a second task to my checklist after creating a second corpus of text data compiled from only stock tickers from each subreddit; could I accurately predict whether a post belongs to r/stocks versus r/wallstreetbets based on the mention of stock tickers alone?

### Data Science Process

Data Collection:

I used [Pushshift's](https://github.com/pushshift/api) API to scrape tens of thousands of posts from  r/stocks and r/wallstreetbets. Using the requests library as well as Beautiful Soup I automated the scraping process using a function that pulled 100 posts at a time and reset the loop based on the created_utc time stamp from the 100th post. Rinse & Repeat. 

Data Cleaning:

There were a handful of anomolies within the scraped data I had to deal with before conducting EDA and building models. For any post that had been deleted by a subreddit moderator or removed by the author my scraping would return "deleted" or "removed" for a post's body text while keeping the original title of the text intact. These two instances accounted for up to 60% of scraped posts across the two subreddits. Additionally, if a post's body contained only an image or gif or other media type the body text would return a NaN. To deal with these instances while keeping as much of the data intact I created a new series for all text data that compiled a post's title text and body text in 1 while ignoring any removed, deleted or nan body text data. 

Preprocesssing:

Because of the unique text found in both subreddits, particularly in r/wsb with all of it's unique slang, as well as all the tickers found across the subs I chose to preprocess the text data by hand while relying on a vectorizer afterwards primarily for the tokenization aspect. To go about this I defined a series of custom built functions using the string library and regex expressions to comb through the text data, creating copies of stock tickers using the [Reticker](https://pypi.org/project/reticker/)library found there, removing extraneous whitespace, newline characters, a swath of nonsense symbols, and particularly getting rid of shared url's which I did not want to include in my text analysis. It was here that I developed a second corpus of ticker text only which set. me up for two levels of analysis and modeling thereafter.

Exploratory Data Analysis:

To conduct EDA I used count vectorizer and tfid vectorizer on the entire corpora of tokenized all_text data and ticker_text data and put them into each into dataframes. I added a column that tagged the subreddit as well. I was primarily concerned with looking at feature tokens by frequency of usage using the count vectorized dataframes and also the importance of feature tokens by using the tfid vectorized dataframe. It was by analyzing the tfid vectorrized dataframe that I learned looking at tickers could be very particularly useful for classification modeling on their own because certain tickers such as GME, AMC, TSLA and AMD had high feature importances and I knew from commbing the r/wallstreetbets sub that these were hot topics there that weren't necessarily as hot in the r/stocks sub.

Modeling:

My baseline accuracy test based on the split of the data by subreddit was r/stocks 0.500453 and r/wallstreetbets 0.499547, almost dead even since I was able to retain nearrly every data entry in my data cleaning and processing. 

The simmplest prediction model you can build would simply guess the majority class, r/stocks in our case, but this would probably only be right ~50% of the time. Therefore for a model I can build to be successful in any degree it would have to outperform an accuracy score of 50%.

The first models I built used a logistic regresssion classifier on count vectorized data. This served as my preliminary model since it was the easiest to constrruct and quickest to fit on the data. On the all_text data I scored 0.90 on the training split and 0.75 on the test split. On the ticker_text data I scored 0.845 and 0.714 on train/test splits respectively. Immediately I could tell the model was overfit but I wanted to test many different types of classifiers rather than tinker and tune the LogReg model. Beyond outperforming the baseline accuracy test pretty significantly, for a prelimary model I was also pleased that the classifier performed almost just as well on ticker data alone as it did using all text data.

After testing model performance using count vs. tfid vectorizes I decided going forward I would favor the tfid vectorizer in model construction. I went on to fit two MultinomialNB classifiers on the all text data and the tfid mNB model outscored the cv mNB model as well. The tfid mNB model scored 0.836 on train data and 0.759 on test data. 

After fitting a decision tree classifier and fittting extra trees & random forest classifiers I decided not to include them in my analysis since they performed significantly worse than my earliest models (5-15% worse accuracy scores). 

I found the AdaBoost classifier was a strong classification option scoring 0.73 on train data and 0.72 on test data. I particularly favored these results since there was a negligent amount of overfitting and wanted a model that could be used most confidently on unseen data for future applications such as a streamlit app that takes user input from an amateur trader and classifies them as being an everyday amateur investor versus belonging to the tribe of wsb apes. With gridsearch I was able to improve the AdaBoost model performance by optimmizing parameters scoring 0.78 on train and 0.74 on test, however the higher bias it allowed in the model led me to favor the first boosted model. 

On ticker text the AdaBoost model scored 0.73 on train and 0.69 on test data. This ticker text model offered the lowest risk of overfitting as well.

The final models I built used a support vector machine classifier (SVC). The SVC was the highest perfoming model based on accurracy scores alone. On all text data it scored 0.98 on train data and 0.79 on test data with a 'poly' kernel and degree of 2. On ticker text data SVC scored 0.83 on train data and 0.69 on test data. Although it was such a strong performer it still offered a rrisk of overfitting and. I didn't want to allow too much bias in my model of choice to deal with variance in potential unseen data applications. 

### Conclusions and Next Steps:

To frame the model process in the lens of the problemm statement I was certainly pleased with the accuracy scores I was able to achieve, classifying all text data with up to ~80% accuracy and ticker text data alone with up to ~70% accuracy. Considering the closely related subject matter of the subreddits and these margins of accuarcy above the baseline particularly when it came to my second challenge of predicting class based solely on the mention of a ticker was more than satisfactory in confirming prredictive ability using classification models and NLP.

To take this project further I would like to build out an interactive streamlit app as afore mentioned and even spring into building a stock price performance application based off the NLP of these subreddits or wsb in particular potentially in conjuction with sentiment analysis which I didn't get into. 

In refining this body of work and my process I would use more data and attempt to fit different types of classifiers such as XGBoost. Additionally, I think I could improve my use of vectorizers to limit the inclusion of more feature tokens from analysis and see if n_grams which I summarily chose to avoid from analysis could've improved predictive capabilities. 
