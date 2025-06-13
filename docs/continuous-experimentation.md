# Continuous Experiment
## Overview
As part of our continuous experiment, we want to evaluate whether certain UI feature can impact user behaviour with our app. From our current app, we think that it is easy for users to overlook that individual cards are clickable.

 Thus, we want to deploy a feature that allows user to realise that these pages are clickable without hurting the aesthetics of the page. We decided to add a feature that have a linkedin icon shown next to each member's name when user hovers over it. To test whether it is effective, we willill deploy 2 apps and use Istio to route traffic towards the 2 version. Prometheus and Grafana will be used to monitor and show the metrics comparision between the metrics betwen the 2 version.

## Hypothesis

**Null hypothesis:**

Adding linkedin icon when hovering over user profile does not impact number of user clicks to our member’s profile

**Alternative Hypothesis**

Adding linkedin icon when hovering over user profile will increase number of user clicks to our member’s profile.

## Experiment
The experiment we will do is have two version of our app v1 and v2. Version 1 (v1) is the base version with no LinkedIn icon. Version 2 (v2) introduces a LinkedIn icon that appears when hovering over a member's card, suggesting that the card is clickable and links to the member’s LinkedIn profile.d will direct to each members linkedin page. V1 does not have such feature.

To measure whether such change lead to difference in user behaviour we will use the **number of clicks to individual linkedin page** as metrics.

## Instruction to reproduce

## Result