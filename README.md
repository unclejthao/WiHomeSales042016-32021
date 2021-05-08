# Wisconsin Home Sales Stats from April 2016 to March 2021
I looked at the public data from the Wisconsin Department of Revenue to answer the question of the best time to sell and/or buy your house in the state of Wisconsin.

The data I looked at was the last five years of available home sales found here:

https://www.revenue.wi.gov/pages/eretr/data-home.aspx

The data was broken down into csv files for each month.  In total there were 60 separate files.

### Combining CVS (Combining CSVs.ipynb)
I use Jupyter notebooks and Pandas to combine the individual files into one single csv file.  The purpose of this was to make importing the data into my SQLite database easier.

```
files = [i for i in glob.glob('*.{}'.format('csv'))]
combinedFiles = pd.concat([pd.read_csv(f, encoding = 'ISO-8859-1') for f in files ])
combinedFiles.to_csv("combined.csv", index=False, encoding = 'ISO-8859-1')
```

### Selecting data with SQLite
Imported “combined.csv” into SQLite and used SQLiteStudio to select the data that I wanted.

I only wanted :
1. Sellers are individuals not a corporation
1. No relationship between seller and buyer
1. Sale is for land and building
1. Single family homes
1. Is an sale or exchange

The RETRmlist.pdf shows the codes for the fields above.

The data I wanted was the Sales number, the date the transaction was recorded, the value of the sale, and County Name.

```
SELECT SaleNumber, DateRecorded, TotalRealEstateValue, CountyName
  FROM WI_home_sales_042016_032021
  WHERE
    GrantorType = '1' AND
    GrantorGranteeRelationship ='1' AND
    PropertyType = '2' AND
    PredominateUse = '1' AND
    (TransferType = '1' OR TransferType = '3')
;
```
### Cleaning data with Excel
After exporting the data I needed from SQLiteStudio into a CSV file (WIHouseSalesFiltered.csv), I used Microsoft Excel to update the data types of each column.  Since the date field was 7-8 digit numbers, I used the following formulas to break the numbers up and reconstruction them into Tableau friendly dates.

```
=IF(LEN(B2)>7,LEFT(B2,2),LEFT(B2,1))
=RIGHT(B2,6)
=LEFT(G2,2)
=RIGHT(G2,4)
=DATE(I2,F2,H2)
```

### Using Tableau for Data Visualization
Using the Date(month-year) as the x-axis and the average home sale and number of houses sold as the Y-axis I was able to find a clear cyclical pattern where both the lowest average sale price and number of houses sold happened during the month of February.  

![Lowest average home sale price and number of houses sold](https://github.com/unclejthao/WiHomeSales042016-32021/blob/master/Low.png)

The highest average sale price and number of houses sold usually happened during the months of June or July.

![Highest average home sale price and number of houses sold](https://github.com/unclejthao/WiHomeSales042016-32021/blob/master/High(1).png)

The forecast shows the same pattern, July was estimated to have the highest both in terms of average sale price and number of houses sold for the years 2021 and 2022.  Likewise, February was the predicted low for both metrics. 

![Estimated average home sale price and number of houses sold](https://github.com/unclejthao/WiHomeSales042016-32021/blob/master/Prediction.png)

The histogram shows a normal curve with an average sale price of $210,712 and a standard deviation of $156,282

![Estimated average home sale price and number of houses sold](https://github.com/unclejthao/WiHomeSales042016-32021/blob/master/Histogram.png)
