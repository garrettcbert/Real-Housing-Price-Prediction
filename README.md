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

[^1]: \<“FHFA House Price Index.” Www.fhfa.gov, www.fhfa.gov/data/hpi.\>

[^2]: \<“BLS Data Viewer.” Bls.gov, 2018, data.bls.gov/dataViewer/view/timeseries/CUUR0000SA0.\>


