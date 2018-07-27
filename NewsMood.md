
# News Mood Challenge

The objective of this challenge is to pull the 100 most recent tweets of seveal news sources and run VADER sentiment analysis on each. Compound scores are then visualized individually and as a whole to see what meaning can be derived from these scores.

### Analysis

Visualization of each individual sentiment score saw a small concentration of neutral scores upon running, but no other distinct trends emerge. Considering that these analyses are run on news sources, one might wonder what this implies about journalistic objectivity. Considering that objectivity is a central tennant of journalism ethics, one would hope to see a such a concentration. However, contents of the news itself would likely influence these scores as well.

When viewed as a whole, however, a more distinct trend emerges: partisan bias. All news sources except for Fox News scored negatively when viewed as a whole. Fox News' right wing preference is well known. Considering that these analyses were run on a slow news day during a Republican administration, it might not be suprising that their overall sentiment would score positively with more objective (or in the case of CNN, distinctly left wing) news sources scoring differently.

That being said, it would be interesting to run these analyses on a 'big news day' or compare to a slow news day during a Democratic administration. 


```python
#import necessary modules and info
import tweepy
import numpy as np
import pandas as pd
from datetime import datetime
import matplotlib.pyplot as plt
from matplotlib import style
style.use('ggplot')

from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()

from config import (consumer_key, 
                    consumer_secret, 
                    access_token, 
                    access_token_secret)
```


```python
#authenticate tweepy
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())
```


```python
#compile list of twitter handles through which to loop
handles = ["@BBCNews", "@CBSNews", "@CNN", "@FoxNews", "@nytimes"]

#create a list to hold dictionaries of the sentiment analysis resules
s_scores = []

#create a variable to store the oldest tweet id for the loop
ot = None
```


```python
#loop through the news organizations
for handle in handles:
    
    #create counter for tabulating tweets
    c = 1
    
    #loop through 5 pages (20 tweets per page) of tweets to acquire 100 tweets
    for x in range(5):
        tweets = api.user_timeline(handle, max_id = ot)
    
        #loop through the tweets themselves
        for t in tweets:
            
            #run Vader analysis on the tweet
            scores = analyzer.polarity_scores(t["text"])
            
            #add dictionary of scores to the list along with the media source date stamp
            s_scores.append({"Media Source": t["user"]["name"],
                            "Date": t["created_at"],
                           "Text": t["text"],
                           "Compound": scores["compound"],
                           "Positive": scores["pos"],
                           "Negative": scores["neu"],
                           "Neutral": scores["neg"],
                           "Tweets Ago": c})
            
            # store the tweet id in the oldest tweet variable and subtract 1 to continue iteration
            ot = t['id'] - 1
            
            #increase the counter to document the next tweet
            c += 1            
```


```python
#convert the list of dictionaries containing the scores into a DataFrame
spd = pd.DataFrame(s_scores)
spd = spd[["Media Source", "Date", "Text", "Tweets Ago", "Compound", "Negative", "Neutral", "Positive"]]
spd.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Media Source</th>
      <th>Date</th>
      <th>Text</th>
      <th>Tweets Ago</th>
      <th>Compound</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>Positive</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>BBC News (UK)</td>
      <td>Fri Jul 27 17:40:56 +0000 2018</td>
      <td>Actor Ed Westwick will not be prosecuted over ...</td>
      <td>1</td>
      <td>-0.3851</td>
      <td>0.667</td>
      <td>0.211</td>
      <td>0.121</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BBC News (UK)</td>
      <td>Fri Jul 27 17:30:20 +0000 2018</td>
      <td>Orlando Bloom twice halts Killer Joe 'over iPa...</td>
      <td>2</td>
      <td>-0.6486</td>
      <td>0.650</td>
      <td>0.350</td>
      <td>0.000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BBC News (UK)</td>
      <td>Fri Jul 27 16:56:08 +0000 2018</td>
      <td>Mamma Mia! Film soundtracks have taken over th...</td>
      <td>3</td>
      <td>-0.3595</td>
      <td>0.815</td>
      <td>0.185</td>
      <td>0.000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BBC News (UK)</td>
      <td>Fri Jul 27 16:53:00 +0000 2018</td>
      <td>RT @BBCPanorama: Stockton-on-Tees is the town ...</td>
      <td>4</td>
      <td>-0.7906</td>
      <td>0.731</td>
      <td>0.269</td>
      <td>0.000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BBC News (UK)</td>
      <td>Fri Jul 27 16:34:03 +0000 2018</td>
      <td>📵 Get ready for Scroll Free September 📵\n\nhtt...</td>
      <td>5</td>
      <td>0.7003</td>
      <td>0.463</td>
      <td>0.000</td>
      <td>0.537</td>
    </tr>
  </tbody>
</table>
</div>




```python
#export the DataFrame to a csv
spd.to_csv("100tweets_by_org.csv")
```


```python
#add color coding into the the DataFrame by news organization
spd.loc[spd['Media Source'].str.contains('BBC'), 'source_key'] = 'gold'
spd.loc[spd['Media Source'].str.contains('CNN'), 'source_key'] = 'steelblue'
spd.loc[spd['Media Source'].str.contains('CBS'), 'source_key'] = 'seagreen'
spd.loc[spd['Media Source'].str.contains('Fox'), 'source_key'] = 'firebrick'
spd.loc[spd['Media Source'].str.contains('York'), 'source_key'] = 'slateblue'
```

### Individual Tweet Review


```python
#get a datestamp in order to label the data
datestamp = datetime.now().strftime("%m/%d/%Y %H:%M")

#set plot size for easy reading
plt.figure(figsize = (15, 10))

#iterate through color mapping to create a subplot per news source
for i, dff in spd.groupby('source_key'):
    plt.scatter(dff['Tweets Ago'], dff['Compound'], s=70, c=i, edgecolors='grey', alpha=0.6, label=dff['Media Source'].unique()[0])

#set legend below the axis
plt.legend(loc='upper center', bbox_to_anchor=(0.5, -0.07),
          fancybox=True, shadow=True, ncol=5)

#label the data
plt.title(f"Sentiment Analysis of 100 Most Recent Media Tweets as of {datestamp}")
plt.xlabel("Tweets Ago")
plt.ylabel("Tweet Polarity Score")

#save figure
plt.savefig('100_tweets_sentiments.png')
# show plot
plt.show()
```


![png](output_12_0.png)


### Overall Tweet Review


```python
#group the sentiment dataframe by media source with color coding to get an average compound score
mean_sentiment = pd.DataFrame(spd.groupby(['Media Source', 'source_key'])['Compound'].mean()).reset_index(level='source_key')
mean_sentiment
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>source_key</th>
      <th>Compound</th>
    </tr>
    <tr>
      <th>Media Source</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BBC News (UK)</th>
      <td>gold</td>
      <td>-0.099121</td>
    </tr>
    <tr>
      <th>CBS News</th>
      <td>seagreen</td>
      <td>-0.048516</td>
    </tr>
    <tr>
      <th>CNN</th>
      <td>steelblue</td>
      <td>-0.010696</td>
    </tr>
    <tr>
      <th>Fox News</th>
      <td>firebrick</td>
      <td>0.033352</td>
    </tr>
    <tr>
      <th>The New York Times</th>
      <td>slateblue</td>
      <td>-0.021146</td>
    </tr>
  </tbody>
</table>
</div>




```python
#set plot size for easy reading
plt.figure(figsize = (15, 10))

#plot the mean compound scores to review an overall sentiment in the last hour
o_plot = mean_sentiment['Compound'].plot(kind="bar", color=mean_sentiment['source_key'], edgecolor='grey', alpha=0.6)

#set aesthetics and titles
plt.title(f'Overall Sentiment of 100 Most Recent Media Tweets as of {datestamp}')
plt.xlabel('Media Source')
plt.ylabel('Tweet Polarity Score')

# Use a for loop to add column value labels
totals = []
for c in o_plot.patches:
    #capture the value
    y_val = c.get_height()
    #determine the position the label needs to be
    if y_val < 0:
        y_pos = y_val + -0.004
    else:
        y_pos = y_val + 0.004
    #write the text
    o_plot.text(c.get_x()+.15, y_pos, str(round(y_val,3)), fontsize=10, color='black')

# Save the Figure
plt.savefig('overall_tweet_sentiments.png')

# Show the Figure
plt.show()
```


![png](output_15_0.png)

