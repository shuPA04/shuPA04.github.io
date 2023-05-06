---
layout: post
title: "Pitch Positioning Predictability in Football"
subtitle: ""
background: '/img/posts/pitch-positioning/header.jpg'
---

## 1 &nbsp; &nbsp; Introduction

This small project have two purposes:

1. Develop a tactical performance indicator to quantify pitch positioning predictability of football players.
2. Compare the predictability values between two halves of a match.

To achieve the first objective, I analysed the tracking data of a football match and computed approximate entropy (ApEn) values for each player. I then used these values as a collective performance indicator that quantifies the pitch positioning predictability of each player.

To achieve the second objective,  I compared the obtained ApEn values of players in the first half of matches to those in the second half to investigate the potential effects of external factors on player's positioning.

### 1.2 &nbsp; &nbsp; Approximate Entropy

Approximate entropy (ApEn) is a technique used to quantify the amount of predictability in a time-series data without any prior knowledge on the data-generating distribution. A low value of ApEn indicates the time-series is predictable, a high value indicates otherwise. Previous studies have applied ApEn to the tracking data from a football match to quantify movement patterns of players in terms of pitch positioning predictability. Examples of such studies are following:


*  Sampaio & Maçãs (2011) investigated the effect of small-sided games on player's tactical understanding. The ApEn was applied to the distance between **team's centroid** and each player over the match. Lower ApEn in the post test indicated player's expertise in football increased due to the training sessions. 

* Memmert et al. (2017) applied ApEn to the distance between each player and **position-specific centroids** in a game between Bayern Munich and Barcelona. The part of  result showed Bayern Munich central midfielder was highly predictable in his positioning relative to his positional centroid, indicating his pivotal role between players. 

This project followed the approach of Memmert et al. (2017), which are following:

* Calculate the positional centroids for each position group (DEF, MID, and ATT) by averaging the players' locations every 100 milliseconds.
* Determine the distance between each player and their corresponding centroid at each 100-millisecond interval.
* Compute the Approximate Entropy (ApEn) using the distance values for each player, which indicates the predictability of the player's positioning.
* Categorize the ApEn values into three groups (high, medium, and low) based on the quantiles of the ApEn distribution to assess the level of predictability.

### 1.3 &nbsp; &nbsp; Positional Centroids vs Team Centroid

To clarify the approach of Memmert et al. (2017), two centroids are visualised below. 

![centroids on the pitch](\img\posts\pitch-positioning\centroids.png)

**Positional centroids** (**right**) are calculated by averaging locations of players who belong to the same position (e.g. defenders). One can also calculate another metric, **team centroid** (**left**) which is the average location of all players on the pitch. While both metrics offer valuable insights, Positional Centroids are generally considered more effective in capturing players' movement patterns (Memmert et al., 2017).

After calculating the positional centroids, we can determine the distance between each player and their corresponding centroid. This process yields a vector containing distance values for each player. By inputting this vector into a function that calculates ApEn, we obtain a scalar value representing the predictability of a player's positioning relative to their centroid.

## 2 &nbsp; &nbsp; Data

Data for this project is tracking data from a football match, stored in two excel files (1st & 2nd half). Each file contains the following features:

* *PlayerID*: A unique identifier for the player.
* *Timestamp*: Time elapsed since the start of the match in milliseconds.
* *X*: X-coordinate of the player's position on the pitch.
* *Y*: Y-coordinate of the player's position on the pitch.
* *Speed*: The speed of the player in meters per second.
* *Distance*: The distance the player has covered since the start of the match in meters.
* *Position*: The player's position on the pitch, such as defender or attacker.
* *Detailed_Position*: A more specific description of the player's position, such as right back or center forward.

Pandas ```read_excel``` method was used to read the excel files into a pandas DataFrame prior to data processing.

### 2.1 &nbsp; &nbsp; Data Processing

Although the data is in a clean tabular format, some preprocessing steps were necessary before conducting the exploratory analysis. The following are some of the crucial steps taken:

* Remove goal keepers as they do not contribute to the formations of positional centroids.

```python
idx_to_drop = df[df['PlayerID'].isin(['VitesseO19101', 'PSVO17179'])].index
df.drop(idx_to_drop, inplace=True)
```

* Handle missing values in *X* and *Y* columns. 

Since the data is recorded in milliseconds, preceding and next rows should contain similar coordinate values. Hence forward filling was used.

```python
df[['X','Y']] = df[['X','Y']].fillna(method= 'ffill')
```

* Calculate positional centroids

We first create a new DataFrame *df_centroids* by getting the average X and Y coordinates of players based on their position and team for each timestamp

```python
df_centroids = df.groupby(['Timestamp', 'Position', 'Team'], as_index=False).agg({'X': np.mean, 'Y': np.mean})
df_centroids.rename(columns = {'X':'Centroid_X', 'Y':'Centroid_Y'}, inplace = True)
```
We can now merge two DataFrames again.

```python
df_merged = df.merge(df_centroids, how='inner', on = ['Timestamp', 'Position', 'Team'])
```

* Calculate the distance between each player and their corresponding centroid at each timestamp.

This is the final step of processing, we calculate the euclidean distance between each player and their position centroid for each timestamp with ```np.linalg.norm```.

```python
df_merged['Distance_to_Centroid'] = np.linalg.norm(
    df_merged.loc[:, ['X', 'Y']].values - df_merged.loc[:, ['Centroid_X', 'Centroid_Y']], 
    axis=1)
```
&nbsp;

## 3 &nbsp; &nbsp; Result

### 3.1 &nbsp; &nbsp; By Positions


<style>
.table-container {
  display: flex;
  justify-content: center;
}
</style>

<div class="table-container">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: left;">
          <th>PlayerID</th>
          <th>Position</th>
          <th>Predictability 1st</th>
          <th>Predictability 2nd</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>PSVO19223</td>
          <td>ST</td>
          <td>Medium</td>
          <td>Low</td>
        </tr>
        <tr>
          <td>VitesseO19104</td>
          <td>ST</td>
          <td>Low</td>
          <td>Low</td>
        </tr>
        <tr>
          <td>VitesseO19119</td>
          <td>LWG</td>
          <td>High</td>
          <td>High</td>
        </tr>
        <tr>
          <td>PSVjong2</td>
          <td>LWG</td>
          <td>High</td>
          <td>High</td>
        </tr>
        <tr>
          <td>VitesseO19123</td>
          <td>RWG</td>
          <td>High</td>
          <td>High</td>
        </tr>
        <tr>
          <td>PSVO19215</td>
          <td>RWG</td>
          <td>Medium</td>
          <td>Medium</td>
        </tr>
        <tr>
          <td>PSVO19177</td>
          <td>CAM</td>
          <td>Low</td>
          <td>Medium</td>
        </tr>
        <tr>
          <td>VitesseO19112</td>
          <td>CAM</td>
          <td>Low</td>
          <td>Low</td>
        </tr>
        <tr>
          <td>VitesseO19108</td>
          <td>LCM</td>
          <td>Low</td>
          <td>Low</td>
        </tr>
        <tr>
          <td>PSVjong3</td>
          <td>LCM</td>
          <td>Low</td>
          <td>Low</td>
        </tr>
        <tr>
          <td>PSVO19173</td>
          <td>RCM</td>
          <td>Low</td>
          <td>High</td>
        </tr>
        <tr>
          <td>VitesseO19120</td>
          <td>RCM</td>
          <td>Low</td>
          <td>Low</td>
        </tr>
        <tr>
          <td>VitesseO19111</td>
          <td>LB</td>
          <td>Medium</td>
          <td>Medium</td>
        </tr>
        <tr>
          <td>PSVO19164</td>
          <td>LB</td>
          <td>High</td>
          <td>High</td>
        </tr>
        <tr>
          <td>PSVO19214</td>
          <td>RB</td>
          <td>High</td>
          <td>High</td>
        </tr>
        <tr>
          <td>VitesseO19116</td>
          <td>RB</td>
          <td>Medium</td>
          <td>Medium</td>
        </tr>
        <tr>
          <td>PSVO19659</td>
          <td>LCB</td>
          <td>High</td>
          <td>High</td>
        </tr>
        <tr>
          <td>VitesseO19114</td>
          <td>LCB</td>
          <td>Medium</td>
          <td>Medium</td>
        </tr>
        <tr>
          <td>PSVO19224</td>
          <td>RCB</td>
          <td>Medium</td>
          <td>Medium</td>
        </tr>
        <tr>
          <td>VitesseO19109</td>
          <td>RCB</td>
          <td>High</td>
          <td>Low</td>
        </tr>
      </tbody>
      <caption class="table-caption" style="caption-side: bottom;"><span>Table 1.</span> Comparison of predictability between the two halves</caption>
    </table>
</div>

The table summarises predictability values by position. Here is the key findings from the current project:

* Wingers from both teams have high predictability in relation to their pitch positioning.

This could be due to their tactical requirements. Wingers, typically employed by 4-3-3 formation, are tasked with staying up wide and hugging the touchline to create space for teammates. This could limit their playing area and make their positioning more predictable.

* Full back also had high to medium predictability in their positioning.

This could also due to their tactical requirement/individual characteristics. Defensive-oriented fullbacks should be associated with high predictability due to its proximity to their defender centroid, whereas attacking-oriented ones exhibit low predictability. In the study of Memmert et al. (2017), full backs from Bayern Munich and FC Barcelona were considered less predictable, it might be possible to say that full backs from this project are more defensive oriented than the others. 

* Strikers from both teams had medium to low predictability.

Again, This could be due to their tactical roles, running in behind the defensive line, running wide to create space or even dropping down to control the game. Strikers with less predictable movements poses danger to the opponent.

### 3.2 &nbsp; &nbsp; Between Halves

One of the objectives of the project was to compare the ApEn values between the halves. 

The plot shows there was a slight decline in the value of approximate entropy between two halves, which wasnt significant when you factor in the quantile of the distribution (see table 1). Most of player's positioning predictability did not change between two halves. Initially, I expected to observe higher ApEn value in the second half, caused by unpredictable (out of position) movements due to external factors such as fatigue. However, only two players increased their ApEn (PSVO19223 & VitesseO19109), indicating such external factors may not affect player's movement patterns and complexity. it is rather difficult to draw definitive conclusions about the impact of such factors on player movement patterns from this single match, though. 

![ApEn between halves](\img\posts\pitch-positioning\ApEn2.png)

In summary, ApEn values can offer valuable insights into the tactical/individual behaviors of players, such as their defensive or attacking orientations, as well as their tactical roles and actions. When combined with additional contextual information, ApEn could be an effective performance indicator for evaluating tactical aspects of a player's performance.

### 3.3 &nbsp; &nbsp; Limitation

**Small sample size**: The data is from a single football match, hence lacks the generalisability of the result. Due to its small size, there is no baseline measure that would conclude the magnitude of approximate entropy values. For example, a midfielder had the value of 0.139, which is considered high relative to other players. Is that really high? given ApEn can range from 0 to infinity. It is necessary to analyze additional match data to establish a more reliable benchmark.

**Contextual information**: The analysis did not include contextual information which is the key to football performance. Further development of this project can include factors such as tactical change, formation change, substituion, scoreline and more. It is also valuable to see the ApEn values during attack and defence, may provide an insight into how players position themselves during the specific phase. 

## 4 &nbsp; &nbsp; Reference

Memmert, D., Lemmink, K. A. P. M., & Sampaio, J. (2017). Current Approaches to Tactical Performance Analyses in Soccer using Position Data. *Sports medicine*, 47(1), 1-10.

Sampaio, J., & Maçãs V. (2012). Measuring tactical behaviour in football. *Int J Sports Med*, 33(5), 395-401.
