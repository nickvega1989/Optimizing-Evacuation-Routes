# Optimizing-Evacuation-Routes
### Introduction and Defining the problem
Our project endeavored to answer whether we can use real time data such as tweets about traffic or public safety conditions to optimize evacaution routes during na emergency. In addition, can this data be used to provide more details to people in need of information, such as emergency responders, beyond pinpointing the  location of accident or delay on Google Maps.

To solve this problem, our team chose to focus on tweets from the Houston, Texas region for the following reasons:
- Houston has notoriously bad traffic and gridlock
- There are several active transportation and public safety related Twitter accounts from which we can collect data.
- There have been historical disasters in Houston where gridlock had been a problem for emergency services and with hindsight, data science could have been employed to map routes to prevent gridlock.

For example, in 2005, Hurricane Rita had been documented as the strongest Gulf of Mexico hurricane in history with a direct track for the Houston area.

According to the Houston Chronicle, "an estimated 2.5 million people hit the road ahead of the storm’s arrival, creating some of the most insane gridlock in U.S. history. More than 100 evacuees died in the exodus. Drivers waited in traffic for 20-plus hours, and heat stroke impaired or killed dozens. Fights broke out on the highway. A bus carrying nursing home evacuees caught fire, and 24 died."

Our team wanted to approach this project to help emergency services identify gridlock or chokepoints at a given time in a road or highway network, so that emergency services can identify these both in real time and identify areas with historical chokepoints (e.g., a highway under construction or a road that is frequently congested)and optimize evacuation routes around these areas.

### Acquiring the data:

Our team decided to obtain real-time traffic information from Twitter to address this problem and selected tweets from the Houston, Texas region to train this model for the reasons described above.

Initially, we acquired a Twitter developer API and attempted to use this API to obtain tweets of interest.
There are time limitations for how far back tweets can be acquired (usually 30 days). To get around this problem, our team identified the 'Get Old Tweets3' library for Python. This library was an efficient resource for obtaining broader historical tweets from a range of accounts.

We obtained approximately 16,000 tweets from various public safety, transit and weather accounts with 'close' or 'closed' referenced in the data. This was achieved by specifying specific terms for the 'set query' argument in the Get Old Tweets library. Based on this, these tweets were classified as closure tweets 'closure' tweets and assigned to the negative class for modeling. An additional pull of tweets was conducted without a set query filter and were classied to the positive class.

The tweets were saved as a dataframe and we ensured that the number of generic tweets matched the number of "closure" tweets in the final dataframe used for modeling to prevent problems caused by unbalanced classes.

### Initial Natural Language Processing

Once the data was organized we were able to conduct NLP. We used both the Count and Tfidf Vectorizers. After conducting more EDA on the vectorized data we then included our own unique stop words to the default list of stop words. We stuck with the default tokenization settings of the vectorizers. We did not lemmatize or stem. We did not experiment much in this domain of NLP for reasons we discuss later.

### Initial Modeling

We began with basic Logistic Regression & Support Vector Machine models. Due to the strength of these default algorithms & default vectorizers and Nick's organized data scraping & cleaning,  we achieved high accuracy scores of about 95%-96.4%. However, considering the context of emergency personnel and evacuation routes we wanted to improve this score since every percentage point could possibly impact lives being saved or lost. Instead of customizing these models and vectorizers we decided to explore other options. This is where advanced NLP and modeling comes in.

### Named Entity Recognition:
Attempted to use the spaCy library to identify named entities in tweets with the idea being we could identify and visualize intersections where there are closures.
Ultimately, there were errors returned due to the underlying data that prevented us from training a model using spaCy.

An alternative approach was used to build a co-occurrence function to identify named entities in tweets. This function analyzes tweets to identify entity pairs that occur in the corpus. For each entity pair that occurs together in the same tweet, records the number of times they occur together in the whole corpus Ultimately, the function returns a ‘weight’ to named entity combinations which can be used to identify the relative importance of a location for mapping purposes

These can be visualized using the networkx library to identify the most frequently occurring named entities and plot intersections.

### Advanced NLP & Modeling with Google's BERT

To classify tweets as relevant or not to road closures we utilized Google's BERT.
BERT stands for Bidirectional Encoder Representations from Transformers. It analyzes a word based on the other words before and after it in the sentence/tweet. This provides context to language which other NLP techniques do not do. It is open sourced and already trained on a large text corpus (Wikipedia) but we can give it additional training that is specific to our task. We utilized Google's free Colab environment so we could access their GPU's since BERT is a computationally heavy. BERT provided an accuracy score of 99.8% and an MCC score of 99.6%.

### Mapping
Naturally we decided that the most efficient way to present information about road closures or certain incidents coming in from Twitter would be to visualize them directly on a map. We considered a couple different tools for this, including the Geopy python library, but ultimately chose to use Mapquest’s Developer API (yes, Mapquest still exists) combined with the Folium library for a few reasons.

First of all, it’s worth noting that the Geopy python library is an excellent tool that offers different Geocoding services. However, some of those services require payments for their APIs (like Google). More importantly for this project, some of them are incapable of mapping street intersections (Openstreetmap), and we had no shortage of street intersections from our tweets. Mapquest’s API is free, capable of mapping street intersections, and the documentation on its developer webpage is very straighforward and helpful. We used Mapquest’s API to gather latitude and longitude coordinates for some of the locations gathered from our tweets, and then we used Folium to plot those coordinates on a map.

The tricky part about using Mapquest - or any mapping service for that matter - is that certain locations like intersections must be entered in a very specific format. A single character or lack thereof can result in multiple spots appearing on the map or simply the wrong spot showing up. For instance, the intersection of South Main St. and MLK St. in Mapquest MUST be written as “S Main Street & MLK Street” followed by the city and state (writing out “street” in full versus abbreviating it to “St” isn’t as important). It will not map an intersection that uses the word “and” rather than an ampersand, and the “S” for “South” is also pertinent to mapping the correct location, even if the Main Street in this example does not cross MLK Street anywhere else in the city. We have written a code that converts a list of street intersections into the correct format necessary for Mapquest’s API to map.

The code then uses the Requests Library in python to gather the latitude and longitude coordinates from the .json provided by Mapquest’s API and put them into columns. From there, the Folium library plots these locations in the jupyter notebook. One of the great things about Folium is that the background is customizable. You can select a background more akin to Google Maps, Openstreetmap, etc. in addition to roads, terrains, satellite, etc. The markers shown on the Folium map in this project’s jupyter notebook all have an icon of a house on them. The markers can be customized to appear with different icons, as different shapes, and in different colors. If this Twitter-mapping concept were put into a real-world application, it could adequately show emergency personnel in real time where there is construction or road closures that aren’t yet shown in Google Maps or Waze.

### Ideas for future development ###
- Fine-Tune and integrate BERT to automatically answer a question such as, “Is this road closed/damaged?” in real time and recognize named entities.
- Optimize Named Entity Recognition to identify and map relative locations of road closures.
- Pass the aforementioned information through a more sophisticated function that can convert the locations of all car accidents, traffic congestion, road closures, etc. into a mapping API-friendly format and then visually map them in real time.
- Customize map icons, colors, and visual patterns to more easily communicate the type of incident
