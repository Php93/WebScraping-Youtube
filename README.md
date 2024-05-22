# Youtube Data Web Scraping and Data Analysis

```python
import pandas as pd
from googleapiclient.discovery import build
import seaborn as sns
```


```python
api_key = "AIzaSyBUMGRzCNdHtELeiBiKUad_KT9WFKXlINM"
channel_id = "UCX6OQ3DkcsbYNE6H8uQQuVA"
channel_ids = [
    "UC7cs8q-gJRlGwj4A8OmCmXg", # Alex the analyst
    "UCsXQTrMEZkcD_UuCR0wMJmQ", # JacobSucksAtCode
    "UCCQ6SXMc7MoJ88jjpn6j-8Q", # Marko
]
```


```python
youtube = build("youtube", "v3", developerKey=api_key)
```

## Function to get channel statistics


```python
def get_channel_stats(youtube, channel_ids):
    result = []
    request = youtube.channels().list(
        part="snippet,contentDetails,statistics",
        id=",".join(channel_ids)
    )
    response = request.execute()

    for channel in response["items"]:
        data = {   
            "Channel_name": channel["snippet"]["title"],
            "Subscribers": channel["statistics"]["subscriberCount"],
            "Views": channel["statistics"]["viewCount"],
            "Total_videos": channel["statistics"]["videoCount"],
            "Playlist_id": channel["contentDetails"]["relatedPlaylists"]["uploads"]
        }
        result.append(data)

    return result
```


```python
channel_statistics = get_channel_stats(youtube, channel_ids)
df = pd.DataFrame(channel_statistics)
```


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Channel_name</th>
      <th>Subscribers</th>
      <th>Views</th>
      <th>Total_videos</th>
      <th>Playlist_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marko</td>
      <td>203000</td>
      <td>9229006</td>
      <td>67</td>
      <td>UUCQ6SXMc7MoJ88jjpn6j-8Q</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alex The Analyst</td>
      <td>776000</td>
      <td>34282669</td>
      <td>304</td>
      <td>UU7cs8q-gJRlGwj4A8OmCmXg</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jacob Sucks At Code</td>
      <td>35300</td>
      <td>1855260</td>
      <td>60</td>
      <td>UUsXQTrMEZkcD_UuCR0wMJmQ</td>
    </tr>
  </tbody>
</table>
</div>




```python
df["Subscribers"] = pd.to_numeric(df["Subscribers"])
df["Views"] = pd.to_numeric(df["Views"])
df["Total_videos"] = pd.to_numeric(df["Total_videos"])
```


```python
df.dtypes
```




    Channel_name    object
    Subscribers      int64
    Views            int64
    Total_videos     int64
    Playlist_id     object
    dtype: object




```python
sns.set(rc={"figure.figsize": (10, 8)})
ax = sns.barplot(x="Channel_name", y="Subscribers", data=df)
```


    
![png](main_files/main_9_0.png)
    



```python
ax = sns.barplot(x="Channel_name", y="Views", data=df)
```


    
![png](main_files/main_10_0.png)
    



```python
ax = sns.barplot(x="Channel_name", y="Total_videos", data=df)
```


    
![png](main_files/main_11_0.png)
    


## Function to get video ids


```python
def get_video_ids(youtube, playlist_id):
    video_ids = []
    
    request = youtube.playlistItems().list(
        part="contentDetails",
        playlistId=playlist_id,
        maxResults=50,
    )
    
    response = request.execute()
    next_page_token = response.get("nextPageToken")

    for video in response["items"]:
        video_ids.append(video["contentDetails"]["videoId"])

    while True:
        if next_page_token is None:
            break
        else:
            request = youtube.playlistItems().list(
                part="contentDetails",
                playlistId=playlist_id,
                maxResults=50,
                pageToken=next_page_token
            )
            
            response = request.execute()
            for video in response["items"]:
                video_ids.append(video["contentDetails"]["videoId"])
            next_page_token = response.get("nextPageToken")

    return video_ids
```


```python
video_ids = get_video_ids(youtube, "UU7cs8q-gJRlGwj4A8OmCmXg")
```

## Function to get video details


```python
def get_video_details(youtube, video_ids):
    all_videos = []
    
    for i in range(0, len(video_ids), 50):
        request = youtube.videos().list(
            part="snippet,statistics",
            id=",".join(video_ids[i:i+50])
        )

        response = request.execute()

        for video in response["items"]:
            video_stats = {
                "Title": video["snippet"]["title"],
                "Published_date": video["snippet"]["publishedAt"],
                "Views": video["statistics"]["viewCount"],
                "Likes": video["statistics"]["likeCount"],
                "Comments": video["statistics"]["commentCount"],
            }
            all_videos.append(video_stats)

    return all_videos
```


```python
video_details = get_video_details(youtube, video_ids)
```


```python
video_df = pd.DataFrame(video_details)
```


```python
video_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Title</th>
      <th>Published_date</th>
      <th>Views</th>
      <th>Likes</th>
      <th>Comments</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Top 5 Most Popular Functions In Microsoft Excel</td>
      <td>2024-05-21T12:00:53Z</td>
      <td>4602</td>
      <td>203</td>
      <td>13</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Infrastructure of Analyst Builder | Entire Tec...</td>
      <td>2024-05-14T12:00:28Z</td>
      <td>2474</td>
      <td>59</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2 Hour Data Analyst Interview Masterclass | In...</td>
      <td>2024-05-07T15:16:53Z</td>
      <td>29055</td>
      <td>1262</td>
      <td>36</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Challenges of Creating Analyst Builder | Chall...</td>
      <td>2024-05-07T12:00:30Z</td>
      <td>1909</td>
      <td>55</td>
      <td>11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>How We Started Analyst Builder | How We Create...</td>
      <td>2024-04-30T12:00:07Z</td>
      <td>5137</td>
      <td>191</td>
      <td>31</td>
    </tr>
  </tbody>
</table>
</div>




```python
video_df[["Views", "Likes", "Comments"]] = video_df[["Views", "Likes", "Comments"]].apply(pd.to_numeric)
```


```python
video_df["Published_date"] = pd.to_datetime(video_df["Published_date"])
```


```python
video_df.dtypes
```




    Title                          object
    Published_date    datetime64[ns, UTC]
    Views                           int64
    Likes                           int64
    Comments                        int64
    dtype: object



## Top 20 most viewed videos


```python
top20 = video_df.sort_values(by="Views", ascending=False).head(10)
```


```python
ax1 = sns.barplot(x="Views", y="Title", data=top20)
```


    
![png](main_files/main_25_0.png)
    


## Videos by month

Adding Month column


```python
video_df["Month"] = video_df["Published_date"].dt.strftime("%b")
```


```python
video_df["Year"] = video_df["Published_date"].dt.year
video_df["Year"] = video_df["Year"].astype("str")
```


```python
video_df["Month"].value_counts()
```




    Month
    Jan    34
    Oct    30
    Mar    29
    Aug    28
    May    26
    Feb    26
    Apr    25
    Dec    24
    Nov    23
    Sep    23
    Jul    18
    Jun    18
    Name: count, dtype: int64




```python
videos_per_month = video_df.groupby(["Year", "Month"], as_index=False).size()
videos_per_month
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Month</th>
      <th>size</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020</td>
      <td>Apr</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020</td>
      <td>Aug</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020</td>
      <td>Dec</td>
      <td>9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020</td>
      <td>Feb</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020</td>
      <td>Jan</td>
      <td>5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2020</td>
      <td>Jul</td>
      <td>4</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2020</td>
      <td>Jun</td>
      <td>4</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2020</td>
      <td>Mar</td>
      <td>7</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2020</td>
      <td>May</td>
      <td>5</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2020</td>
      <td>Nov</td>
      <td>8</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2020</td>
      <td>Oct</td>
      <td>9</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2020</td>
      <td>Sep</td>
      <td>6</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2021</td>
      <td>Apr</td>
      <td>4</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2021</td>
      <td>Aug</td>
      <td>5</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2021</td>
      <td>Dec</td>
      <td>4</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2021</td>
      <td>Feb</td>
      <td>5</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2021</td>
      <td>Jan</td>
      <td>7</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2021</td>
      <td>Jul</td>
      <td>2</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2021</td>
      <td>Jun</td>
      <td>5</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2021</td>
      <td>Mar</td>
      <td>7</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2021</td>
      <td>May</td>
      <td>4</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2021</td>
      <td>Nov</td>
      <td>3</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2021</td>
      <td>Oct</td>
      <td>3</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2021</td>
      <td>Sep</td>
      <td>3</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2022</td>
      <td>Apr</td>
      <td>5</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2022</td>
      <td>Aug</td>
      <td>6</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2022</td>
      <td>Dec</td>
      <td>5</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2022</td>
      <td>Feb</td>
      <td>4</td>
    </tr>
    <tr>
      <th>28</th>
      <td>2022</td>
      <td>Jan</td>
      <td>5</td>
    </tr>
    <tr>
      <th>29</th>
      <td>2022</td>
      <td>Jul</td>
      <td>5</td>
    </tr>
    <tr>
      <th>30</th>
      <td>2022</td>
      <td>Jun</td>
      <td>4</td>
    </tr>
    <tr>
      <th>31</th>
      <td>2022</td>
      <td>Mar</td>
      <td>4</td>
    </tr>
    <tr>
      <th>32</th>
      <td>2022</td>
      <td>May</td>
      <td>5</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2022</td>
      <td>Nov</td>
      <td>6</td>
    </tr>
    <tr>
      <th>34</th>
      <td>2022</td>
      <td>Oct</td>
      <td>8</td>
    </tr>
    <tr>
      <th>35</th>
      <td>2022</td>
      <td>Sep</td>
      <td>4</td>
    </tr>
    <tr>
      <th>36</th>
      <td>2023</td>
      <td>Apr</td>
      <td>6</td>
    </tr>
    <tr>
      <th>37</th>
      <td>2023</td>
      <td>Aug</td>
      <td>13</td>
    </tr>
    <tr>
      <th>38</th>
      <td>2023</td>
      <td>Dec</td>
      <td>6</td>
    </tr>
    <tr>
      <th>39</th>
      <td>2023</td>
      <td>Feb</td>
      <td>4</td>
    </tr>
    <tr>
      <th>40</th>
      <td>2023</td>
      <td>Jan</td>
      <td>7</td>
    </tr>
    <tr>
      <th>41</th>
      <td>2023</td>
      <td>Jul</td>
      <td>7</td>
    </tr>
    <tr>
      <th>42</th>
      <td>2023</td>
      <td>Jun</td>
      <td>5</td>
    </tr>
    <tr>
      <th>43</th>
      <td>2023</td>
      <td>Mar</td>
      <td>5</td>
    </tr>
    <tr>
      <th>44</th>
      <td>2023</td>
      <td>May</td>
      <td>8</td>
    </tr>
    <tr>
      <th>45</th>
      <td>2023</td>
      <td>Nov</td>
      <td>6</td>
    </tr>
    <tr>
      <th>46</th>
      <td>2023</td>
      <td>Oct</td>
      <td>10</td>
    </tr>
    <tr>
      <th>47</th>
      <td>2023</td>
      <td>Sep</td>
      <td>10</td>
    </tr>
    <tr>
      <th>48</th>
      <td>2024</td>
      <td>Apr</td>
      <td>6</td>
    </tr>
    <tr>
      <th>49</th>
      <td>2024</td>
      <td>Feb</td>
      <td>7</td>
    </tr>
    <tr>
      <th>50</th>
      <td>2024</td>
      <td>Jan</td>
      <td>10</td>
    </tr>
    <tr>
      <th>51</th>
      <td>2024</td>
      <td>Mar</td>
      <td>6</td>
    </tr>
    <tr>
      <th>52</th>
      <td>2024</td>
      <td>May</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
sort_order = [
    "Jan", 
    "Feb", 
    "Mar",
    "Apr",
    "May",
    "Jun",
    "Jul",
    "Aug",
    "Sep",
    "Oct",
    "Nov",
    "Dec",
]
```


```python
videos_per_month.index = pd.CategoricalIndex(
    videos_per_month["Month"], 
    categories=sort_order,
    ordered=True
)
```


```python
videos_per_month = videos_per_month.sort_index()
```


```python
ax2 = sns.barplot(
    x="Month", 
    y="size", 
    hue="Year", 
    data=videos_per_month,
    hue_order=["2024", "2023", "2022", "2021", "2020"]
)
```


    
![png](main_files/main_35_0.png)
    

