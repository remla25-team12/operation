# Continuous Experiment
## Overview
As part of our continuous experiment, we want to evaluate whether certain UI feature can impact user behaviour with our app. From our current app, we think that it is easy for users to overlook that the individual cards under the People tab are clickable as there are no clear marking stating that the individual cards contain any links. In addition, even though they might realise it is clickable, it would be unclear what the link leads to.

 Thus, we want to deploy a feature that allows user to realise that these pages are clickable without hurting the aesthetics of the page. We decided to add a feature that have a linkedin icon shown next to each member's name when user hovers over it. To test whether it is effective, we will deploy 2 apps and use Istio to route traffic towards the 2 version. Prometheus and Grafana will be used to monitor and show the metrics comparision between the 2 version.

## Hypothesis

**Null hypothesis:**

Adding linkedin icon when hovering over user profile **does not** impact number of user clicks to our member’s profile.

**Alternative Hypothesis**

Adding linkedin icon when hovering over user profile **increase** number of user clicks to our member’s profile.

## Experiment
For the experiment, we will have two version of our app, namely v1 and v2. 
- **Version 1 (v1)** is the base version with no LinkedIn icon. 
- **Version 2 (v2)** introduces a LinkedIn icon that appears when hovering over a member's card, suggesting that the card is clickable and links to the member’s LinkedIn profile. 

For both version, clicking each member's card will direct to each members linkedin page. However, the key of this experiment is to evaluating whether having such icon can help user know that it directs to members linkedin page.

To measure whether such change lead to difference in user behaviour we will use the **number of total clicks members individual linkedin page** as metrics.

**V1**   \
<img src="imgs/Experiment-without-linkedin-example.png" alt="Grouped bar chart" title="Grouped bar chart" width="300"/>   \

**V2**   \
<img src="imgs/Experiment-example.png" alt="Grouped bar chart" title="Grouped bar chart" width="300"/>

## Instruction to reproduce
Follow instructions on readme **README** to setup the app and open grafana. Then to mannually create statistics, simply go to the peoples page on our app and refresh while click on each members link. These clicks are reflected in the **Total Click Comparision between Versions** Bar chart.

## Result
The following screenshot shows the Grafana dashboard panel specific for this experiment. It presents the comparision of number of clicks between the two version. The x axis is the two versions, where V1 is 0.5.11 and V2 is 0.5.12. The Y axis represents the total number of clicks.
![Pie chart screenshot](imgs/cont-exp-example-plot.png "Grouped bar chart")


## Decision Process
We compare the total clicks and can only conclude if number of total clicks in V1 is 20% higher than V2. However, in our result as shown in the image above, although V2, whcih is version 0.5.12 in this case, has a higher total click, it is not statistically significant enough for us to draw the conclusion. Thus, we cannot reject the null hypothesis.

## Conclusion
According to our statistics, we can conclude that the feature we introduced, which is having an linkedin icon when user hover over individual card in our people's page does not lead to more user clicking to access member's linkedin profile.
