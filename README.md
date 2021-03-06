# AC209a Twitter Bot Detection Project

This repository contains all of the notebooks that were developed for the purpose of detecting bots on Twitter using data science and machine learning techniques, for the AC209a fall semester class at Harvard University.

Access the Github Pages website [here](https://mrdragonbear.github.io/AC209a-Twitter-Project/).

The notebooks outlined the procedure of data acquisition using the **tweepy** Twitter API library for Python, followed by exploratory data analysis and model development. Natural language processing techniques such as topic modeling and sentiment analysis were also implemented to obtain both tweet-level features user-level features. The machine learning techniques utilized in this study were:

- Logistic Regression
- LDA/QDA
- Random Forest
- Boosting
- K Nearest Neighbors
- Feed Forward Artificial Neural Network
- Support Vector Machines
- Stacking (Meta Ensembling)
- Blended Ensemble

These models are compared and the most informative features to determine whether a particular Twitter account is a bot or a legitimate user are discussed. A discussion is also given which aims to answer the following question:

 **Can we identify the ratio of bots to legitimate users using a subset of tweets about Trump?**

![jpg](/img/bot.jpg)

## Summary

In summary, bot detection is not a trivial task and it is difficult to evaluate the performance of our bot detection algorithms due to the lack of certainty about real users scraped from the Twitter API. Even humans cannot achieve perfect accuracy in bot detection. That said, the models we developed over the course of this project were able to detect bots with a high accuracy. They even had comparable performance to well-developed models from industry trained on datasets ten times the size of ours and with more than 1000 features. Adding NLP-related features to our models significantly improved their accuracy on our test set. However, we were most effective at bot detection on an unseen data set using only user-based features. With more time, we would train our NLP models on a larger diversity of bots and users to allow for stronger fingerprinting of bots versus legitimate users.