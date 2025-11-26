## sales-trends-analysis_powerbi
Annonymized exploration of customer engagement, retention and check- in behaviour for a gym. Data model and dax measures used in this analysis will be provided at the end of the report. 

## Annonymisation notes 
To protect the confidentiality of the gym, this repository will only contain screenshots. All sensitive values have been blurred. The dashboard, data model and DAX structure remain representative of the analytical work. 

## Exploration Analysis 

### Project Overview 
The goal of this dashboard was to understand: 
* Daily and monthly check in patterns
* Year of Year performance overall (and within each entry type)
* Long terms trends and rolling averages 
* Customer engagement levels with different initiatives
* Demographics of existing customers
  
These insights will help to guide membership strategies, event programming, customer segmentations and retention improvement efforts. 

![Overview](screenshots/gym_analysis1.png)
This is a multi- angled view of customer activity in 2025. It intergrates summary KPIs (check ins, individual product performance and demographic insights) and overall trends. 

Date Slicer: 
* Hierarchial "Year and Month" slicer enables filtering across multi- year data and drilled down to months. It allows for easy navigation for time- series exploration.
  
Key metrics: 
* Total check ins: total number of check ins and gym activity within the selected time period
* Average check ins per customer: frequency on visits of an average member, with a higher number reflecting loyalty and recurring user behaviour

Daily activity trends: 
* Total daily check ins (blue line): reflects overall trend acros the year
  - Slicer allows filtering by year and month
  - Visible mid year dips and spikes suggesting seasonal behaviours
* Rolling 6 month average (dark line): trend line minimizes daily noise and better reflects long term directions
  - Visible rising trends towards the end of the year
The combination of daily check ins, and a rolling 6 month average helps to differentiate between short term volatility and long term trends in customer engagements. 

Entry type performance & YoY Changes: 
* YoY change: +xx% shows growth/ decline within each entry group
* Overall YoY change: -xx% indicates overall growth/ decline across the year

Age Distribution: 
* Histogram: shows strong concentration of check ins within binned age groups

![Customer Engagement(screenshots/gym_analysis2.png)]

Total check ins: The same trend line was used from the previous visual 

Event/ Incentive Marker: 
* Each colored dot represents a specific incentive or promotion (birthday events, initatives, promotion)
* Vertical dashed line indicate the major events
This visual gives insight into:
1. Which events drives the biggest spikes?
2. Are the promotion well timed and aligned with seasonal trends?
3. Do promotions sutain long term interest or short term spikes?
4. What campaigns relate to long term growths?

![Retention Rate(screenshots/gym_analysis3.png)]

Total check ins vs Returned Within 2 weeks: the bar graph shows the total check ins per day, while the line chart plots the number of people returning within *2 weeks* of their *first visit*
Retention table: table calculates return rate ratios for each month with 3 rentention windows (2 weeks, 3 months, 6 months)

This visual gives insight into: 
1. Which months build the strongest repeat patterns? (seasonal trends?)
2. Which days encourage repeat visits? Do weekdays or weekends differ in attracting casual entry members or members?
3. Understanding rentention by entry types (Do casual entry members come back more, as compared to those holding 10 pass memberships?) 

## Data Model Summary 

A separate *Initiatives dimension table* was created- containing the initative name, start date and type. The star schema design used allows for easy filtering and a cleaner model. 

Fact table: captures all check in details 
- check in date/ time
- entry/ event type
- customer ID
- contains all DAX measures

Dimension tables: 
*The primary key used to link the dimension to fact tables is the Customer ID. Not all variables in the tables have been listed

1. Calendar: fully populated (includes every single day in the reporting period, regardless of whether a check in occured on that date) date dimension table to drive time intelligence functions
- Date
- Day (and day_num) 
- Month (and month_num)

*The fact table might not cover every date, and would break any time series analysis* 

2. Initiatives table: separate table created to allow for easy filtering
- initative name, type
- start date

3. Customer: provides demographics, and customer information
- birth date
- address
- contact details

4. Rentention: stores DAX measures of retention metrics
- First visit (date)
- retention in 2 weeks
- rentention in 3 months
- rentention in 6 months 

All relatioships are single direction and references customer id as the primary key to avoid circular relationships 

## DAX Measures Used 
1. Customers = DISTINCTCOUNT(Checkins[Customer ID])

2. Total Checkins = SUM(Checkins[Count])

3. Average_Checkins = DIVIDE([Total Checkins], [Customers])

4. Rolling 6M Average = 
AVERAGEX(
    DATESINPERIOD(
        'Calendar'[Date],
        MAX('Calendar'[Date]),
        -6,
        MONTH
    ),
    [Total Checkins]
)

5. YoY % Change = 
VAR CurrentValue = [Total Checkins]  
VAR LastYearValue =
    CALCULATE(
        [Total Checkins],
        SAMEPERIODLASTYEAR('Calendar'[Date])
    )
RETURN
DIVIDE(CurrentValue - LastYearValue, LastYearValue)

6. ReturnedWithin2w = 
VAR first = Retention[FirstVisit]
VAR last  = first + 14
RETURN
IF (
    CALCULATE (
        COUNTROWS ( Checkins ),
        FILTER (
            Checkins,
            Checkins[Customer ID] = Rentention[Customer ID] &&
            Checkins[Date] > first &&
            Checkins[Date] <= last
        )
    ) > 0,
    1,
    0
)

For each customer's check in, the formula looks at subsequent check ins by the same customer and checks if it occurs again within the 2 week window. A binary flag is given (1/0) if the customer returned within the window or not. 

7. Return Rate_2wks = 
DIVIDE(
    SUM('Retention'[ReturnedWithin2w]),
    COUNT('Retention'[ReturnedWithin2w])
)
*the same syntax was done for 3 months, 6 months*


