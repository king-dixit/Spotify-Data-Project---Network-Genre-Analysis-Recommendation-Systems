# Spotify-Data-Project---Network-Genre-Analysis-Recommendation-Systems


Being lovers of music we chose the Spotify dataset on Kaggle to explore our interests in the context of data mining. We are interested in the features that make a song popular and are interested in exploring the relationships between genres. 
Some applications of this project are if we decide to produce music of our own, we have an idea of what features to maximize or minimize if we want to increase the chances of our song being popular.
We also wanted to find out how certain features of a song have changed through the years through different era’s of music. (instrumentalness , acousticness,valence etc)  We wanted to explore how many genres and subgenres of a song would be there and how we can use that to create a hierarchical classification.
Some challenges include a lot of preprocessing the dataset as well as potentially web scraping.



Recommendation Systems
We built  our recommendation system based on the genre we get and the distance from numerical features. The steps we followed are:
1.We first create 20 clusters to divide the songs into 20 main genres
2.We filter songs with the same genre cluster.
3.We choose the songs based on the distance and the number of songs depends on the user input within the clusters .
The reason we chose 20 clusters is for optimizing the performance as there are 181 genres , creating those many clusters and computing it would not be efficient enough and through domain knowledge there were 20-25 main genres in our dataset so we chose the value of 20 to be the number of clusters in our Kmeans



Genre Network Modeling
	We sought to construct a network of the valid genres and their inheritance.
  For instance, 'east coast hip hop' intuitively inherits from 'hip hop'. 
  By constructing this hierarchy, we could capitalize on the hierarchical nature of genres and construct a hierarchical classification algorithm. 


We assumed the following:
In the original dataset, if an artist was tagged with a certain genre, he/she would also be tagged with all valid parent genres. (so if an artist was tagged with 'classical piano' he/she would also be tagged with 'classical') 
There are a maximum possible 5 levels of inheritance, each corresponding to an n-gram.
A subgenre (such as 'east coast hip hop') can be considered to inherit from a parent genre iff the latter is a subset of the former. For instance, the words 'hip' and 'hop are found in 'east coast hip hop' so 'hip hop' is a possible valid parent (more specifically, grandparent) 
For a parent genre to be a valid candidate, they must have a normalized score/count >0. This normalized score was computed for a particular n-gram by subtracting the counts of all (n+1)-grams that fully contain the n-gram. This normalization penalizes an n-gram genre if it appears as part of more specific genres. For instance, ‘italian pop’ appears only once in the dataset, but since the number of subgenres of ‘italian pop’ (‘vintage italian pop’, ‘italian adult pop’, ‘classic italian pop’, etc.) far outnumber ‘italian pop’ itself, the algorithm gives ‘italian pop’ a negative score. This negative score can be interpreted as being not unique enough of a genre to allow inheritance down the tree. However, a negative score for a genre does not preclude it from inheriting from a parent genre - so long as it has at least one parent genre with a positive score. In other words, a negative score for 'italian pop' prevents subgenres to inherit from it; however, it will still allow 'italian pop' to inherit from a 'pop', which has a positive score (437).
If there are multiple candidates for a parent, we will choose the parent with the highest score. For instance, 'technical brutal death metal' could be inherited from either 'brutal death metal' (score of 15) or 'technical death metal' (score of 12); we will choose the one with the higher confidence of validity. Ties are broken alphabetically. In non-normalized tri-gram, (('technical', 'death', 'metal'), 14), and (('brutal', 'death', 'metal'), 16). (('technical', 'brutal', 'death', 'metal'), 1), (('technical', 'melodic', 'death', 'metal'), 1), with no 5-grams. If 'brutal melodic death metal' was a genre in the dataset (it isn't) and occurred >=4 times, then 'technical brutal death metal' would instead inherit from 'technical death metal'. 
If there are no immediate parents for a genre, we will follow steps 2-4) to find any valid grandparents ((n+2)grams) to inherit directly from; else, we will check for any valid great-grandparents ((n+3)grams), etc. This allows us to correctly classify 'east coast hip hop' as inheriting from 'hip hop'. 
Note that we will always prioritize a closer parent than a more distant one. For instance, in deciding whether or not to classify 'pop r & b' as inheriting from 'pop' or 'r & b' we will choose 'r & b' even if the score for 'pop' is higher. This allows us to determine more granular relationships between genres and their subgenres.



With this constructed hierarchy in mind, future steps would be to implement a hierarchical classification algorithm. Here is a simplified overview of how this would work in practice:
Use knn to predict among top layer (181 genres) 
If confidence genre predicted > threshold:
Recursively predict down the graph (step 1)
If confidence genre predicted < threshold:
‘Blocking’: do not predict further down the graph
To accomplish this, we need to build a family of ‘local classifiers per parent node’. We would train each local classifier separately. Here is an example of the algorithm working in practice:

Confidence threshold = 0.7
Top-level classifier classifies *NSYNC as ‘pop’ with confidence 0.81
0.81 > 0.7, so send artist parameters to ‘pop’ classifier
‘Pop’ classifier classifies artist as ‘dance pop’ with confidence 0.85
0.85 > 0.7 so send artist to ‘dance pop’ classifier
‘Dance pop’ classifier classifies artist as ‘deep dance pop’ with confidence 0.65
0.65 < 0.7, so end the algorithm here.
NSYNC classified as [‘pop’, ‘dance pop’]
