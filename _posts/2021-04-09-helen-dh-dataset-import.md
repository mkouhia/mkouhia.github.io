---
layout: post
title: Helen district heat generation dataset 2015-2020
---

I was notified yesterday that Helsinki district heating company Helen published their district heat
generation data from 2015-2020. This is very nice indeed, since generation data has been notoriously
hard to get as a researcher. I'm leaving here some notes on how to read the data in a Jupyter notebook.

See the announcement and link to dataset [here](https://www.helen.fi/helen-oy/vastuullisuus/ajankohtaista/avoindata),
in Finnish.

In this notebook, we read in the Helen DH dataset and fix their timestamps.

First, we request content from the server (link as of 2021-04-09) and inspect the data.


```python
import requests
import pandas as pd

from io import StringIO

HELEN_DATA_URL = 'https://www.helen.fi/globalassets/helen-oy/vastuullisuus/hki_dh_2015_2020_a.csv'

content = requests.get(HELEN_DATA_URL).text
content[0:100]
```




    'date_time;dh_MWh\r\n1.1.2015 1:00;936\r\n1.1.2015 2:00;924,2\r\n1.1.2015 3:00;926,3\r\n1.1.2015 4:00;942,1\r\n'



## Fixing timestamps

It seems that there are two columns, first including a naive timestamp and second the generation in MWh per hour. Let's read the data into a Pandas dataframe and then see to the timestamps


```python
df = pd.read_csv(StringIO(content), sep=';', decimal=',', parse_dates=['date_time'], dayfirst=True)\
    .set_index('date_time')
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dh_MWh</th>
    </tr>
    <tr>
      <th>date_time</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-01-01 01:00:00</th>
      <td>936.000</td>
    </tr>
    <tr>
      <th>2015-01-01 02:00:00</th>
      <td>924.200</td>
    </tr>
    <tr>
      <th>2015-01-01 03:00:00</th>
      <td>926.300</td>
    </tr>
    <tr>
      <th>2015-01-01 04:00:00</th>
      <td>942.100</td>
    </tr>
    <tr>
      <th>2015-01-01 05:00:00</th>
      <td>957.100</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-12-31 19:00:00</th>
      <td>1191.663</td>
    </tr>
    <tr>
      <th>2020-12-31 20:00:00</th>
      <td>1155.601</td>
    </tr>
    <tr>
      <th>2020-12-31 21:00:00</th>
      <td>1149.378</td>
    </tr>
    <tr>
      <th>2020-12-31 22:00:00</th>
      <td>1097.287</td>
    </tr>
    <tr>
      <th>2020-12-31 23:00:00</th>
      <td>1110.583</td>
    </tr>
  </tbody>
</table>
<p>52607 rows × 1 columns</p>
</div>



Now, with naive timestamps there is most likely confusion with the daytime changes. Let's two locations we know it happening, on 29th March 2015 03:00 and on 25th October 2020 at 4:00.


```python
df.loc[((df.index >='2015-03-29 00:00') & (df.index < '2015-03-29 06:00'))
        | ((df.index >= '2020-10-25 00:00') & (df.index < '2020-10-25 06:00'))]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dh_MWh</th>
    </tr>
    <tr>
      <th>date_time</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-03-29 00:00:00</th>
      <td>958.673</td>
    </tr>
    <tr>
      <th>2015-03-29 01:00:00</th>
      <td>930.965</td>
    </tr>
    <tr>
      <th>2015-03-29 02:00:00</th>
      <td>919.913</td>
    </tr>
    <tr>
      <th>2015-03-29 04:00:00</th>
      <td>913.885</td>
    </tr>
    <tr>
      <th>2015-03-29 05:00:00</th>
      <td>908.093</td>
    </tr>
    <tr>
      <th>2020-10-25 00:00:00</th>
      <td>871.322</td>
    </tr>
    <tr>
      <th>2020-10-25 01:00:00</th>
      <td>860.284</td>
    </tr>
    <tr>
      <th>2020-10-25 02:00:00</th>
      <td>848.808</td>
    </tr>
    <tr>
      <th>2020-10-25 03:00:00</th>
      <td>851.583</td>
    </tr>
    <tr>
      <th>2020-10-25 03:00:00</th>
      <td>842.317</td>
    </tr>
    <tr>
      <th>2020-10-25 04:00:00</th>
      <td>819.973</td>
    </tr>
    <tr>
      <th>2020-10-25 05:00:00</th>
      <td>820.491</td>
    </tr>
  </tbody>
</table>
</div>



So there it is, time hops two hours between 2015-03-29 02:00:00 and 2015-03-29 04:00:00,
and two similar timestamps exist at 2020-10-25 03:00:00. We will fix that next with `tz_localize`,
[see documentation](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DatetimeIndex.tz_localize.html).
Ambiguous parameter 'infer' attempts to sort out autumn daylight saving time transition based on appearance order.


```python
df.index = df.index.tz_localize(tz='Europe/Helsinki', ambiguous='infer')
```


```python
df.loc[((df.index >='2015-03-29 00:00') & (df.index < '2015-03-29 06:00'))
        | ((df.index >= '2020-10-25 00:00') & (df.index < '2020-10-25 06:00'))]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dh_MWh</th>
    </tr>
    <tr>
      <th>date_time</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-03-29 00:00:00+02:00</th>
      <td>958.673</td>
    </tr>
    <tr>
      <th>2015-03-29 01:00:00+02:00</th>
      <td>930.965</td>
    </tr>
    <tr>
      <th>2015-03-29 02:00:00+02:00</th>
      <td>919.913</td>
    </tr>
    <tr>
      <th>2015-03-29 04:00:00+03:00</th>
      <td>913.885</td>
    </tr>
    <tr>
      <th>2015-03-29 05:00:00+03:00</th>
      <td>908.093</td>
    </tr>
    <tr>
      <th>2020-10-25 00:00:00+03:00</th>
      <td>871.322</td>
    </tr>
    <tr>
      <th>2020-10-25 01:00:00+03:00</th>
      <td>860.284</td>
    </tr>
    <tr>
      <th>2020-10-25 02:00:00+03:00</th>
      <td>848.808</td>
    </tr>
    <tr>
      <th>2020-10-25 03:00:00+03:00</th>
      <td>851.583</td>
    </tr>
    <tr>
      <th>2020-10-25 03:00:00+02:00</th>
      <td>842.317</td>
    </tr>
    <tr>
      <th>2020-10-25 04:00:00+02:00</th>
      <td>819.973</td>
    </tr>
    <tr>
      <th>2020-10-25 05:00:00+02:00</th>
      <td>820.491</td>
    </tr>
  </tbody>
</table>
</div>



It seems correct now! Let's check.


```python
len(df.index.unique()) == len(df)
```




    True



Finally, we will check that there is one hour difference between all rows


```python
index_steps = df.index.to_frame().diff()
index_steps.loc[index_steps['date_time'] != pd.Timedelta('1 hours')]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date_time</th>
    </tr>
    <tr>
      <th>date_time</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-01-01 01:00:00+02:00</th>
      <td>NaT</td>
    </tr>
  </tbody>
</table>
</div>



# Basic plotting

Let's have a look at the data, plotted fully, and then a subplot


```python
df.plot(figsize=(20,6))
```




    <AxesSubplot:xlabel='date_time'>




    
![png](assets/images/2021-09-04-helen-dh-dataset-import-output_14_1.png)
    



```python
df.loc[(df.index > '2019-01-01') & (df.index < '2019-03-01')].plot(figsize=(20,6))
```




    <AxesSubplot:xlabel='date_time'>




    
![png](assets/images/2021-09-04-helen-dh-dataset-import-output_15_1.png)
    


## Next steps

The data seems to be in order. Next one would like to plot correlations with the temperature, build models _et cetera_. This will be left for another time.
