# Project Overview

### Objective

Create a clean, reproducible analysis of four friends' Spotify listening histories and produce a combined top-100 "OGs" playlist that captures the tracks and artists most listened to across the group. The goal was to transform raw Spotify `StreamingHistory` JSON files into summarized metrics (listening time and play counts), visualize listening habits per person, and produce a shareable CSV/JSON of the group's top tracks.

### Context

This is a personal exploratory data project built from exported Spotify `StreamingHistory` files for four people (Vico, Peter, Immanuel and Nedum). It demonstrates skills in data ingestion, time parsing, aggregation, visualization, and export a practical example of cleaning real application logs and deriving actionable artifacts (a playlist) from them.

### **Duration**

The project took 2 to 3 weeks, encompassing data collection from Spotify and analysis.

### **Role**

I collected the exported Spotify JSON files, cleaned and transformed them, performed exploratory analysis for each person, combined the datasets, and produced a group top-100 playlist. I worked on the project alone.

### **Tools and Methodologies**

Data analysis was conducted using Python for statistical computations and visualizaion. The methodologies included correlation analysis, linear regression, and cluster analysis to identify patterns and draw insights.

- **Language & environment:** Python, Jupyter Notebook
- **Libraries:** `pandas`, `numpy`, `matplotlib`, `seaborn` (for plotting), `IPython.display` for formatted outputs
- **Methods:** data ingestion, cleaning, datetime parsing, timedelta conversions, aggregation (groupby), sorting, visualization (bar charts, side-by-side comparisons), and export (CSV/JSON).

# **The Approach and Process**

### **Data Collection**

- The source files were Spotify `StreamingHistory` JSON exports (e.g. `VStreamingHistory0.json`, `PStreamingHistory0.json`, `NStreamingHistory0.json`, `IStreamingHistory0.json`).
- Each file corresponds to one person’s streaming history and contains fields such as `endTime`, `artistName`, `trackName`, and `msPlayed`.

### Data Preparation & Cleaning

Key steps taken for each person's file:

1. **Read the JSON file into pandas**
2. **Convert `endTime` to a pandas `datetime` and create `Play_Time`**
3. Derive temporal features (year, month, day, weekday, hour) for time-based analysis
4. **Convert `msPlayed` to a human-friendly timedelta and extract hours/minutes**
5. Drop unneeded columns after transformation
    
    Some users had more than one streaming history files so it had to be read and joined together. And each step was repeated for each user.
    
    **Why these steps?** They normalize the timing information and listening duration so we can fairly compare play counts and true listening time across tracks and users.
    
    ```python
    vic = pd.read_json('VStreamingHistory0.json')
    vic2 = pd.read_json('VStreamingHistory1.json')
    
    ## joining the 2 dataframes together
    vic_stream = pd.concat([vic,vic2], ignore_index = True)
    
    ## converting endTime to datetime
    vic_stream['Play_Time'] = pd.to_datetime(vic_stream['endTime'])
    
    ## converting msPlayed to extract hours and minutes
    vic_stream['Time-Played (hh-mm-ss)'] = pd.to_timedelta(vic_stream['msPlayed'], unit='ms')
    
    # helper functions
    def hours(td):
    return td.seconds / 3600
    
    def minutes(td):
    return (td.seconds / 60) % 60
    
    vic_stream['Listening Time(Hours)'] = vic_stream['Time-Played (hh-mm-ss)'].apply(hours)
    vic_stream['Listening Time(Minutes)'] = vic_stream['Time-Played (hh-mm-ss)'].apply(minutes)
    
    ##derive year,month day weekday and hour
    vic_stream['year'] = pd.DatetimeIndex(vic_stream['Play_Time']).year
    vic_stream['month'] = pd.DatetimeIndex(vic_stream['Play_Time']).month
    vic_stream['weekday'] = pd.DatetimeIndex(vic_stream['Play_Time']).weekday
    vic_stream['hour'] = pd.DatetimeIndex(vic_stream['Play_Time']).hour
    
    #dropping unneeded columns
    vic_stream.drop(columns=['endTime', 'msPlayed', 'Time-Played (hh-mm-ss)'], inplace=True)
    ```
    

### Aggregation & Metrics

For per-user and group analyses we computed:

- **Play count per track**: `groupby(['trackName'])['trackName'].count()` or grouping on `['artistName', 'trackName']` and taking `count()`.
- **Total listening time per track**: group and `sum()` the `Listening Time(Minutes)` or `Listening Time(Hours)` columns.
- **Top-k lists**: head(10) for charts and head(100) when producing the combined playlist.

Example to make an individual top-10 by hours and by count:

```python
vic_top_artist_count = vic_stream.groupby(['artistName'])[['Listening Time(Hours)', 'Listening Time(Minutes)', 'count']].sum().sort_values(by = 'count', ascending = False)
vic_top_artist_count.head(10)
```

### Combining the Group and Producing the "OGs" Playlist

- Concatenate all cleaned streaming DataFrames into one `mix_stream`
- Group by `['artistName', 'trackName']` and compute play `count()` (or sum of listening time) then sort to get the top 100 tracks across the whole group
- Output the playlist to CSV/JSON for sharing/upload

```python
## joining cleaned dataframes together
mix_stream = pd.concat([vic_stream, pet_stream, ned_stream, imm_stream], ignore_index=True)

##grouping artistname and trackname by playtime
mixed_playlist = mix_stream.groupby(['artistName', 'trackName'])[['count']].count().sort_values(by = 'count', ascending = False).head(100)
```

### **Challenges and Solutions**

Inconsistencies with the amount of data each user had based on how long they have been using Spotify. It was solved by finding the users with the smallest amount and cutting the rest to match it so everything can be the same.

### **Visualizations and Insights**

- **Per-user top artists:** bar charts revealing personal favorites artist based on hours listened vs number of times listened
- **Per-user top tracks (count):** shows what people replay often vs how long they listen to the songs.
- **Group top-100 :** a combined ranked list that can be exported and, optionally, converted to a Spotify playlist.

![Screenshot 2024-03-04 235641.png](attachment:b1c6cbff-44dd-40cd-a4ca-7e2b257fd7f1:Screenshot_2024-03-04_235641.png)

![image.png](attachment:a417b081-8a46-4e9e-833b-8df5c3d2a056:image.png)

![Screenshot 2025-10-10 051754.jpg](attachment:1bcf9f78-8502-4c04-97f0-a37159110d07:Screenshot_2025-10-10_051754.jpg)

# **End Results and Recommendations**

**Deliverable:** `OGs.csv` and `OGs.json` ; the group’s top-100 tracks by play count (and optional listening time metric), plus per-user top-10 summary charts.

### **Reflections**

This project is a strong example of practical data wrangling it starts with messy real-world exports and ends with a tangible deliverable (a collaborative playlist). It demonstrates attention to data hygiene (time parsing and duration conversion), clear aggregation logic, and an output that non-technical users can consume.

### **Next Steps**

- **Incorporate Qualitative Data:** Integrate passenger feedback for a holistic service assessment.
- **Monitor Implementation:** Track changes and measure success using key performance indicators.
- **Iterative Improvement:** Refine services based on feedback and emerging needs.

### **Conclusion**

This notebook is a concise, well-scoped example for a portfolio: it shows ETL skills, exploratory analysis, teamwork-oriented deliverables (the group "OGs" playlist), and clear next steps for improving the product.
