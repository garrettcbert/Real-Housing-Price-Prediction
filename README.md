# Real Housing Price Prediction

#### By: Garrett Seibert
#### 12/28/24


## **Introduction**

The housing market plays a pivotal role in the US economy. In this investigation, I look to accurately predict an increase in overall housing affordability. In order to build an accurate prediction, I need to perform analysis on factors such as the inflation rate, interest rates, and the housing prices themselves. Finding conclusions about the housing market can tell us much about the economy as a whole as it is often a good representation of the state of the economy. Affordable housing means wage rates comparable to costs, an increase in GDP, and perhaps most importantly, more opportunities for all people in this country to sleep with a roof over their heads. Predicting housing affordability means much more than just predicting future economic upturn, it means predicting quality of American life. But because the economy can be so wildly unpredictable at times, in order for a prediction to be possible, a few assumptions must be stated.

## **Assumptions and Limitations:**

Events like a spontaneous world event, political changes or even expectations of the future can drastically change the housing market and the future prices. This is why, in order to conduct any analysis whatsoever we need to assume the following things will stay true in the near future:

-   **The inflation rate will remain around 2%**

-   **There are no world events (global pandemic, wars, etc.)**

-   **Demand for housing remains relatively constant**

-   **The Fed's goals / policies remain constant**

Because of how unpredictable our economy can be at times, the natural limitation of the following model is that is will likely only work for the very near future. The world is always changing, so this model is more a hypothetical representation of the housing market if the economy stays consistent and predictable.

## **Natural Change in Housing Prices:**

The **Housing Price Index** (HPI) is the number that the US government attributes to the overall price level of houses on the market. This number is constantly rising due to the fact that the inflation rate is always positive. However, especially to new home buyers doing research on affordability, this number can be misleading. Yes, prices are always going up, but given a stable and consistent economy targeting around 2% inflation and moderate interest rates, wages are also always going up. When we adjust the HPI numbers for inflation, however, we can find the **Real HPI** which is a better representation of the state of the housing market. I pulled the HPI numbers from the past 24 years from the **Federal Housing Finance Agency**[^1]. I gathered the Consumer Price Index (CPI) numbers from the **US Bureau of Labor Statistics**[^2]. The CPI is the way that the US government measures changes in overall prices (inflation). We can find the **Real HPI** adjusted for inflation using the following formula for each month of recorded HPI:

[^1]: "FHFA House Price Index.” Www.fhfa.gov, www.fhfa.gov/data/hpi.

[^2]: “BLS Data Viewer.” Bls.gov, 2018, data.bls.gov/dataViewer/view/timeseries/CUUR0000SA0.

$$
\frac{\text{Nominal HPI}}{\text{CPI}} * 100 = \text{Real HPI}
$$

The graph below represents the change in the real housing price index over the last 24 years. As you can see, events like the 2008 housing market crash and the 2020 pandemic had substantial impacts on housing prices.

<p align="center">
  <img src=https://github.com/garrettcbert/Real-Housing-Price-Prediction/images/HPI_data.png width="650" height="450"/>
</p>

## **Forming a Prediction**

It is difficult to tell just by looking at the graph above where the Real HPI will go in the following months. It is especially difficult considering how unpredictable our economy is. However, there are things that we can look at to try and extract a predicted rate to use to predict a specific value in the near future.

The Federal Reserve is targeting around a 2% inflation rate, and as of right now, we are very close to or already in this zone. Meaning that in order to find the rate at which the housing prices will increase or decrease in the near future, we want to take the rate from a different time period. One with similar economic activity that we see today:

-   **Around 2% inflation**

-   **Similar demand for houses**

-   **Similar interest rates**

### **Finding a moderate inflation rate:**

The graph below shows the change in inflation rate over the past 24 years. There are many time periods that we can chose from, but we can certainly exclude any time after or during 2020, as that was a time of economic crisis as well as economic recovery. Since we are almost fully recovered, economically speaking, from the 2020 pandemic, this is not a good time frame to chose from.

Inflation rate data was also gathered from the **US Bureau of Labor Statistics**[^3].

[^3]: “BLS Data Viewer.” Bls.gov, 2018, data.bls.gov/dataViewer/view/timeseries/CUUR0000SA0.

<p align="center">
  <img src=images/Inflation_trends.png width="650" height="450"/>
</p>

### **Finding Similar Demand:**

A time of similar housing demand means no times of housing market crisis. This removes the beginning of 2008 to the end of 2009, the time during the pandemic as well as the early 2000s of high housing demand that led to the 2008 crash. We are still left with the years between 2010 and 2020 to find a time that best corresponds to the economy that we are experiencing today. Using one more qualifier we can filter to a more specific range.

### **Finding Similar Interest Rates:**

The graph below shows the change in interest rates from the past 24 years. As you can see, there is only one good time frame that matches with where the interest rates have been recently. The years 2010 and 2011 not only fit our criteria for moderate inflation and similar demand, but also match with our current interest rates. Furthermore, the Fed is expecting to cut interest rates slightly in the following months putting the rate right near the level that it was at between 2010 and 2011.

The data for the interest rates was collected from the website of the **Federal Reserve Bank of St. Louis**[^4].

[^4]: “Effective Federal Funds Rate.” *Stlouisfed.org*, 1 Nov. 2024, fred.stlouisfed.org/series/FEDFUNDS.

<p align="center">
  <img src=images/interest_rate.png width="650" height="450"/>
</p>

## **Calculating the Sample Rate**

Using our sample time period, we can extract a rate of change that we can then apply to the current state of **Real HPI**. The equation for predicted rate of change given any relationship goes as follows:

$$\hat{\beta}_1 = r * \frac{s_y}{s_x}$$

$\hat{\beta}_1 = \text{Estimated rate of change}$

$r = \text{Correlation between the x variable (Months) and the y variable (Real Housing Prices)}$

$s_y = \text{Standard devaition of all y values in our sample}$

$s_x = \text{Standard devaition of all x values in our sample}$

Because we need the x values to be linear and numeric, we will substitute the Year-Month for months in numeric form, with a growing rate of 1 per month.

### **Calculations for correlation coefficient** ($r$)

The full calculation for understanding the relationship between Month-Numeric and Real Housing Price Index over the this 2 year duration goes as follows:

$$r = \frac{\sum_{i=1}^n (X_i - \bar{X})(Y_i - \bar{Y})}{\sqrt{\sum_{i=1}^n (X_i - \bar{X})^2 \cdot \sum_{i=1}^n (Y_i - \bar{Y})^2}}$$

$n = \text{The number of data points (24 months)}$

$\bar{X} = \text{The average real housing price across all data points}$

$\bar{Y} = \text{Half of the number of months (12.5) / average number of months}$

$X_i = \text{The specific real housing price of the } i\text{-th month}$

$Y_i = \text{The corresponding number of the } i\text{-th month}$

When calculated:

$$r = -0.9588151$$

This means there is a strong negative correlation. In other words, during the years 2010 and 2011 real housing prices fell relatively consistently each month.

After finding the standard deviation for both sets of values:

$s_y = 3.366658$

$s_x = 7.071068$

Plugging everything in:

$$\hat{\beta}_1 = -0.9588151 * \frac{3.366658}{7.071068} = -0.4565085$$

This means that for every month, our predicted change in real housing price index per month is -0.4565.

The graph below models how this sample rate is seen through the months in between 2010 and 2011.

<p align="center">
  <img src=images/sample_rate.png width="650" height="450"/>
</p>

## **How do we apply this?**

We can use this predicted rate from our matching time period and apply it to the most recent **Real HPI** values to find the predicted real housing price in April of 2025, though this can method can be used with any future month that meets our assumptions.

$\text{Predicted Real HPI value in } x \text{ months} = \text{Most recent real HPI value} + \text{Predicted Rate of Change} * x$

$\text{August 2024 Real HPI} = 135.65293$

$x \text{ (number of months between most recent month and month being projected)} = 8$

$\text{Predicted Rate of change} = -0.4565085$

$$\text{Predicted Average Real Housing Price in April 2025} = 135.65293 + -0.4565085 * 8 = 132.0009$$

The graph below applies the sample rate of change gathered from the months between 2010 and 2011 to the most recent gathered **Real HPI** number. We can see that in 8 months, using the sample rate, the **Real HPI** would be 132.

<p align="center">
  <img src=images/final_prediction.png width="650" height="450"/>
</p>

## **Conclusion**

Using these factors, I have predicted that if all assumptions are met, housing will become more affordable in the coming months. Policies made by trusted economists, in all honesty, rarely go perfectly to plan, so naturally this model depends on much going right and very little going wrong.

But, ultimately, it is important to me that prospective home buyers to get all of the information, not just the information the media readily gives out. If the public believes that purchasing a home is entirely unaffordable and will continue to be that way, the demand will forever be lower than it could be, meaning a surplus of supply of homes and lost economic potential. The pandemic caused the US economy issues that it is still working on recovering from, but we are so much better now than we were even a year ago and I believe that the near future of the housing market is worth being optimistic about.
