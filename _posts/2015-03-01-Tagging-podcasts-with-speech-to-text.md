---
layout: post
title: Tagging podcasts with speech-to-text
---
<!--<img src="/images/fulls/01.jpg" class="fit image">-->
This project aims to investigate tagging of podcasts by using a speech-to-text algorithm and extracting keywords from the text file.

## Introduction

Since I have always been interested in music and audio I wanted to do a small project involving this. The most useful I could think off, was to try to rip text from a podcast and find the most used words. These could then be attached as "tags/keywords" (meta data), making it possible to search on these in a database of podcasts.

The podcast used in this project is from a show called Clockwise. The title was: "Glue an iPad to a Speaker, Done!" and is an interview/conversation style podcast with two or more speakers. Due to limited computing power I had to use a smaller snipped of the full audio file.

For this example we will use the following libraries:

```python
import dataiku
import matplotlib.pyplot as plt
from IPython.display import Audio
import speech_recognition as sr
import pocketsphinx
from wordcloud import WordCloud, STOPWORDS
import collections
from rake_nltk import Rake
import operator
```

As a side note, I am using Dataiku to organise the project. I like to use Dataiku as my main platform for all kinds of data projects. By making a "managed folder" Dataiku can handle any kind of data type and you can access these files any way you like with a python recipe.

![png](/images/podcast-tags/flow.png)

## Setting up the file path

When attaching a python recipe to a managed folder, Dataiku automatically loads information about the folder to a dictionary in the begining of the code:

You can then extract the path like so:

```python
folder_path = audio_info['path'] + '/'
filename = 'podcast.wav'
file_path = folder_path + filename
```

Using the IPython.display.Audio library you can easily play audio from inside the notebook:

```python
Audio(data=file_path)
```
![png](/images/podcast-tags/player.png)

## Ripping text from audio file

The speech recognition library simply records the audio to memory and uses the PocketSphinx, which runs a recognition algorithm locally on your computer.

```python
r = sr.Recognizer()

with sr.AudioFile(file_path) as source:
    
    # listen for the data (load audio to memory)
    audio_data = r.record(source)
    
    # recognize (convert from speech to text)
    text = r.recognize_sphinx(audio_data)
```
The text looks like this:

![png](/images/podcast-tags/text.png)

From listening to the audio and reading the ripped output, it is clear that the recognition of the words is not impressive. For faster and better performance it is possible to use multiple online ML tools by Google and Microsoft eg., if you have a license to use their API. However, to show the concept the PocketSphinx should do fine.

## Creating wordsclouds from text

Now that we have the podcast in text format let us try to filter out the most common stopwords and make a wordcloud:

```python
cloud = WordCloud(stopwords=STOPWORDS).generate(text)
plt.figure(figsize=(8, 8), facecolor=None)
plt.imshow(cloud, interpolation="bilinear")
plt.axis("off")
plt.tight_layout(pad=0)
plt.show()
```
![png](/images/podcast-tags/wc1.png)

The most common words are bigger in size and seems to be one, apple, back, way, remote etc. From the title of the podcast "Apple" and "remote" possible makes good sense. Even though we have filtered out many stopwords most of the other words seem quite generic.

We can simply update the stopwords list if we want to remove more words:

```python
stopwords_new = list(STOPWORDS) + ['back', 'clockwise']
```

And the wordcloud now looks like this:

![png](/images/podcast-tags/wc2.png)

To get the list of top word let us try and find the most used ones in another way:

```python
filtered_words = [word for word in text.split() if word not in stopwords_new]
counted_words = collections.Counter(filtered_words)

words = []
counts = []
for letter, count in counted_words.most_common(10):
    words.append(letter)
    counts.append(count)
```

```python
list(zip(words, counts))
```

![png](/images/podcast-tags/score1.png)

As we can see it corresponds well with the wordcloud above.

These 10 most common words would not be very good "tags" for this podcast unfortunately. One thing is of course that it is only a snippet of the whole podcast and the Sphinx is not the best interpreter. However, a lot of custom tweaking has to be done to filter out the most relevant tags, and for this to work as automated as possible. Most likely specific tweaks has to be done per podcast series you are trying to tag.

## Getting keywords with Rake

Another library designed to extract keywords from text is called Rake. This algorithm tries to cluster words that are often used together while also ranking them on the amount of times they are used.

```python
rake = Rake(stopwords=stopwords_new)
tags = rake.extract_keywords_from_text(text)
top_tags = rake.get_ranked_phrases_with_scores()
```

```python
top_tags[:10]
```

![png](/images/podcast-tags/score2.png)

This result is obviously not the most useful. Since the text is interpreted from spoken word, and that there were multiple people speaking, this algorithm might have some difficulties working properly. With a more advanced setup, it could be interesting to test if it is possibe to recognise the different speakers from eachother and see if that could help the algorithm by looking at their sentences seperately.