# absolutetrash

## An exploration of flytipping in Birmingham
![Histogram](assets/flytipping1.jpeg)

Using publicly available datasets on fly-tipping I have conducted exploratory data analysis then built a time series analysis based on two datasets for incidents reported in Birmingham, and one dataset for all incidents reported nationally using Python. The data shows that fly-tipping has increased nationally, not just in Birmingham, although as a member of the community here I had anticipated that the anecdotal rise in fly-tipping incidents was a result of bin strikes. What the data reveals is that this is a UK-wide issue, and that an investigation into the root causes of fly-tipping will help us remedy the increase in reports.

## Data Sources

[Monthly data for fly-tipping incidents in Birmingham was obtained and consolidated from mysociety](https://github.com/mysociety/fms_geographic_data/tree/master)

mysociety operates fixmystreet.com, a website where citizens can report fly-tipping incidents, amongst other local issues.

[Yearly data for fly-tipping incidents across the UK was obtained from gov.uk](https://www.gov.uk/government/statistics/fly-tipping-in-england).

The UK government publishes a wide variety of statistical data, fly-tipping data is collected from local authorities.

## Data Cleaning

Very little data cleansing was required as the data has been aggregated and both organisations provide background information on their methodologies which show consistency in data collection and cleansing. 

Data Cleaning covered:

* Checking for duplicate rows
* Checking for null values, removing rows with missing values
* Checking for errors in values
* Standardising date - ensuring 'date' column was formatted as mm/dd/yyyy (datetype) and 'reports' as a whole number (int64)

No nulls were detected in gov.uk data using Power Query's column profiling:

<img src="/assets/PQ - LA data - No null.png" alt="Power Query No Null gov.uk">

#### $${\color{blue}Figure 1}$$

Some errors were detected in the total number of incidents for certain regions/years. There were 21 rows with errors of a total of 2930 rows of data (<1%), and so the errors were removed rather than replaced. The datset is large enough to be viable without these 21 rows:

<img src="/assets/PQ - error with total incidents - LA.png">

#### $${\color{blue}Figure 2}$$

However once resolved there were no further issues with the fixmystreet.com data:

<img src="/assets/PQ FMS no null.png">

#### $${\color{blue}Figure 3}$$

Duplicated reports are not able to be identified from these aggregated datasets and so we are reliant on the methodologies used by mysociety and the UK government. 

## EDA

Exploratory data analysis was conducted in PowerBI where the two different datasets could be brought into one model and viewed as a schema, where relationships could be made between related columns:

<img src="/assets/pqschema.png"> 

#### $${\color{blue}Figure 4}$$

A comparison of the __gov.uk__ and __fixmystreet.com (FMS)__ data shows that whilst there are fewer reports in the FMS dataset, the shape of and trend in the data appears to correlate.

<img src="/assets/pbi_bham.png">

#### $${\color{blue}Figure 5}$$

## Time Series Analysis

### The full code for the Time Series analysis undertaken in Python can be found [here](https://github.com/ducksinarowdata/absolutetrash/blob/main/Flytipping_Summative.ipynb)

Key steps are illustrated below:

* Firstly, we visualise the FMS data as line graph

```
# Defining a function that when called will output a line plot for us.
# We will pass the dataframe, the x and y columns and a title

def plot_df(df, x, y, title="", xlabel='Date', ylabel='Reports', dpi=100):

    plt.figure(figsize=(16,5), dpi=dpi)
    plt.plot(x, y, color='tab:red')
    plt.gca().set(title=title, xlabel=xlabel, ylabel=ylabel)
    plt.show()

# Call the function we just created to plot the line graph of the values
plot_df(df_fms, df_fms.index, y=df_fms.reports, title='Monthly Reports of Flytipping in Birmingham, UK (FMS')
```
<img src="/assets/fms_ts.png">

#### $${\color{blue}Figure 6}$$

* Secondly, an additive composition is created to show a breakdown of the trend, seasonality and residuals in the data. This data shows both an upward trend and seasonality.

```
from statsmodels.tsa.seasonal import seasonal_decompose
from dateutil.parser import parse

result_add = seasonal_decompose(df_fms['reports'], model='additive', extrapolate_trend='freq')

result_add.plot()
plt.show()
```
<img src="/assets/fms_decomp.png">

#### $${\color{blue}Figure 7}$$

* Finally, after the data is prepared, it is split into test and train datasets to begin predictive modelling.

```
# Split data into train-test
# We are going to train on the first months and test on the last 24 months

steps = 24 # predicting the final 24 months
data_train = df_fms[:-steps]  

data_test  = df_fms[-steps:]   # this sets data_test to be the last 24 values

print(f"Train dates : {data_train.index.min()} --- {data_train.index.max()}  (n={len(data_train)})")
print(f"Test dates  : {data_test.index.min()} --- {data_test.index.max()}  (n={len(data_test)})")

# Plotting the test/train data
fig, ax = plt.subplots(figsize=(7, 2.5))
data_train['reports'].plot(ax=ax, label='train')
data_test['reports'].plot(ax=ax, label='test')
ax.legend();
```
The model shows the number of predicted reports of fly-tipping incidents for 2019-2020 and shows that reports are predicted to begin dropping in early 2020.

<img src="/assets/fms_tt.png">

#### $${\color{blue}Figure 8}$$

Returning to Power BI (and __actual__, not predictive, data) we can see that at a local level, these predictions hold true:

<img src="/assets/pbi_local.png">

#### $${\color{blue}Figure 9}$$

It is also possible to see that against other large cities in the UK (ppopulations approx. 500,000), Birmingham (with a population of 1m) does not have a high number of reports. We would expect that at double the size, the city may have double the reported number of incidents.

Nationally, the trend continues upwards for reported incidents:

<img src="/assets/pbi_national.png">

#### $${\color{blue}Figure 10}$$

## Results, Takeaways and Conclusion

* Birmingham, relative to its size, does not have a statistically high number of reported fly-tipping incidents. Bin strikes do not appear to have had a lasting, adverse effect on the reported statistics.
* In large cities, reported incidents appear to be trending down. This is counter to the national (UK) trend, where reported incidents are trending up.
* Additional research and modelling of fly-tipping data to analyse other contributing factors (ie. the pandemic which occurred in 2020, where we see fly-tipping reports at a peak, and cuts to local authority spending) will give more insight into the complex causes of fly-tipping, and how these incidents might be mitigated or remedied.
* Future analysis could also look at the population of a region or city versus number of reported insights, this will help draw out insight on what types of areas are most prone to fly-tipping.
