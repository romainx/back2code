---
title: Working days
date: '2016-11-23'
categories: ['data']
tags: ['python']
---

I’ve worked a lot with Excel spreadsheets on financial data. Among other things, I had to use working days. Excel provides a function (`WORKDAY`) to compute working days but holidays have to be provided manually and in France we have a lot of public holidays :-)---and they are not all anchored.

The python **Pandas** package comes from the financial analysis domain and in consequence, it provides a lot of features to manipulate time series data.

Here is how to create a custom calendar. With this french example you should be able to create your own.

```python
from pandas.tseries.holiday import AbstractHolidayCalendar, Holiday, EasterMonday, Easter
from pandas.tseries.offsets import Day, CustomBusinessDay

class FrBusinessCalendar(AbstractHolidayCalendar):
    """ Custom Holiday calendar for France based on
    https://en.wikipedia.org/wiki/Public_holidays_in_France
     - 1 January: New Year's Day
     - Moveable: Easter Monday 
       (Monday after Easter Sunday (one day after Easter Sunday)
     - 1 May: Labour Day
     - 8 May: Victory in Europe Day
     - Moveable Ascension Day 
       (Thursday, 39 days after Easter Sunday)
     - 14 July: Bastille Day
     - 15 August: Assumption of Mary to Heaven
     - 1 November: All Saints' Day
     - 11 November: Armistice Day
     - 25 December: Christmas Day
    """
    rules = [
        Holiday('New Years Day', month=1, day=1),
        EasterMonday,
        Holiday('Labour Day', month=5, day=1),
        Holiday('Victory in Europe Day', month=5, day=8),
        Holiday('Ascension Day', month=1, day=1, offset=[Easter(), Day(39)]),
        Holiday('Bastille Day', month=7, day=14),
        Holiday('Assumption of Mary to Heaven', month=8, day=15),
        Holiday('All Saints Day', month=11, day=1),
        Holiday('Armistice Day', month=11, day=11),
        Holiday('Christmas Day', month=12, day=25)
    ]
```

Here is how to get the public holidays whatever the year.

```python
import pandas as pd
from datetime import date

# Creating some boundaries
year = 2016
start = date(year, 1, 1)
end = start + pd.offsets.MonthEnd(12)

# Creating a custom calendar
cal = FrBusinessCalendar()
# Getting the holidays (off-days) between two dates
cal.holidays(start=start, end=end)

# DatetimeIndex(['2016-01-01', '2016-03-28', '2016-05-01', '2016-05-05',
#                '2016-05-08', '2016-07-14', '2016-08-15', '2016-11-01',
#                '2016-11-11', '2016-12-25'],
#               dtype='datetime64[ns]', freq=None)
And finally this is how to count the number of working days by month — it is very useful for a lot of things like building a rough schedule.
from pandas.tseries.offsets import CDay

# Creating a series of dates between the boundaries 
# by using the custom calendar
se = pd.bdate_range(start=start, 
                    end=end,
                    freq=CDay(calendar=cal)).to_series()
# Counting the number of working days by month
se.groupby(se.dt.month).count().head()

# 1    20
# 2    21
# 3    22
# 4    21
# 5    21
```