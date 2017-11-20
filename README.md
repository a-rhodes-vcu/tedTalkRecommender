# Ted Talk Recommender

This repo contains Ipython/Jupyter notebooks for basic exploration of transcripts of Ted Talks using Natural Language Processing (NLP), topic modeling, and a recommender that lets you enter key words from the title of a talk and finds 5 talks that are similar.
    The data consists of transcripts from Ted and TedX talks. Thanks to the lovely Rounak Banik and his web scraping I was able to dowload transcripts from 2467 Ted and TedX talks from 355 different Ted events. I downloaded this corpus from Kaggle, along with metadata about every talk. I encourage you to go to kaggle and download it so that he can get credit for his scraping rather than put it in this repo.
    https://www.kaggle.com/rounakbanik/ted-talks
    

The initial cleaning and exploration are done in 
   
    ## ted_clean_explore.nb

   Start by importing the csv files and looking at the raw data. Combine the metadata and transcripts and save as 'ted_all' (both datasets have a url column, so we merge on that one). Create a variable that holds only the transcripts called 'talks'. Below is a sample of the transcript from the most popular (highest views) Ted Talk. 'Do Schools Kill Creativity? by Sir Ken Robinson. 

    Good morning. How are you?(Laughter)It\\'s been great, hasn\\'t it? I\\'ve been blown away by the whole thing. In fact, I\\'m leaving.(Laughter)There have been three themes running through the conference which are relevant to what I want to talk about. One is the extraordinary evidence of human creativity

The first thing I saw when looking at these transcripts was that there were a lot of parentheticals for various non-speech sounds. For example, (Laughter) or (applause) or (Music).  There were even some cute little notes when the lyrics of a performance were transcribed
    
    someone like him ♫♫ He was tall and strong,
 
   I decided that I wanted to look at only the words that the speaker said, and remove these words in parentheses. Although, it would be interesting to collect these non-speech events and keep a count in the main matrix, especially for things like 'laughter' or applause or multimedia (present/not present) in making recommendations or calculating the popularity of a talk.
   
    Lucky for me, all of the parentheses contained these non-speech sounds and any of the speaker's words that required parenthesis were in brackets, so I just removed them with a simple regular expression. Thank you, Ted transcribers, for making my life a little easier!!!
    
    clean_parens = re.sub(r'\\([^)]*\\)', ' ', text)
    
# Cleaning Text with NLTK\n",
Four important steps for cleaning the text and getting it into a format that we can analyze:\n",
1.tokenize\n",
2.lemmatize\n",
3.remove stop words/punctuation\n",
4.vectorize\n",
    "\n",
    "[NLTK (Natural Language ToolKit)][1] is a python library for NLP. I found it very easy to use and highly effective.\n",
    "\n",
    "[1]: http://www.nltk.org/\n",
    "\n",
    "* **tokenize**- This is the process of splitting up the document (talk) into words. There are a few tokenizers in NLTK, and one called **wordpunct** was my favorite because it separated the punctuation as well.\n",
    "    \n",
    "    ```\n",
    "    from nltk.tokenize import wordpunct_tokenize\n",
    "    doc_words2 = [wordpunct_tokenize(docs[fileid]) for fileid in fileids]\n",
    "    print('\\n-----\\n'.join(wordpunct_tokenize(docs[1])))\n",
    "    \n",
    "    OUTPUT:\n",
    "    Good\n",
    "    morning\n",
    "    .\n",
    "    How\n",
    "    are\n",
    "    you\n",
    "    ?\n",
    "    ```\n",
    "    \n",
    "The notes were easy to remove by adding them to my stop words. Stopwords are the words that don't give us much information, (i.e., the, and, it, she, as) along with the punctuation. We want to remove these from our text, too. \n",
    "\n",
    "* We can do this by importing NLTKs list of **stopwords** and then adding to it. I added a lot of words and little things that weren't getting picked up, but this is a sample of my list. I went through many iterations of cleaning in order to figure out which words to add to my stopwords. \n",
    "\n",
    "```\n",
    "    from nltk.corpus import stopwords\n",
    "    stop = stopwords.words('english')\n",
    "    stop += ['.',\" \\'\", 'ok','okay','yeah','ya','stuff','?']\n",
    "```\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "2.  **Lemmatization** - In this step, we get each word down to its root form. I chose the lemmatizer over the stemmer because it was more conservative and was able to change the ending to the appropriate one (i.e. children-->child, capacities-->capacity). This was at the expense of missing a few obvious ones (starting, unpredictability).\n",
    "```\n",
    "    from nltk.stem import WordNetLemmatizer\n",
    "    lemmizer = WordNetLemmatizer()\n",
    "    clean_words = []\n",
    "    \n",
    "    for word in docwords2:\n",
    "    \n",
    "        #remove stop words\n",
    "        if word.lower() not in stop:\n",
    "            low_word = lemmizer.lemmatize(word)\n",
    "    \n",
    "            #another shot at removing stopwords\n",
    "            if low_word.lower() not in stop:\n",
    "                clean_words.append(low_word.lower())\n",
    "```\n",
    "\n",
    "Now we have squeaky clean text! Here's the same excerpt that I showed you at the top of the README.\n",
    "\n",
    "```\n",
    "good morning great blown away whole thing fact leaving three theme running conference relevant want talk one extraordinary evidence human creativity\n",
    "```\n",
    "As you can see it no longer makes a ton of sense, but it will still be very informative once we process these words over the whole corpus of talks.\n",
    "\n",
    "Let's look at some of the n-grams. This is just pairs of words (or tuples) that show up together. It will tell us something about our corpus, but also guide us in our next step of vectorization. Here's what we get for the top few most fequent bi-grmas.\n",
    "\n",
    "```\n",
    "from collections import Counter\n",
    "    from operator import itemgetter\n",
    "    \n",
    "    counter = Counter()\n",
    "    \n",
    "    n = 2\n",
    "    for doc in cleaned_talks:\n",
    "        words = TextBlob(doc).words\n",
    "        bigrams = ngrams(words, n)\n",
    "        counter += Counter(bigrams)\n",
    "    \n",
    "    for phrase, count in counter.most_common(30):\n",
    "        print('%20s %i' % (\" \".join(phrase), count))\n",
    "\n",
    "OUPUT:\n",
    "year ago 2074\n",
    "little bit 1607\n",
    "year old 1365\n",
    "united states 1103\n",
    "one thing 1041\n",
    "around world 938\n",
    "new york 894\n",
    "can not 877\n",
    "first time 751\n",
    "\n",
    "```\n",
    "The tri-grams were not very informative or useful aside from \"new york city\" and \"0000 year ago\" which get picked up in the bi-grams. \n",
    "```\n",
    "OUPTUT:\n",
    "new york city 236\n",
    "000 year ago 135\n",
    "new york times 123\n",
    "10 year ago 118\n",
    "every single day 109\n",
    "million year ago 109\n",
    "people around world 101\n",
    "```\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "* **Vectorization** is the important step of turning our words into numbers. The method that gave me the best results was count vectorizer. This function takes each word in each document and counts the number of times the word appears. You end up with each word as your columns and each row is a document (talk), so the data is the frequency of each word in each document. As you can imagine, there will be a large number of zeros in this matrix; we call this a sparse matrix. \n",
    "\n",
    "large number of zeros in this matrix; we call this a sparse matrix. \n",
    "\n",
    "```\n",
    "c_vectorizer = CountVectorizer(ngram_range=(1,3), \n",
    "                             stop_words='english', \n",
    "                             max_df = 0.6, \n",
    "                             max_features=10000)\n",
    "\n",
    "# call `fit` to build the vocabulary\n",
    "c_vectorizer.fit(cleaned_talks)\n",
    "\n",
    "# finally, call `transform` to convert text to a bag of words\n",
    "c_x = c_vectorizer.transform(cleaned_talks)\n",
    "```\n",
    "\n",
    "Now we are ready for topic modeling!\n",
    "Open:\n",
    "\n",
    "topic_modeling_ted_1.nb\n",
    "\n",
    "First get the cleaned_talks from the previous step. Then import the models\n",
    "\n",
    "```\n",
    "from sklearn.decomposition import LatentDirichletAllocation,  TruncatedSVD, NMF\n",
    "```\n",
    "\n",
    "We will try each of these models and tune the hyperparameters to see which one gives us the best topics (ones that make sense to you). It's an art.\n",
    "\n",
    "This is the main format of calling the model, but I put it into a function along with the vectorizers so that I could easily manipulate the paremeters like 'number of topics, number of iterations (max_iter),n-gram size (ngram_min,ngram_max), number of features (max_df):  \n",
    "```\n",
    "lda = LatentDirichletAllocation(n_components=topics,\n",
    "                                    max_iter=iters,\n",
    "                                    random_state=42,\n",
    "                                    learning_method='online',\n",
    "                                    n_jobs=-1)\n",
    "    \n",
    "lda_dat = lda.fit_transform(vect_data)\n",
    "```\n",
    "The functions will print the topics and the most frequent 20 words in each topic. \n",
    "\n",
    "The best parameter to tweak is the number of topics, higher is more narrow, but I decided to stay with a moderate number (20) because I didn't want the recommender to be too specific in the recommendations. \n",
    "\n",
    "Once we get the topics that look good, we can do some clustering to improve it further. However, as you can see, these topics are already pretty good, so we will just assign the topic with the highest score to each document. \n",
    "```\n",
    "topic_ind = np.argmax(lda_data, axis=1)\n",
    "topic_ind.shape\n",
    "y=topic_ind\n",
    "```\n",
    "\n",
    "Then, you have to decide what to name each topic. Do this and save it for plotting purposes in topic_names. Remember that LDA works by putting all the noise into one topic, so there should be a 'junk' topic that makes no sense. I realize that as you look at my code, you will see that I have not named a 'junk' topic here.  The closest was the 'family' topic but  I still felt like it could be named.  Usually, when running the models with a higher number of topics (25 or more) you would see one that was clearly junk.\n",
    "\n",
    "``` \n",
    "    topic_names = tsne_labels\n",
    "    topic_names[topic_names==0] = \"family\"  \n",
    "    . . .\n",
    "```\n",
    "\n",
    "Then we can use some visualization tools to 'see' what our clusters look like. The pyLDAviz is really fun, but only plots the first 2 components, so it isn't exactly that informative. I like looking at the topics using this tool, though. Note: you can only use it on LDA models.\n",
    "\n",
    "The best way to 'see' the clusters, is to do another dimensionality reduction and plot them in a new (3D) space. This is called tSNE (t-Distributed Stochastic Neighbor Embedding. When you view the tSNE ,it is important to remember that the distance between clusters isn't relevant, just the clumpiness of the clusters. For example, do the points that are red clump together or are they really spread out? If they are all spread out, then that topic is probably not very cohesive (the documents in there may not be very similar).  \n",
    "\n",
    "After the tSNE plot, you will find the functions to run the other models (NMF, Truncated SVD). \n",
    "\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Recommender\n",
    "\n",
    "\n",
    "Recommender_ted.nb\n",
    "\n",
    "Load the entire data set, and all the results from the LDA model. \n",
    "The function will take in a talk (enter the ID number) and find the 10 closest talks using nearest neighbors.  \n",
    "\n",
    "The distance, topic name, url, and ted's tags for the talk will print for the talk you enter and each recommendation. "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Flask App\n",
    "\n",
    "There is an even better verion of the recommender in the form of a flask app. This app can also 'find' your talk even if you don't remember the title.\n",
    "\n",
    "ted_rec.html \n",
    "\n",
    "ted_app.py \n",
    "\n",
    "You enter the keywords, or words from the title. \n",
    "Then, it returns your talk's title and url along with 5 similar ted talks (urls) that are similar to yours.  "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.5.4"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}
