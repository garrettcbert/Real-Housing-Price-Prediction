---
title: "Real Housing Price Prediction"
output: 
  html_document: 
    theme: lumen
    highlight: tango
author: "By: Garrett Seibert"
date: 12/28/24
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
library(tidyverse)
raw_housing_data <- read_csv("../../Desktop/RealHousingProject/data/hpi_master.csv")
inflation_data <- read_csv("../../Desktop/RealHousingProject/data/Inflation_Index.csv")
avg_interest_rates <- read_csv("../../Desktop/RealHousingProject/data/AvgInterestRate_20010131_20241031.csv")
cpi_data <- read_csv("../../Desktop/RealHousingProject/data/cpi_data.csv")
```

```{r, echo = FALSE, include = FALSE}
processed_cpi <- cpi_data %>% 
  select(Label, Value) %>% 
  mutate(YearMonth = paste0(substr(Label, 1, 4), "-", substr(Label, 6, 8))) %>% 
  select(-Label) %>% 
  rename(CPI = Value)

housing_data <- raw_housing_data %>% 
  drop_na() %>% 
  mutate(YearMonth = paste0(yr, '-', case_when(
    period == 1 ~ "Jan",
    period == 2 ~ "Feb",
    period == 3 ~ "Mar",
    period == 4 ~ "Apr",
    period == 5 ~ "May",
    period == 6 ~ "Jun",
    period == 7 ~ "Jul",
    period == 8 ~ "Aug",
    period == 9 ~ "Sep",
    period == 10 ~ "Oct",
    period == 11 ~ "Nov",
    period == 12 ~ "Dec"
  )))

realpricedata <- processed_cpi %>% 
  right_join(housing_data %>% filter(place_name == "United States", frequency == "monthly")) %>% 
  select(CPI, YearMonth, index_sa, yr) %>% 
  mutate(RealPrice = (index_sa/CPI)*100) %>% 
  drop_na() %>% 
  select(YearMonth, RealPrice, yr)
```

```{r, echo = FALSE}
avg_interest_rates <- avg_interest_rates %>%
  filter(`Security Type Description` == "Interest-bearing Debt") %>%
  mutate(
    `Record Date` = as.character(`Record Date`),
    MonthYear = paste0(substr(`Record Date`, 1, 4),
      "-",
      case_when(
        substr(`Record Date`, 6, 7) == "01" ~ "Jan",
        substr(`Record Date`, 6, 7) == "02" ~ "Feb",
        substr(`Record Date`, 6, 7) == "03" ~ "Mar",
        substr(`Record Date`, 6, 7) == "04" ~ "Apr",
        substr(`Record Date`, 6, 7) == "05" ~ "May",
        substr(`Record Date`, 6, 7) == "06" ~ "Jun",
        substr(`Record Date`, 6, 7) == "07" ~ "Jul",
        substr(`Record Date`, 6, 7) == "08" ~ "Aug",
        substr(`Record Date`, 6, 7) == "09" ~ "Sep",
        substr(`Record Date`, 6, 7) == "10" ~ "Oct",
        substr(`Record Date`, 6, 7) == "11" ~ "Nov",
        substr(`Record Date`, 6, 7) == "12" ~ "Dec",
        TRUE ~ NA_character_
      )
    )
  ) %>% select(MonthYear, `Average Interest Rate Amount`) %>% rename("YearMonth" = MonthYear)

```

```{r, echo = FALSE, include = FALSE}
real_housing_mortgage <- realpricedata %>% 
  filter(yr >= 2001) %>% 
  left_join(inflation_data) %>% 
  rename(inflation = Value) %>% 
  left_join(avg_interest_rates) %>% 
  rename(interest_rate = `Average Interest Rate Amount`) %>% 
  mutate(interest_rate = as.double(interest_rate))

real_housing_mortgage_ordered <- real_housing_mortgage %>%
  mutate(
    Year = as.numeric(substr(YearMonth, 1, 4)),
    Month_abb = substr(YearMonth, 6, 8),
    Month = match(Month_abb, month.abb)
  ) %>%
  arrange(Year, Month) %>%
  mutate(
    YearMonth = factor(YearMonth, levels = unique(YearMonth))
  ) %>%
  select(-Year, -Month_abb, -Month)
```

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

[^1]: \<“FHFA House Price Index.” Www.fhfa.gov, www.fhfa.gov/data/hpi.\>

[^2]: \<“BLS Data Viewer.” Bls.gov, 2018, data.bls.gov/dataViewer/view/timeseries/CUUR0000SA0.\>

$$
\frac{\text{Nominal HPI}}{\text{CPI}} * 100 = \text{Real HPI}
$$

The graph below represents the change in the real housing price index over the last 24 years. As you can see, events like the 2008 housing market crash and the 2020 pandemic had substantial impacts on housing prices.

```{r, echo = FALSE, warning = FALSE, fig.align = 'center'}
ggplot(real_housing_mortgage_ordered, aes(x = YearMonth, y = RealPrice, group = 1)) +
  geom_line(color = "blue", size = 1) +
  geom_point(color = "red", size = 2) +
  theme_minimal(base_size = 14) +
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1, color = "gray20"),
    plot.title = element_text(hjust = 0.5, face = "bold", size = 16),
    plot.subtitle = element_text(hjust = 0.5, size = 12, color = "gray40"),
    axis.title = element_text(face = "bold"),
    panel.grid.major = element_line(color = "gray80", linetype = "dotted")
  ) +
  scale_x_discrete(breaks = c("2008-Jan", "2020-Aug")) +
  scale_y_continuous(limit = c(70, 145), labels = scales::comma) +
  labs(
    title = "Housing Price Indices Over the Past 24 Years",
    subtitle = "Adjusted for Inflation and Seasonality",
    x = "Year-Month",
    y = "Real Housing Price Index",
    caption = "Data Source: Federal Housing Finance Agency"
  )
```

## **Forming a Prediction**

It is difficult to tell just by looking at the graph above where the Real HPI will go in the following months. It is especially difficult considering how unpredictable our economy is. However, there are things that we can look at to try and extract a predicted rate to use to predict a specific value in the near future.

The Federal Reserve is targeting around a 2% inflation rate, and as of right now, we are very close to or already in this zone. Meaning that in order to find the rate at which the housing prices will increase or decrease in the near future, we want to take the rate from a different time period. One with similar economic activity that we see today:

-   **Around 2% inflation**

-   **Similar demand for houses**

-   **Similar interest rates**

### **Finding a moderate inflation rate:**

The graph below shows the change in inflation rate over the past 24 years. There are many time periods that we can chose from, but we can certainly exclude any time after or during 2020, as that was a time of economic crisis as well as economic recovery. Since we are almost fully recovered, economically speaking, from the 2020 pandemic, this is not a good time frame to chose from.

Inflation rate data was also gathered from the **US Bureau of Labor Statistics**[^3].

[^3]: \<“BLS Data Viewer.” Bls.gov, 2018, data.bls.gov/dataViewer/view/timeseries/CUUR0000SA0.\>

```{r, echo = FALSE, fig.align = 'center'}
real_housing_mortgage_ordered %>%
  ggplot(aes(YearMonth, inflation, group = 1)) +
  # Plot the main line
  geom_line(color = "blue", size = 1.2) +
  # Add horizontal lines for inflation thresholds
  geom_hline(yintercept = 3, color = "red", linetype = "dashed", size = 1.2) + 
  geom_hline(yintercept = 1, color = "darkgreen", linetype = "dashed", size = 1.2) + 
  # Annotate the horizontal lines
  annotate("text", x = 50, y = 3 + 0.3, label = "3% Inflation", color = "red", size = 5, fontface = "italic") +
  annotate("text", x = 50, y = 1 - 0.3, label = "1% Inflation", color = "darkgreen", size = 5, fontface = "italic") +
  # Adjust x-axis with desired breaks and expand the x-axis range
  scale_x_discrete(breaks = c("2021-Jan")) +
  # Limit y-axis range for better focus
  scale_y_continuous(limits = c(0, 7.5), expand = c(0, 0)) +
  # Enhance the theme
  theme_minimal(base_size = 14) +
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1, color = "gray20"),
    axis.title = element_text(face = "bold"),
    plot.title = element_text(face = "bold", size = 16, hjust = 0.5),
    plot.subtitle = element_text(size = 12, hjust = 0.5, color = "gray40"),
    panel.grid.major = element_line(color = "gray85", linetype = "dotted"),
    panel.grid.minor = element_blank()
  ) +
  # Add titles and labels
  labs(
    title = "Inflation Trends Over Time",
    subtitle = "With Thresholds for 1% and 3% Inflation",
    x = "Year-Month",
    y = "Inflation Rate (%)",
    caption = "Data Source: US Bureau of Labor Statistics"
  )
```

### **Finding Similar Demand:**

A time of similar housing demand means no times of housing market crisis. This removes the beginning of 2008 to the end of 2009, the time during the pandemic as well as the early 2000s of high housing demand that led to the 2008 crash. We are still left with the years between 2010 and 2020 to find a time that best corresponds to the economy that we are experiencing today. Using one more qualifier we can filter to a more specific range.

### **Finding Similar Interest Rates:**

The graph below shows the change in interest rates from the past 24 years. As you can see, there is only one good time frame that matches with where the interest rates have been recently. The years 2010 and 2011 not only fit our criteria for moderate inflation and similar demand, but also match with our current interest rates. Furthermore, the Fed is expecting to cut interest rates slightly in the following months putting the rate right near the level that it was at between 2010 and 2011.

The data for the interest rates was collected from the website of the **Federal Reserve Bank of St. Louis**[^4].

[^4]: FRED. “Effective Federal Funds Rate.” *Stlouisfed.org*, 1 Nov. 2024, fred.stlouisfed.org/series/FEDFUNDS.

```{r, echo = FALSE, fig.align = 'center'}
real_housing_mortgage_ordered %>%
  ggplot(aes(YearMonth, interest_rate, group = 1)) + 
  geom_line(color = "blue", size = 1.2) + 
  geom_hline(yintercept = 3.35, color = "red", linetype = "dashed", size = 1) + 
  geom_hline(yintercept = 2.7, color = "darkgreen", linetype = "dashed", size = 1) +
  scale_x_discrete(breaks = c("2010-Jan", "2011-Dec", "2024-May"), 
                   labels = c("Jan 2010", "Dec 2011", "May 2024")) + 
  scale_y_continuous(expand = expansion(mult = c(0.05, 0.1))) +
  labs(
    title = "Interest Rate Trends Over Time",
    x = "Year-Month",
    y = "Interest Rate (%)",
    caption = "Source: Federal Reserve Bank of St. Louis"
  ) +
  theme_minimal(base_size = 14) +
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1, color = "gray20"),
    axis.text.y = element_text(color = "gray20"),
    axis.title = element_text(face = "bold"),
    plot.title = element_text(face = "bold", size = 16, hjust = 0.5, color = "blue"),
    plot.subtitle = element_text(size = 12, hjust = 0.5, color = "gray40"),
    panel.grid.major = element_line(color = "gray85", linetype = "dotted"),
    panel.grid.minor = element_blank(),
    plot.caption = element_text(hjust = 1, size = 10, color = "gray50")
  )

```

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

```{r, echo = FALSE, fig.align = 'center'}
real_housing_mortgage_ordered %>%
  ggplot(aes(YearMonth, RealPrice, group = 1)) +
  # Plot the main line
  geom_line(color = "blue", size = 1) +
  # Add the slope line
  geom_abline(slope = -0.4565085, intercept = 137.5, color = "red", linetype = "dashed", size = 1) +
  # Annotate the slope value
  annotate(
    "text", 
    x = 90,  # Adjust x and y for proper placement
    y = 120, 
    label = "Slope: -0.457", 
    color = "red", 
    fontface = "italic",
    size = 5
  ) +
  # Add extended x-axis room
  scale_x_discrete(breaks = c("2010-Jan", "2011-Dec"), expand = expansion(mult = c(0.05, 0.2))) +
  # Limit y-axis range
  scale_y_continuous(limits = c(70, 145), expand = c(0, 0)) +
  # Enhance the theme
  theme_minimal(base_size = 14) +
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1, color = "gray20"),
    axis.title = element_text(face = "bold"),
    plot.title = element_text(face = "bold", size = 16, hjust = 0.5),
    plot.subtitle = element_text(size = 12, hjust = 0.5, color = "gray40"),
    panel.grid.major = element_line(color = "gray85", linetype = "dotted"),
    panel.grid.minor = element_blank()
  ) +
  # Add titles and labels
  labs(
    title = "Real Housing Prices with Sample Trendline",
    subtitle = "Examining the predicted negative slope over time",
    x = "Year-Month",
    y = "Real Housing Price Index",
    caption = "Data Source: US Bureau of Labor Statistics"
  )

```

## **How do we apply this?**

We can use this predicted rate from our matching time period and apply it to the most recent **Real HPI** values to find the predicted real housing price in April of 2025, though this can method can be used with any future month that meets our assumptions.

$\text{Predicted Real HPI value in } x \text{ months} = \text{Most recent real HPI value} + \text{Predicted Rate of Change} * x$

$\text{August 2024 Real HPI} = 135.65293$

$x \text{ (number of months between most recent month and month being projected)} = 8$

$\text{Predicted Rate of change} = -0.4565085$

$$\text{Predicted Average Real Housing Price in April 2025} = 135.65293 + -0.4565085 * 8 = 132.0009$$

The graph below applies the sample rate of change gathered from the months between 2010 and 2011 to the most recent gathered **Real HPI** number. We can see that in 8 months, using the sample rate, the **Real HPI** would be 132.

```{r, echo = FALSE, fig.align = 'center'}
real_housing_mortgage_ordered %>%
  ggplot(aes(YearMonth, RealPrice, group = 1)) +
  # Plot the main line
  geom_line(color = "blue", size = 1) +
  # Add the slope lines
  geom_abline(slope = -0.4565085, intercept = 137.5, color = "red", linetype = "dashed", size = 1) +
  geom_abline(slope = -0.4565085, intercept = 265, color = "red", linetype = "dashed", size = 1) +
  # Add the horizontal line
  geom_hline(yintercept = 132.0009, color = "darkgreen", linetype = "dotted", size = 1) +
  # Annotate the slope
  annotate(
    "text", 
    x = 100,  # Adjust x for proper placement
    y = 120, 
    label = "Sample Slope: -0.457", 
    color = "red", 
    fontface = "italic",
    size = 5
  ) +
  # Annotate the horizontal line
  annotate(
    "text", 
    x = 100,  # Adjust x for proper placement near Feb 2025
    y = 132.0009 + 3,  # Position slightly above the line
    label = "Predicted Value for Apr 2025: 132.0009", 
    color = "darkgreen", 
    fontface = "bold",
    size = 4
  ) +
  # Add extended x-axis room
  scale_x_discrete(breaks = c("2010-Jan", "2011-Dec"), expand = expansion(mult = c(0.05, 0.2))) +
  # Limit y-axis range
  scale_y_continuous(limits = c(70, 145), expand = c(0, 0)) +
  # Enhance the theme
  theme_minimal(base_size = 14) +
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1, color = "gray20"),
    axis.title = element_text(face = "bold"),
    plot.title = element_text(face = "bold", size = 16, hjust = 0.5),
    plot.subtitle = element_text(size = 12, hjust = 0.5, color = "gray40"),
    panel.grid.major = element_line(color = "gray85", linetype = "dotted"),
    panel.grid.minor = element_blank()
  ) +
  # Add titles and labels
  labs(
    title = "Real Housing Prices with Sample Trendline",
    subtitle = "Examining the predicted negative slope over time",
    x = "Year-Month",
    y = "Real Housing Price Index",
    caption = "Data Source: US Bureau of Labor Statistics"
  )

```

## **Conclusion**

Using these factors, I have predicted that if all assumptions are met, housing will become more affordable in the coming months. Policies made by trusted economists, in all honesty, rarely go perfectly to plan, so naturally this model depends on much going right and very little going wrong.

But, ultimately, it is important to me that prospective home buyers to get all of the information, not just the information the media readily gives out. If the public believes that purchasing a home is entirely unaffordable and will continue to be that way, the demand will forever be lower than it could be, meaning a surplus of supply of homes and lost economic potential. The pandemic caused the US economy issues that it is still working on recovering from, but we are so much better now than we were even a year ago and I believe that the near future of the housing market is worth being optimistic about.
