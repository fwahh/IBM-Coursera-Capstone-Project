# Exploring Singapore Neighborhoods

## Introduction
With Covid-19 catalysing adoption of remote work, an even larger proportion of our time is spent at home compared to pre-Covid times. In Singapore, most young adults of ages between 25~35 will be preoccupied with getting an apartment of their own. Apartment buying, has several factors involved, numero uno being price. However price aside, location is also an important consideration, especially for remote workers who will be having lunches, evening jogs, coffee breaks around their residential area.

In this project, I'll be exploring the city state, Singapore, and attempting to perform a clustering of neighborhoods, in hopes that this information may be useful for any young adults looking to buy an apartment in future. At the very least, I know it may be some good reference for me.

## Data sources
To tackle the area of interest, we'll be gathering the following data:
* Singapore Neighborhoods boundaries: In particular, we'll be utilizing the boundaries as described in 2019 Masterplan found in data.gov.sg <sup>1</sup>
* Singapore resident population and age distribution: We'll be utilizing another data set from Data.gov.sg<sup>2</sup>, which will inform us on the resident population in each neighborhood by age group and gender, as of 2015. We'll be extracting 2 parts of the data. The first will be used as a proxy for how many people are living in a neighborhood, and the second will be used to help us estimate the average age of residents in a neighborhood. According to Singapore Department of Statistics, Singapore Residents refer to Singapore citizens and permanent residents. So unfortunately, this dataset does not have information on non-residents, a group which approximately accounts for 30% of the total population<sup>3</sup>. To be more specific, this dataset is actually a composite of 4 datasets in the following order:
  <ol>
  <li>Total Resident Count in each subzone</li>
  <li>Total Resident Count in each subzone, with a further breakdown on age group</li>
  <li>Total Resident Count in each subzone, with a further breakdown on sex</li>
  <li>Total Resident Count in each subzone, witha further breakdown on age group and sex</li>
  </ol>
* Venues in neighborhoods: We won't be able to perform any analysis without knowledge of the places. Fret not though, we won't be exploring by foot but instead, will be gathering these information with the help of Foursquare Places API <sup>4</sup>. No doubt, this method of gathering information will be much more thorough and comprehensive than looking up a guidebook on Singapore.

## Methodology
During exploratory data analysis, we found out that there's 332 subzones designated by the government. Of the 332, 214 would qualify as residential areas. The remaining are ones which are either sparsely populated (populations < 200), or serve as idustrial areas, business parks, offshore islands. Furthermore, armed with the population of residents across age group, we estimate the average age in each neighborhood. 

![image](https://user-images.githubusercontent.com/65491089/123555900-14a2c580-d7bb-11eb-91e8-609432876ba0.png)

With this, we utilize Folium to present 2 possible representations of residential areas in Singapore. Choropleth map according to population size: 

![image](https://user-images.githubusercontent.com/65491089/123555918-3308c100-d7bb-11eb-9070-183da13a4372.png)

Choropleth map according to estimated average age: 

![image](https://user-images.githubusercontent.com/65491089/123555925-3d2abf80-d7bb-11eb-960c-b5fe355a481f.png)

Subsequently, with the help of Foursquare Places API, we gathered the venues as follows 

![image](https://user-images.githubusercontent.com/65491089/123555943-53388000-d7bb-11eb-91d3-cee3a342bf94.png)

Seems like Singapore truly is a bustling city state, we had capped the venues per neighborhood to be 100, and have gotten a total of 12629 venues across 214 neighborhoods. Among the venues, there are 353 unique categories. Next we move on to implementing K-means clustering. K-means clustering does not accept categorical values. As such, we'll use one-hot encoding on categories, turning it into 353 features corresponding to each unique category. Subsequently, we group venues by neighborhood since our clustering is meant to be done on each neighborhood, reducing the values to be the frequency of occurence of each category. A value of 0.2 for a category would mean that of the venes within the neighborhood, 20% are of said category.
We move on to use these venues in helping us to cluster Singapore neighborhoods by employing K-means algorithm. For K-means, the hyperparameter on number of cluster needs to be defined first. As such, we run the algorithm for no. of clusters ranging from 1 to 30, and plot the sum of squared errors for each k clusters.

![image](https://user-images.githubusercontent.com/65491089/123555950-5fbcd880-d7bb-11eb-8e6e-343f0bfc135d.png)

Though we hope to employ the elbow method in helping to select the optimal number of clusters, from the above graph, it is actually pretty hard to tell definitively where the elbow point is but we'll select 8 as the number of clusters.

## Results
Setting no. of clusters as 8, we arrive at the following cluster distribution.

![image](https://user-images.githubusercontent.com/65491089/123555963-6e0af480-d7bb-11eb-85b8-acdab4dd184c.png)

A glance at cluster 1:
![image](https://user-images.githubusercontent.com/65491089/123555976-7c591080-d7bb-11eb-8897-a8b0f7df5bd1.png)

Cluster 7, a foodie haven?
![image](https://user-images.githubusercontent.com/65491089/123555873-f210ac80-d7ba-11eb-954d-4ffcb20dc099.png)

Analysing the top 10 most common venues of each neighborhood by cluster. The following seems to be the main characteristics of clusters:<br>
Cluster 0: A neighborhood defined by accessibility, where bus station and facilities rank prominently as the most common.<br>
Cluster 1: A more eclectic mix of venues that might be popular with tourists and/or expats, with common venues ranging from hotels, restaurants of various cuisines and cafes. this happens to be the second largest cluster.<br>
Cluster 2: This cluster seems similar to cluster 7 at first glance where its top spots are occuped by food court and coffee shops. However, the venues have a wider variety and are also filled with non-food places.<br>
Cluster 3: This cluster seems to just be defined by the Loyang neighborhood.<br>
Cluster 4: Cluster with the biggest size. Chinese, asian restaurants being the most common venue.<br>
Cluster 5: Neighborhoods with food court being the most common venue.<br>
Cluster 6: Neighborhoods with nature trail being the most common venue.<br>
Cluster 7: Neighborhoods where the most common venues are a mix-up between coffeeshop, foodcourt, fastfood restaurant and chinese restaurants. In fact, even more than the other clusters, most of the top 10 spots seem to be occupied by something food related and in contrast to other clusters, ethnic ones like Chinese, Malay, Indian restaurants feature more prominently in this cluster.<br>

## Discussion
While unsupervised learning algorithms such as K-means clustering work great in helping us recognize patterns we might not have thought of, it is hard to validate their models, because there's no response variable indicating if we've classified it correctly or wrongly. As such, it is similarly difficult for hyperparameter tuning. In this project, I've chosen the simple elbow method to help choose hyperparameters. There are other proposed ways of evaluating K-means models<sup>5</sup>.

Our methodology on data extraction could also use some refining. In this study, we use 1 coordinate for each neighborhood to search for venues within 1km of said coordinates. However from the maps above, we can easily see that the neighborhoods do not all have the same size. For small neighborhoods, the current approach could suffice but for large neighborhoods, the venues contained within would end up being underrepresented. A better approach could be generating coordinates at regular intervals and using Shapely<sup>6</sup> to check which neighborhood the generated coordinates lie in. With this approach, larger neighborhoods would have more starting coordinates, hence generating results more representative of said neighborhood.

For a more comprehensive overview, we could also consider including price as an additional dimension in future analysis. This does not necessarily have to be the price of houses, but could be the price of venues, such as how costly the venues are, in particular for dining establishments. However, more isn't necessarily good. In this project, we ended up with over 300 features which made interpretation hard. We could consider some dimension reducing methods in future.

## Conclusion
Singapore truly is a food haven, where a majority of the clusters are defined by food. Though after performing clustering, patterns emerge. Some smaller-sized clusters are closer to nature, others have a more eclectic mix of venues. We hope that this offers an alternate view on choosing a suitable place of residence by leveraging on data.

### References
1. https://data.gov.sg/dataset/master-plan-2019-subzone-boundary-no-sea?resource_id=84b62d90-c1b7-4ada-acfc-f5874b5fd945
2. https://data.gov.sg/dataset/resident-population-by-planning-area-subzone-age-group-and-sex-2015?view_id=8cbe66d9-2235-436c-82cd-3e7574a741ad&resource_id=c33dd6dd-330f-4a1c-9280-efe8a67ff073
3. https://www.singstat.gov.sg/find-data/search-by-theme/population/population-and-population-structure/latest-data
4. https://developer.foursquare.com/docs/places-api/
5. https://medium.com/@ODSC/assessment-metrics-for-clustering-algorithms-4a902e00d92d
6. https://autogis-site.readthedocs.io/en/latest/notebooks/L3/02_point-in-polygon.html

