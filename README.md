# Simple Sentiment Analysis for Tweets

**This project is only for learning purpose. It doesn't have any intentions to choose the side. This analysis is based on the recent 4k tweets collected on July 10,2020. The sentiments might be different if data of any other date or time will be used**

## Introduction
As elections in US is upon us. I thought to analyse the sentiments people are showing for both the Presidential canditates on twitter.  
This is the python code to analyse the tweets contains keyword Donald Trump or/and Joe Biden. It will show how many of the tweets in which they are mentioned have positive and negative polarity.



## Fetching Tweets
To fetch the data from tweet, you need to create a developers account and then you need to create an app. Then you will get the values for the variables access_token, access_token_secret, api_key and api_secret_key. For detailed information, refer this https://medium.com/@jayeshsrivastava470/how-to-extract-tweets-from-twitter-in-python-47dd07f4e8e7

Tweepy is used to fetch the data from the Twitter. Below is the code to connect to your twitter API.

```python
import tweepy
auth = tweepy.OAuthHandler(api_key,api_secret_key)
auth.set_access_token(access_token,access_token_secret)
api = tweepy.API(auth)
```

## Streaming Tweets
To get the latest 2000 tweets for Trump and Biden respectively, streamListener is required. 
```python
from tweepy import Stream
from tweepy import StreamListener
```
Below is the code to stream the tweets
```python
tweet_stream = Stream(auth, Listener())
tweet_stream.filter(track=['Trump', 'Biden'])
  ```
## Custom Exception
As this is a mini project, I want to fetch around 2k tweets. To implement this condition, I write my custom Exceptions.
```python
class Error(Exception):
  """Base class for other exceptions"""
  pass
class SufficientTweetsCollected(Error):
  """Raised when the sufficent tweets are collected"""
  pass
```

## Processing the Tweets
We are getting the stream of the tweets. To handle that we need to create a Listener class where we will write our logic to pre-process the tweets adn save the analysis into the file.
We need to override two methods of Listener class - on_data() and on_error()
### on_data() function
This function is responsible to fetch the tweets one by one. There are lots of data in a tweet like datetime, id, location, tweet text, likes, retweets etc. As we only interested in text. So we will extract that data. Each tweet will be pre-processed. Converted into Blob which contains all the sentences. Then sentiment polarity of each sentence is calculated.

**Fetching Tweet text**
```python
raw_tweets = json.loads(data)
tweets = raw_tweets['text']
```

**Pre-processing**

We need to remove all the mentions( starts with @username ), all the links , RT.
```python
tweets = ' '.join(re.sub("(@[A-Za-z0-9]+)|([^0-9A-Za-z \t])|(\w+:\/\/\s+)"," ",tweets).split())
tweets = ' '.join(re.sub('RT',' ', tweets).split())
```

**sentiment analysis**

TextBlob is used to do sentiment analysis. Here is the link for its [documentaion](https://textblob.readthedocs.io/en/dev/)

Below code will give a numeric number which will tell how positive or negative the sentences in the tweets are?
```python
blob = TextBlob(tweets.strip())
for sentence in blob.sentences:
  print(sentence.sentiment.polarity)
```

**Complete Code explaination of the method on_data()**

The code is in try and catch block to handle three types of exception primarily( AttributeError,KeyError,NameError) which is thrown while fetching tweets.
First, code will check if the tweet is related to Donald Trump or Joe Biden. Then, the sentiment of each tweet is calculated by adding their polarity score and store in variable (trump_sentiment/biden_sentiment). That value will be added to the global variable(trump/biden) which will have the consolidated polarity score of the tweets.
We are saving the polarity score in the csv file. The data of the csv file will be used to plot graph.

Also, we are counting tweets for each canditate. When both the canditates has atleast 2k tweets then code will throw an exception and the streaming will stop.
```python
  def on_data(self, data):
    raw_tweets = json.loads(data)
    
    try:
      tweets = raw_tweets['text']
      tweets = ' '.join(re.sub("(@[A-Za-z0-9]+)|([^0-9A-Za-z \t])|(\w+:\/\/\s+)"," ",tweets).split())
      tweets = ' '.join(re.sub('RT',' ', tweets).split())
      blob = TextBlob(tweets.strip())
      trump_sentiment = 0
      biden_sentiment =0
      global trump
      global biden
      global cnt_trump_tweets
      global cnt_biden_tweets
      for sentence in blob.sentences:
        if ("Trump" in sentence) and ("Biden" not in sentence):
          print("Trump:",sentence.sentiment.polarity)
          trump_sentiment = trump_sentiment + sentence.sentiment.polarity
          cnt_trump_tweets=cnt_trump_tweets+1
          print("******************************",cnt_trump_tweets)
        else:
          print("Biden:",sentence.sentiment.polarity)
          biden_sentiment = biden_sentiment + sentence.sentiment.polarity
          cnt_biden_tweets=cnt_biden_tweets+1
          print("******************************",cnt_biden_tweets)

      trump = trump + trump_sentiment
      biden  = biden +  biden_sentiment

      with open('sentiment.csv',mode='a') as file:
        writer = csv.DictWriter(file,fieldnames=['Trump','Biden'])
        info={
            'Trump': trump,
            'Biden': biden
        }
        writer.writerow(info)
      
      # this code is responsible to stop streaming og the tweets.
      if (cnt_biden_tweets > 2000) and (cnt_trump_tweets > 2000):
        raise Exception(SufficientTweetsCollected)
      # printing tweets on console  
      print(tweets)
    except (AttributeError,KeyError,NameError) as e :
      print('Attribute Error')      
   ```

## Plotting graph
The code is for realtime plotting of the graphs. If we keep on fetching tweets in background and adding new data in the csv file then the graph will keep on changing. 
```python
import pandas as pd
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

plt.style.use('fivethirtyeight')

frame_len =10000
fig = plt.figure(figsize=(9,6))

def animate(i):
  data = pd.read_csv('/content/sentiment.csv')
  print("Total Trump Tweets:", cnt_trump_tweets,"\n","Total Biden Tweets:", cnt_biden_tweets )
  y1  = data['Trump']
  y2 = data['Biden']

  if len(y1)<=frame_len:
    plt.cla()
    plt.plot(y1,label='Donald Trump')
    plt.plot(y2, label='Joe Biden')
  else:
    plt.cla()
    plt.plot(y1[-frame_len:],label='Donald Trump')
    plt.plot(y2[-frame_len:],label='Joe biden')   
  
  plt.legend(loc='upper right')
  plt.tight_layout()
ani = FuncAnimation(plt.gcf(),animate,interval=100)

```
Here the final plot
![Sentiment Analysis](https://github.com/shilpisirohi12/Simple-Sentiment-Analysis-for-Tweets/blob/master/plot.PNG)

This shows that though in realtime, there are less tweets related to Joe Biden posted on twitter. But they are more positive.

Please refer the complete notebook for the code.
