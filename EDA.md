---
nav_include: 3
title: Exploratory Data Analysis
notebook: EDA.ipynb
---

## Contents
{:.no_toc}
*  
{: toc}

## Generate Training and Test 

Before we look at the data we first generate a training and test set which will be used later for model evaluation. The data is normalized such that it is easier to compare feature variables and because some of the models require normalized data, especially when regularization is used.

```python
bot_df = pd.read_csv(r"bot_df_final.csv",index_col='User ID')
user_df = pd.read_csv(r"user_df_final.csv",index_col='User ID')

bot_df['bot']=1
user_df['bot']=0

total_df = bot_df.append(user_df)

train_data, test_data = train_test_split(total_df, test_size = 0.3, random_state=99)

Y_train=train_data['bot']
Y_test=test_data['bot']
X_train=train_data.drop('bot',axis=1)
X_test=test_data.drop('bot',axis=1)

def normalize(df,df_train):
    result = df.copy()
    for feature_name in df_train.columns:
        max_value = df_train[feature_name].max()
        min_value = df_train[feature_name].min()
        result[feature_name] = (df[feature_name] - min_value) / (max_value - min_value)
    return result

X_train_scaled=normalize(X_train,X_train)
X_test_scaled=normalize(X_test,X_train)
```

## Preliminary Model Discovery

The first model that was developed as a way to trial the data used only two variables: friend count and follower count. These two variables, when shown in a log format for both legitimate users and bots, have a relatively clear boundary between them. These two features were used as a baseline model to compare other models too.

![png](img/Boundary.png){: .center}


## User Feature Statisitcs

In this section we look at several different plots to study our features and how they are different for bots and legitimate users.

### Feature Histograms

First we compared the distributions of the user features for bots and human users. 

```python
features = ['Screen name length', 'Number of digits in screen name', 'User name length', 'Account age (days)', 'Number of unique profile descriptions','Default picture (binary)','Number of friends','Number of followers','Number of favorites','Number of tweets per hour', 'Number of tweets total','timing_tweet']
fig, axes = plt.subplots(len(features),1, figsize = (10,25))
for i in range(len(features)):
    sns.kdeplot(user_df[features[i]], ax = axes[i], label = 'user')
    sns.kdeplot(bot_df[features[i]], ax = axes[i], label = 'bots')
    axes[i].set_xlabel(features[i])
    axes[i].legend()

plt.tight_layout()
plt.show()
```


![png](EDA_files/EDA_6_0.png){: .center}

Although we expected some features to differ, some of the actual distributions might not be easily distinguished between bots and human users, such as number of digits in the screen name and number of unique descriptions. Some of them are easier to differentiate, such as number of friends, number of followers, number of favorites and number of tweets, indicating that those features might play an important role in the bot detection.

Next, let's repeat the process for our natural language and text-based features.

### Natural Language Feature Histograms

#### First, let's look at the sentiment and subjectivity features:


```python
features = ['overall_sentiment', 'overall_polarity', 'var_sentiment', 'var_polarity']
fig, axes = plt.subplots(len(features),1, figsize = (10,10))
for i in range(len(features)):
    sns.kdeplot(legit_df[features[i]], ax = axes[i], label = 'user')
    sns.kdeplot(bots_df [features[i]], ax = axes[i], label = 'bots')
    axes[i].set_xlabel(features[i])
    axes[i].legend()

plt.tight_layout()
plt.show()
```

![png](EDA_files/EDA_9_0.png){: .center}

From these smoothed histograms, we can see that bots score high on average across all metrics of sentiment and subjectivity, but follow a similar distribution.


#### Text style based features


```python
features = ['percent_with_emoji', 'percent_hashtag','avg_num_caps']
fig, axes = plt.subplots(len(features),1, figsize = (10,8))
for i in range(len(features)):
    sns.kdeplot(legit_df[features[i]], ax = axes[i], label = 'user')
    sns.kdeplot(bots_df [features[i]], ax = axes[i], label = 'bots')
    axes[i].set_xlabel(features[i])
    axes[i].legend()

plt.tight_layout()
plt.show()
```


![png](EDA_files/EDA_10_0.png){: .center}

Bots have a much wider spread for emoji use-- some bots post emojis in almost every post, and others don't post at all. Humans are more consistent in their emoji use. The opposite appears to be true for percent of tweets with hashtags and average number of capital letters: humans have a broader distribution, while bots are clustered around 10% of tweets with hashtags and ~1 capital letter per tweet. 


#### Tweeting style (retweets, mentions) based features:


```python
features = ['percent_mention', 'avg_num_mentions',
            'avg_time_between_mention',  
            'avg_time_between_rt','percent_tweet_rt']
fig, axes = plt.subplots(len(features),1, figsize = (10,14))
for i in range(len(features)):
    sns.kdeplot(legit_df[features[i]], ax = axes[i], label = 'user')
    sns.kdeplot(bots_df [features[i]], ax = axes[i], label = 'bots')
    axes[i].set_xlabel(features[i])
    axes[i].legend()

plt.tight_layout()
plt.show()

```


![png](EDA_files/EDA_11_0.png){: .center}

Again, we see very different distributions for humans and bots. Bots have an interesting bimodal distribution for percent of mentions, while humans have a much smoother distribution. Average number of mentions follows a similar pattern.There is a large spread on time between mention for both bots and humans, but bots mention more quickly than humans on average. Finally, the retweet patterns are very different between bots and humans. Notably, most bots either never retweet, or retweet all the time. Humans have a smoother distribution. Note that the spikes in usuage often refer to the imputed mean values. 


#### Natural Language-based features


```python
features = ['avg_word_len','word_diversity' ,'difficult_words_score', 
            'avg_words_per_tweet','avg_readability_combined_metric',  
            'avg_flesch_reading_ease', 'avg_readability_DC']
fig, axes = plt.subplots(len(features),1, figsize = (10,20))
for i in range(len(features)):
    sns.kdeplot(legit_df[features[i]], ax = axes[i], label = 'user')
    sns.kdeplot(bots_df [features[i]], ax = axes[i], label = 'bots')
    axes[i].set_xlabel(features[i])
    axes[i].legend()

plt.tight_layout()
plt.show()

```


![png](EDA_files/EDA_12_0.png){: .center}

Finally, when we compare the NLP features, we see that users tend to use longer words. Bots have a bimodal word diversity score-- some bots have low diversity and others high, while human users have less defined bimodal distribution. Interestingly, bots also have a bimodal difficult words score, with a lower average difficult word score than humans. For average words per tweet, bots sometimes used a high number (40>), but usually used around 10-20 words. Humans also used around 10-20 words, but sometimes posted very short tweets as well (0-5 words). As with word metrics, readability was similarly bimodal for bots in particular. Interestingly, it seems as though there are some "smart" bots and "dumb" bots, e.g. some that score higher in "intelligence" type metrics than humans, while others score lower. Perhaps this reflects different types of bots, or the retweet patterns of different bots and their sources for content.

### Pearson Correlation Heatmap (User Features)

Correlation plots of the features tell us how related one feature is to the next. We would like there to be little correlation between features such that all the features are informative, meaning there are no redundant features being used in our predictions.

```python
colormap = plt.cm.RdBu
plt.figure(figsize=(14,12))
plt.title('Pearson Correlation of Features', y=1.05, size=15)

# Generate a mask for the upper triangle
corr = X_train_scaled.astype(float).corr()
mask = np.zeros_like(corr, dtype=np.bool)
mask[np.triu_indices_from(mask)] = True
with sns.axes_style("white"):
    ax = sns.heatmap(corr, mask=mask, vmax=.3, square=True)
```

![png](EDA_files/EDA_7_0.png){: .center}

The correlation plot tells us that most of the features are not correlated, which is what we like to see. There are a couple of features that have a correlation of approximately 0.3, this is higher than the rest of the features but is still relatively low and should not cause any issues. Thus, we can leave all of these features in our dataset.

### Pearson Correlation Heatmap (NPL and Text-Based Features)

```python
colormap = plt.cm.RdBu
plt.figure(figsize=(14,12))
plt.title('Pearson Correlation of NLP and Text-Based Features', y=1.05, size=15)

# Generate a mask for the upper triangle
corr = legit_df[['avg_word_len','word_diversity' ,'difficult_words_score', 
            'avg_words_per_tweet','avg_readability_combined_metric',  
            'avg_flesch_reading_ease', 'avg_readability_DC',
            'percent_mention', 'avg_num_mentions',
            'avg_time_between_mention',  
            'avg_time_between_rt','percent_tweet_rt',
            'percent_with_emoji', 'percent_hashtag','avg_num_caps',
            'overall_sentiment', 'overall_polarity', 'var_sentiment', 
            'var_polarity']].astype(float).corr()
mask = np.zeros_like(corr, dtype=np.bool)
mask[np.triu_indices_from(mask)] = True
with sns.axes_style("white"):
    ax = sns.heatmap(corr, mask=mask, vmax=.3, square=True)
```
The NLP correlation plot tells us that most features are uncorrelated. The only strongly correlated features are the readability features. This is because the readability metrics are related by word length, word diversity, and other similar measures. We include both readibility metrics because they provide extra information about the tweets, but we are cautious to interpret any coefficients or importance values of one vs. the other due to multicollinearity.

The word length is also correlated with a number of features, including number of words in a tweet, word diversity, difficult word score, etc. simply due to tweets being a fixed length. For example, if a user uses longer words, they cannot put as many words in a tweet. This correlation cannot be avoided.

![png](EDA_files/EDA_13_0.png){: .center}

### Pairplot

A pairplot allows us to get a visual plot of the relationship between different feature variables. We can also color code the plot to show which of the particular users are bots and which are not, so that we can visually see the boundaries between users and bots to aid us in the model development and feature engineering processes.

```python
pairplot_df = total_df.copy()
pairplot_df['Number of friends'] = np.log10(pairplot_df['Number of friends'])
pairplot_df['Number of followers'] = np.log10(pairplot_df['Number of followers'])
pairplot_df['Number of favorites'] = np.log10(pairplot_df['Number of favorites'])
pairplot_df['Number of tweets total'] = np.log10(pairplot_df['Number of tweets total'])

sns.set(style="ticks", color_codes=True)
g = sns.pairplot(pairplot_df, vars=[u'Account age (days)', 'Number of friends', u'Number of followers', u'Number of favorites', u'Number of tweets total', u'timing_tweet'],
                 hue='bot', palette = 'seismic',diag_kind = 'kde',diag_kws=dict(shade=True),plot_kws=dict(s=10))
#g.set(xticklabels=[])
```


![png](EDA_files/EDA_8_0.png){: .center}

We see from the pairplot that there are several features where there is a relatively clear boundary between bots and users, indicating that we should be able to obtain good predictions from our models when utilizing these features. However, none of these features are able to completely separate bots from legitimate users, and hence using many features will get us better predictions. 


