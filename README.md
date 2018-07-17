# Insight Project: GameOn - Quickly evaluate board games based on user reviews
Created during my time at Insight (Summer 2018)


## The Problem I Wanted to Solve

#### Reviews in General
In the age of e-commerce, consumer product reviews are an increasingly important tool for online consumers. When buying a product online, consumers are not able to see and/or try the product like they might be able to do if they bought it in person at a brick-and-mortar store. Instead, they often look to the opinions and experiences of other consumers who have bought and tried the product. However, sometimes it can be difficult to find relevant information within user reviews. In spite of the importance of reviews in informing purchasing decisions, many websites only offer minimal tools to aid consumers in navigating through user reviews. I wanted to create a tool that would help consumers find the information they’re looking for more efficiently. 

#### BoardGameGeek.com
Board games have been increasing in popularity recently. The industry as a whole ($1.4 billion market in US and Canada) is growing rapidly, with an increase in sales by 28% in 2017 alone. Fivethirtyeight.com has described the current period as a “board game renaissance”, with an ever increasing number of games being released into the market by independent distributors. BoardGameGeek.com is the premier online hub for board game enthusiasts – where they can discover new (and not so new) games. They offer an abundance of information about over 990k games, which can at times be overwhelming. 

Additionally, they provide a platform for users to rate and review the board games. However, like many websites, the user reviews on BoardGameGeek.com are difficult to navigate, especially if you’re looking for specific information. After talking to some board game consumers, it seemed like they only dive into the user reviews once already having learned more general information about the game first. They use the user reviews to find out how others felt about specific topics relevant to their interests and intents. For example, they may want to know specifically whether a game facilitates player interaction, or whether it’s appropriate to play with kids. 

#### Scoping the Problem and Getting Feedback from Board Game Consumers
Before and throughout the process of building and improving upon my project, I spoke to a number of board game consumers to make sure I was creating a tool that would be helpful and useful to them. 

Originally, I had wanted to provide consumers with a list of the most frequent topics that were being discussed in reviews for a particular game, and show how the reviewers felt about each topic. I initially used a couple of different approaches (LDA and TF-IDF) to help me extract common topics/themes discussed in reviews. However, the topics that were formed using these bottom-up approaches were at times too general, and also either lumped different topics together under the same heading (LDA) and/or split semantically similar topics apart (LDA and TF-IDF). 

However, after showing a barebones prototype of the project to some board game consumers, I learned that a product that allowed them to specify the topic themselves would be most effective in meeting their needs. This would enable them to search for specific topics that are relevant to their personal interests (rather than being shown a broad range of topics). 

## GameOn Tool Allows Consumers to Search for User-Generated Topics
The tool I built is meant to help consumers find the information they’re looking for more efficiently. If this approach were to be implemented within the boardgamegeek.com website, it would allow consumers to select a board game, and search the reviews for a particular feature that is of interest to them. It then shows consumers the percentage of reviews related to this feature that are positive and negative. It also displays the top 5 most positive and negative reviews. Note that I did not build a front-end web application for a couple of reasons: 1.) I believe this tool would be most useful if integrated into the website rather than existing as a standalone application, and 2.) My goal was to help improve the boardgamegeek.com interface, and I did not want to pull foot traffic away from their site. 


## Building the GameOn Tool

#### Data
I obtained about 1.8 million reviews from over 2000 board games via the BoardGameGeek.com API. I elected to extract only the games with 1000+ reviews. I stored the data in a SQL database. Although the data I obtained was not necessary with the final version of the product that I built, I wanted to ensure that I had enough data to be able to pursue any number of routes. Additionally, a larger game and review database to pull from would allow me to test my product at a larger-scale level. 

#### Approach to Feature/Topic Extraction 
To find and extract reviews related to the consumer’s selected feature of interest, one option I considered was to perform a simple keyword search, and find reviews that contain the exact words entered into the search. However, because reviewers might describe the same feature in different ways, it was important for my algorithm to be able to group semantically similar reviews together in order to best serve the consumer’s needs. For this reason, I chose to use Word2Vec, because it allowed me to take semantics into account. 

#### “Pre-Gaming:” Text Cleaning
I cleaned the text of the reviews using fairly standard practices. I first made all the reviews lowercase. I then removed the (lowercase) names of board games from reviews (except for a subset of board game names, that are contained in common words; e.g., “Ra” and “Tak”). Because I would be taking an average of the word vectors within a review, I didn’t want the names of games to pull the average in the direction of the game name. I then parsed the reviews into separate words (i.e., tokenization). I subsequently removed “stop words” and the word signaling commonly used emoticons that are irrelevant (e.g., “star”). 

#### Word2Vec
Generally, Word2Vec is a technique that converts words into numerical vector representations. 
What’s cool about these models is that vector relationships can actually capture the semantic relationships between words. I used the pre-trained Google News Word2Vec model to generate a vector for each word. I spoke to a few NLP experts, who encouraged me to use a pre-trained model rather than training my own model. When reviews and features contained multiple words, I averaged across all the word vectors to create a single vector per review and search feature. 

#### Comparing User-Generated Feature with Reviews
I then estimated the cosine distance between the feature of interest and each review. Cosine distance essentially estimates differences in orientation between the feature vector and the review vectors. I used a pre-specified cut-off for the maximum distance, and extracted all reviews with cosine distances below that cut-off. 

#### Sentiment Analysis 
Then, for each extracted review, I estimated the overall sentiment polarity of that review using TextBlob (which ranged from -1 to 1). I then assigned that polarity estimate to a binary category of either negative or positive. I subsequently summarized this output by informing consumers what percentage of extracted reviews are positive, and what percentage are negative. 


## Validation: Feature Search
To validate the feature extraction method, I hand-labeled the features covered in 500 reviews from a single game (Catan - a classic game I have yet to play). I used the F1 score to evaluate model performance, which is the weighted average of precision and recall. I compared my “Review2Vec” search method (graphed in blue) to a simple Keyword Search (graphed in purple). On average across multiple features, Keyword Search performed better than the Review2Vec Search method.
However, there was a subset of features on which keyword search performed badly - this is where Review2Vec performed better. These features (like replayability) tended to be described in many different ways by reviewers. For example, reviewers might say, “replayability is high”, or “I’ve played this game a hundred times and I still love it.” Review2Vec was able to flag these descriptions as similar, whereas Keyword Search could not. These results suggest that when it comes to the feature extraction method, an ensemble approach that combines both of these different search methods would be best moving forward. 


## Summary
The GameOn tool was built to help users navigate more efficiently through reviews so they can more quickly find the information they need to make purchasing decision. Applying an ensemble approach that combines both "Review2Vec" with keyword search would be most effective as a search strategy, allowing Review2Vec to make up for when keyword search fails. 

Although for my project I focused on the board game industry, they are not the only ones that could benefit from this tool. This same approach could be applied to essentially any website that hosts a platform where consumers can rate and review products online. I think it would behoove any website to focus on how they can improve the customer's experience. 


## Next Steps (if I had more time!)
If I were to spend more time on this project (i.e., more than the allotted three weeks within the Insight program), I would first take steps to refine and improve my Review2Vec model. I would perform feature extraction and sentiment analysis on a per sentence basis (instead of treating the reviews as a whole). I would also train my own Word2Vec model on the review data (instead of using the pre-trained model on Google News data) to see if that improved my model’s performance. I would then implement the ensemble approach that combines both keyword search and Review2Vec). 

Additionally, I would spend some more time on the sentiment analysis. I would validate its performance, and try to improve upon it as well.  
