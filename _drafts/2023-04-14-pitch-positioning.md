---
layout: post
title: "Pitch Positioning Predictability in Football"
subtitle: "Apply approximate entropy to quantify how predictable player's positioning is."
background: '/img/posts/pitch-positioning/header.jpg'
---

## **1 &nbsp; &nbsp; Project Introduction**

This small project have two purposes:

1. Develop a collective performance indicator to quantify pitch positioning predictability of football players.
2. Compare the predictability values between two halves of a match.

To achieve the first objective, I analysed the tracking data of a football match and computed approximate entropy (ApEn) values for each player. I then used these values as a collective performance indicator that quantifies the pitch positioning predictability of each player.

To achieve the second objective,  I compared the obtained ApEn values of players in the first half of matches to those in the second half.

### **1.2 &nbsp; &nbsp; Approximate Entropy**

Approximate entropy (ApEn) is a technique used to quantify the amount of predictability in a time-series data without any prior knowledge on the data-generating distribution. A low value of ApEn indicates the time-series is predictable, a high value indicates otherwise. Previous studies have applied ApEn to the tracking data from a football match to quantify movement patterns of players in terms of pitch positioning preditability. Examples of such studies are following:


*  Sampaio & Maçãs (2011) investigated the effect of small-sided games on player's tactical understanding. The ApEn was applied to the distance between **team's centroid** and each player over the match. Lower ApEn in the post test indicated player's expertise in football increased due to the training sessions. 

* Memmert et al. (2017) applied ApEn to the distance between each player and **position-specific centroids** in a game between Bayern Munich and Barcelona. The part of  result showed Bayern Munich central midfielder was highly predictable in his positioning relative to his positional cenetroid, indicating his pivotal role between players. 

This project followed the approach of Memmert et al. (2017), which are following:

* Calculate positional centroids by averaging the locations of players with the same position group (DEF, MID & ATT) in every 100 milliseconds. 
* Calculate the distance between each player and his corresponding centroid in every 100 milliseconds. 
* Compute ApEn from the distance values of each player, which indicates how predictable player's positioning is.
* Classify the ApEn value in three groups (high, medium & low) based on the quantiles of distribution of ApEn.

## **2 &nbsp; &nbsp; Data**

The data used for this project is tracking data from a football match, including the following features:

* *Timestamp*: Time elapsed since the start of the match in milliseconds.
* *X*: X-coordinate of the player's position on the pitch.
* *Y*: Y-coordinate of the player's position on the pitch.
* *Speed*: The speed of the player in meters per second.
* *Distance*: The distance the player has covered since the start of the match in meters.
* *Heartbeat*: Heartbeat measured per min.
* *TransponderID*: A unique identifier for the transponder attached to the player.
* *PlayerID*: A unique identifier for the player.
* *Position*: The player's position on the pitch, such as defender or attacker.
* *Detailed_Position*: A more specific description of the player's position, such as right back or center forward.

### **2.1 &nbsp; &nbsp; Data Preparation**

Although the data is in a clean tabular format, some preprocessing steps were necessary before conducting the exploratory analysis. The following are some of the crucial steps taken:

* Remove goal keepers as they do not contribute to the formations of positional centroids.
* Handle missing values in *X* and *Y* columns. Since the data is recorded in milliseconds, preceding and next rows should contain similar coordinate values. Hence forward filling was used.
* Calculate positional centroids and the distance between each player and their corresponding centroid at each timestamp.

### **2.2 &nbsp; &nbsp; Data Analysis - Visualising Centroids**

**Positional Centroids**

Positional centroids are calculated by averaging locations of players who belong to the same position (e.g. defenders). The below is an example of such centroids. 

![Image of position specific centroids](\img\posts\pitch-positioning\positional_centroids.jpg)

The figure shows the average location of players for the first half and their corresponding centroids (blue circle). White dotted lines indicate the distance between players and their centroid. 

**Team Centroids**

Team centroid was not used in this project, but the below is a figure to illustrate the difference between the two.

![Image of team centroids](\img\posts\pitch-positioning\team_centroids.jpg)

The reason for choosing positional centroids over the other for positioning predictability is its sensitibity to capture player's movement patterns better (Memmert et al., 2017).

## 3 &nbsp; &nbsp; Result

The plot shows player's locations on the pitch during two halves for PSV and Vitesse. The colour of the circle is determined by the player's ApEn value, which captures the player's movement complexity on the field. 

![Image of ApEn values on the pitch](\img\posts\pitch-positioning\approximate_entropy_5.jpg)




































<style>
.table-container {
  display: flex;
  justify-content: center;
}
</style>

<div class="table-container">
    <table border="1" class="dataframe">
      <caption class="table-caption"><span>Table 2.</span> Comparison of predictability between the two halves</caption>
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
    </table>
</div>