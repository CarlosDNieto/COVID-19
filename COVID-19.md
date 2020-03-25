```python
import git
import os
import pandas as pd
from glob import glob
import datetime as dt
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import plotly.express as px
import plotly.graph_objects as go
```

    C:\ProgramData\Anaconda3\lib\site-packages\statsmodels\tools\_testing.py:19: FutureWarning: pandas.util.testing is deprecated. Use the functions in the public API at pandas.testing instead.
      import pandas.util.testing as tm
    

# COVID-19

### Update the data source
The data source is a repository by the Johns Hopkins University Center for Systems Science and Engineering (JHU CSSE): [Johns Hopkins Github repository](https://github.com/CSSEGISandData/COVID-19).

We want to make the pull in the easiest and fastest possible way, so I make the next script for that:

Update the data sources pulling the [Johns Hopkins repository](https://github.com/CSSEGISandData/COVID-19).


```python
# update data source
git_local_path = 'C:\\Users\\nieto\\Google Drive\\COVID19\\CSSEGISandData_COVID-19'

git_repo_path = 'https://github.com/CSSEGISandData/COVID-19.git'

g = git.cmd.Git(git_local_path)

g.pull(git_repo_path)
```




    'Updating 8d7bdf1..0bba23f\nFast-forward\n .../time_series_covid19_confirmed_global.csv                         | 5 +++--\n .../csse_covid_19_time_series/time_series_covid19_deaths_global.csv  | 3 ++-\n 2 files changed, 5 insertions(+), 3 deletions(-)'




```python
# path to the data sources
path_time_series = 'CSSEGISandData_COVID-19\\csse_covid_19_data\\csse_covid_19_time_series'

# file names
file_confirmed = 'time_series_covid19_confirmed_global'
file_deaths = 'time_series_covid19_deaths_global'
file_recovered = 'time_series_19-covid-Recovered'

# read our data
Confirmed = pd.read_csv(path_time_series+'\\'+file_confirmed+'.csv')
Deaths = pd.read_csv(path_time_series+'\\'+file_deaths+'.csv')
Recovered = pd.read_csv(path_time_series+'\\'+file_recovered+'.csv')

# list of the countries that have confirmed cases
Countries = list(Confirmed['Country/Region'].unique())
Countries.sort()

# in this index, the columns of our sources begin to be dates
start_index = 1

# columns of hour data
Columns = Confirmed.columns.to_list()
```

### Functions


```python
# useful functions

def cols_to_datetime(dataframe,start_index=None,string=False):
    """Returns columns of time series in pandas datetimes
    
    Args:
        dataframe,
            dataframe of the time series.
        start_index,
            int can be null, in this index the date columns start to appear.
        string:
            bool default False, if True the desired output is in strings 
            
    Returns:
        list of strings or array with datetimes
    """
    
    cols = dataframe.columns.to_list()
    if start_index!=None:
        new_cols = pd.to_datetime(cols[start_index:])
        
        if string: # if output format is string 
            new_cols = list(map(str,new_cols))
            new_cols = [date.split()[0] for date in new_cols]
            cols[start_index:] = new_cols
            return cols
        
        else: # if output format is in pandas datetime
            cols[start_index:]=new_cols
            return cols
        
    else:
        new_cols = pd.to_datetime(cols)
        
        if string: # if output format is string 
            new_cols = list(map(str,new_cols))
            new_cols = [date.split()[0] for date in new_cols]
            cols = new_cols
            return cols
        
        else: # if output format is in pandas datetime
            cols=new_cols
            return cols
        
        
def transform_cols(dataframe,start_index=None):
    
    """Transform and correct the date strings columns in the data frame from 'm/d/y'
    to 'yyyy-mm-dd' to have homogeneous date columns.
    
    Args:
        dataframe,
            we are going to transform the columns of this pandas dataframe
        start_index,
            int can be null, in this index the date columns start to appear
        
    Returns:
        pandas dataframe
    """
    
    cols = dataframe.columns.to_list()
    new_cols = cols_to_datetime(dataframe,start_index=start_index,string=True)
    d = dict(zip(cols,new_cols))
    result = dataframe.rename(d,axis=1)
    return result

def get_last_update_date(timeseries, start_index=None, string=False):
    
    """Returns the last update date of a time series.
    
    Args:
        timeseries:
            DataFrame
            
        start_index,
            int can be null, in this index the date columns start to appear.
        
        string:
            bool default False, if True the desired output is in strings 
    Returns:
        str or datetime, last update date of the time series
    """
    
    if start_index!=None:
        dates = cols_to_datetime(timeseries,start_index)
        dates = dates[start_index:]
        last_update = max(dates)

        if string:
            last_update = str(last_update).split()[0]
            return last_update
        
        else:
            return last_update
        
    else:
        dates = cols_to_datetime(timeseries)
        last_update = max(dates)

        if string:
            last_update = str(last_update).split[0]
            return last_update
        
        else:
            return last_update

        
def get_ts_values(timeseries,start_index=None):
    """Returns the numeric values of a time series in a numpy array
    
    Args:
        timeseries:
            dataframe of timeseries
        
        start_index,
            int default null, in this index the date columns start to appear.
            
    Returns:
        numpy array
    """
    if start_index==None:
        dates = get_dates(timeseries)
        values = timeseries[dates].to_numpy()

        if len(values)==1:
            return values[0]

        else:
            return values
        
    else:
        dates = get_dates(timeseries,start_index=start_index)
        values = timeseries[dates].to_numpy()

        if len(values)==1:
            return values[0]

        else:
            return values
    
def get_dates(timeseries,start_index=None):
    """Returns the dates of the time series columns
    
    Args:
        timeseries:
            dataframe of timeseries
        
        start_index,
            int not null, in this index the date columns start to appear.
    
    Returns:
        list
    """
    if start_index!=None:
        dates = cols_to_datetime(dataframe=timeseries,
                                start_index=start_index,
                                string=True)
        dates = dates[start_index:]

        return dates
    
    else:
        dates = cols_to_datetime(dataframe=timeseries,
                                start_index=None,
                                string=True)
        
        return dates
#def get_actual_value(df):

def make_country_time_series(name,data):
    """This function helps to get the transformed (see: transform_cols function) time series grouped by 
    'Country/Region' the correspondent data.
    
    Note: also drops the 'Lat' and 'Long' columns.
    
    Args:
        name:
            str not null, name of the country with first capital letter
        
        data:
            dataframe not null, data of the time series. 
            It can be: Confirmed/Deaths/Recovered
            
    """
    df = data[data['Country/Region']==name].groupby('Country/Region').sum()
    df.drop(['Lat','Long'],axis=1,inplace=True)
    df = transform_cols(df)
    return df

def consolidate_time_series(array):
    """Consolidate different time series and drop all the columns with zeros in order
    to help us make better charts and have a better insight of the date of the first
    confirmed case.
    
    Args:
        array: 
        not null, list or array like object of the time series to consolidate.
        
    Returns:
        dataframe
        
    """
    df = pd.concat(array)
    df = df.replace({0:np.nan})
    df.dropna(axis=1,thresh=1,inplace=True)
    return df

def get_global(kind='Confirmed'):
    """Gets global time series of the desired kind.
    
    Args:
        kind:
            string default 'Confirmed',
            options: ['Confirmed','Deaths','Recovered']
    
    Returns:
        dataframe
        
    """
    if kind.lower()=='confirmed':
        df = Confirmed.groupby(lambda x: True).sum()
        new_cols = cols_to_datetime(df,start_index=2)[2:]
        df.drop(['Lat','Long'],inplace=True,axis=1)
        df.rename(dict(zip(df.columns.to_list(),new_cols)),axis=1,inplace=True)
        df.rename({True:'Global'}, axis=0,inplace=True)
        return df
    
    elif kind.lower()=='deaths':
        df = Deaths.groupby(lambda x: True).sum()
        new_cols = cols_to_datetime(df,start_index=2)[2:]
        df.drop(['Lat','Long'],inplace=True,axis=1)
        df.rename(dict(zip(df.columns.to_list(),new_cols)),axis=1,inplace=True)
        df.rename({True:'Global'}, axis=0,inplace=True)
        return df
    
    elif kind.lower()=='recovered':
        df = Recovered.groupby(lambda x: True).sum()
        new_cols = cols_to_datetime(df,start_index=2)[2:]
        df.drop(['Lat','Long'],inplace=True,axis=1)
        df.rename(dict(zip(df.columns.to_list(),new_cols)),axis=1,inplace=True)
        df.rename({True:'Global'}, axis=0,inplace=True)
        return df
    else:
        error = """Please enter a valid kind.
        valid kinds:
            ['Confirmed','Deaths','Recovered']
        """
        print(error)
        raise
        
def get_deltas(timeseries):
    """compute deltas in a time series dataframe
    
    Arg:
        timeseries not null
    """
    deltas = timeseries.diff(axis=1)/timeseries
    return deltas


def get_top_values(kind='Confirmed',top=10,ascending=True):
    """gets top values for a desired kind of information.
    
    Args:
        kind:
            string default 'Confirmed',
            options: ['Confirmed','Deaths','Recovered','Deltas-mean']
        top:
            int default 10, number of elements you want to get
        ascending:
            bool default True, if True then we are going to get the top
            in ascending order (minimun values), if false we get the
            maximun values.
    
    Returns:
        tuple of dataframes
            df1: 
                top10
            df2:
                complete df with all values
    """
    
    if kind.lower() == 'confirmed':
        vals = []
        for country in Countries:
            vals.append(Country(country).get_actual_confirmed()[0])
            
        df = pd.DataFrame({kind:vals}, index=Countries)
        df2 = df.sort_values(by=kind,axis=0,ascending=ascending).head(top)
        return df2,df
    
    elif kind.lower() == 'deaths':
        vals = []
        for country in Countries:
            vals.append(Country(country).get_actual_deaths()[0])
        
        df = pd.DataFrame({kind:vals}, index=Countries)
        df2 = df.sort_values(by=kind,axis=0,ascending=ascending).head(top)
        return df2,df
    
    elif kind.lower() == 'recovered':
        vals = []
        for country in Countries:
            vals.append(Country(country).get_actual_recovered()[0])
        
        df = pd.DataFrame({kind:vals}, index=Countries)
        df2 = df.sort_values(by=kind,axis=0,ascending=ascending).head(top)
        return df2,df

    elif kind.lower() == 'deltas-mean':
        vals=[]
        for country in Countries:
            vals.append(get_deltas(Country(country).get_confirmed_ts()).loc[country].mean())
        
        df = pd.DataFrame({kind:vals},index=Countries)
        df2 = df.sort_values(by=kind,axis=0,ascending=ascending).head(top)
        return df2,df
    
    else:
        error = """Please enter a valid kind.
        valid kinds:
            ['Confirmed','Deaths','Recovered']
        """
        print(error)
        raise
```

### Classes


```python
class Country:
    
    def __init__(
        self,
        name,
        time_series_confirmed=None,
        time_series_deaths=None,
        time_series_rec=None
    ):
        
        if not name in Countries:
            error_name='''Invalid country name, please validate your input
or capitalize the first letter of the country name.
            \nExample: china -> China'
            For United States use 'US'
            '''
            print(error_name)
            raise
            
        else:
            self.name = name
            self.time_series_confirmed = make_country_time_series(self.name,Confirmed)
            self.time_series_deaths = make_country_time_series(self.name,Deaths)
            self.time_series_rec = make_country_time_series(self.name,Recovered)
     
    
    def get_confirmed_ts(self,):
        ts = self.time_series_confirmed
        #drop_zeros(ts)
        return self.time_series_confirmed
    
    def get_death_ts(self):
        return self.time_series_deaths
    
    def get_rec_ts(self):
        return self.time_series_rec
    
    def get_actual_confirmed(self):
        #print('Actual confirmed cases in '+self.name+':')
        return self.get_actual_value('confirmed')
    
    def get_actual_deaths(self):
        #print('Actual deaths number in '+self.name+':')
        return self.get_actual_value('deaths')
    
    def get_actual_recovered(self):
        #print('Actual recovered number in '+self.name+':')
        return self.get_actual_value('rec')
    
    def get_actual_value(self,value='confirmed'):
        """returns the actual value of a desired time series.
        
        Args:
            value:
                str, desired value we desire
                'confirmed' for confirmed
                'deaths' for deaths
                'rec' for recovered
        Returns:
            int, number of confirmed/deaths/recovered
        """
        valid_values = ['confirmed','deaths','rec']
        dfs = [self.time_series_confirmed,self.time_series_deaths,self.time_series_rec]
        d = dict(zip(valid_values,dfs))
        
        if value.lower() in valid_values:
            ts = d[value.lower()]
            ts = ts.groupby('Country/Region').sum()
            last_update = get_last_update_date(ts,start_index=start_index,string=True)
            return ts[last_update]
        
        else:
            error = """ Please enter a valid value:
            valid values:
                'confirmed' for confirmed
                'deaths' for deaths
                'rec' for recovered
            """
            print(error)
            raise
            
    def get_mortality_rates(self):
        """
        """
        confirmed = get_ts_values(self.time_series_confirmed.groupby('Country/Region').sum())
        deaths = get_ts_values(self.time_series_deaths.groupby('Country/Region').sum())
        mortality_rates = deaths/confirmed
        ix = get_dates(self.time_series_confirmed.groupby('Country/Region').sum())
        mortality_rates = pd.Series(data=mortality_rates,index=ix)
        return mortality_rates
    
    def get_actual_mortality_rate(self):
        """
        """
        return self.get_mortality_rates()[-1]
```

### Get actual values of a country

> **Note:** if you want to see the list of countries available run a code cell with ``Countries``

**Get actual confirmed** 


```python
Country('India').get_actual_confirmed()
```




    Country/Region
    India    536
    Name: 2020-03-24, dtype: int64



**Get actual deaths cases**


```python
Country('San Marino').get_actual_deaths()
```




    Country/Region
    San Marino    21
    Name: 2020-03-24, dtype: int64



**Get actual recovered cases**


```python
Country('San Marino').get_actual_recovered()
```




    Country/Region
    San Marino    4.0
    Name: 2020-03-23, dtype: float64



**Get actual mortality rate**


```python
Country('San Marino').get_actual_mortality_rate()
```

    C:\ProgramData\Anaconda3\lib\site-packages\ipykernel_launcher.py:87: RuntimeWarning:
    
    invalid value encountered in true_divide
    
    




    0.11229946524064172



### Get and make the curve of the time series of a country

First, **we can display the DataFrame with the time series of a Country**


```python
# ts for TimeSeries
ts = Country('China').get_confirmed_ts()
ts
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
      <th>2020-01-22</th>
      <th>2020-01-23</th>
      <th>2020-01-24</th>
      <th>2020-01-25</th>
      <th>2020-01-26</th>
      <th>2020-01-27</th>
      <th>2020-01-28</th>
      <th>2020-01-29</th>
      <th>2020-01-30</th>
      <th>2020-01-31</th>
      <th>...</th>
      <th>2020-03-15</th>
      <th>2020-03-16</th>
      <th>2020-03-17</th>
      <th>2020-03-18</th>
      <th>2020-03-19</th>
      <th>2020-03-20</th>
      <th>2020-03-21</th>
      <th>2020-03-22</th>
      <th>2020-03-23</th>
      <th>2020-03-24</th>
    </tr>
    <tr>
      <th>Country/Region</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>China</th>
      <td>548</td>
      <td>643</td>
      <td>920</td>
      <td>1406</td>
      <td>2075</td>
      <td>2877</td>
      <td>5509</td>
      <td>6087</td>
      <td>8141</td>
      <td>9802</td>
      <td>...</td>
      <td>81003</td>
      <td>81033</td>
      <td>81058</td>
      <td>81102</td>
      <td>81156</td>
      <td>81250</td>
      <td>81305</td>
      <td>81435</td>
      <td>81498</td>
      <td>81591</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 63 columns</p>
</div>




```python
y = get_ts_values(ts)
x = ts.columns.to_list()

fig = go.Figure(
    go.Scatter(
        name='China',
        mode='lines+markers',
        x=x,
        y=y,
        line=dict(
            color='firebrick'
        )
    )
)

# layout
fig.update_layout(
    title='Confirmed cases in China',
    xaxis_title="Date",
    yaxis_title=None,
    font=dict(
        family="Arial, bold",
        size=14,
        #color="auto"
    )
)

fig.show()
```


        <script type="text/javascript">
        window.PlotlyConfig = {MathJaxConfig: 'local'};
        if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
        if (typeof require !== 'undefined') {
        require.undef("plotly");
        define('plotly', function(require, exports, module) {
            /**
* plotly.js v1.51.2
* Copyright 2012-2019, Plotly, Inc.
* All rights reserved.
* Licensed under the MIT license
*/
        });
        require(['plotly'], function(Plotly) {
            window._Plotly = Plotly;
        });
        }
        </script>




<div>


            <div id="31847cf8-7204-4916-bc4f-eb32cbfd7bba" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("31847cf8-7204-4916-bc4f-eb32cbfd7bba")) {
                    Plotly.newPlot(
                        '31847cf8-7204-4916-bc4f-eb32cbfd7bba',
                        [{"line": {"color": "firebrick"}, "mode": "lines+markers", "name": "China", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [548, 643, 920, 1406, 2075, 2877, 5509, 6087, 8141, 9802, 11891, 16630, 19716, 23707, 27440, 30587, 34110, 36814, 39829, 42354, 44386, 44759, 59895, 66358, 68413, 70513, 72434, 74211, 74619, 75077, 75550, 77001, 77022, 77241, 77754, 78166, 78600, 78928, 79356, 79932, 80136, 80261, 80386, 80537, 80690, 80770, 80823, 80860, 80887, 80921, 80932, 80945, 80977, 81003, 81033, 81058, 81102, 81156, 81250, 81305, 81435, 81498, 81591]}],
                        {"font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Confirmed cases in China"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('31847cf8-7204-4916-bc4f-eb32cbfd7bba');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


### Compare different time series


```python
# consolidate 3 time series
# see func 'consolidate_time_series(array)'
ts1 = Country('Mexico').get_confirmed_ts()
ts2 = Country('Brazil').get_confirmed_ts()
ts3 = Country('Chile').get_confirmed_ts()
conso = consolidate_time_series([ts1,ts2,ts3])

y1 = conso.loc['Mexico']
y2 = conso.loc['Brazil']
y3 = conso.loc['Chile']
x = conso.columns.to_list()

fig = go.Figure()

fig.add_trace(go.Scatter(name='Brazil',x=x,y=y2,mode='lines+markers',line=dict(color='firebrick'),
                        text=y2))
fig.add_trace(go.Scatter(name='Mexico',x=x,y=y1,mode='lines+markers',line=dict(color='steelblue',width=2)))
fig.add_trace(go.Scatter(name='Chile',x=x,y=y3,mode='lines+markers',line=dict(color='darkgreen')))

# layout
fig.update_layout(
    title='Confirmed cases in Mexico, Brazil and Chile',
    xaxis_title="Date",
    yaxis_title="Confirmed Cases",
    font=dict(
        family="Arial, bold",
        size=14,
        #color="auto"
    ),
    annotations=[
        dict(
            x=x[-1],
            y=y2[-1],
            xref="x",
            yref="y",
            text='{}'.format(int(y2[-1])),
            showarrow=True,
            arrowhead=7
        ),
        dict(
            x=x[-1],
            y=y1[-1],
            xref="x",
            yref="y",
            text='{}'.format(int(y1[-1])),
            showarrow=True,
            arrowhead=7
        ),
        dict(
            x=x[-1],
            y=y3[-1],
            xref="x",
            yref="y",
            text='{}'.format(int(y3[-1])),
            showarrow=True,
            arrowhead=7
        )
    ]
)
fig.show()
```


<div>


            <div id="e9380a86-7289-4ea3-b8da-bff0a3621c06" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("e9380a86-7289-4ea3-b8da-bff0a3621c06")) {
                    Plotly.newPlot(
                        'e9380a86-7289-4ea3-b8da-bff0a3621c06',
                        [{"line": {"color": "firebrick"}, "mode": "lines+markers", "name": "Brazil", "text": [1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 4.0, 4.0, 13.0, 13.0, 20.0, 25.0, 31.0, 38.0, 52.0, 151.0, 151.0, 162.0, 200.0, 321.0, 372.0, 621.0, 793.0, 1021.0, 1546.0, 1924.0, 2247.0], "type": "scatter", "x": ["2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 4.0, 4.0, 13.0, 13.0, 20.0, 25.0, 31.0, 38.0, 52.0, 151.0, 151.0, 162.0, 200.0, 321.0, 372.0, 621.0, 793.0, 1021.0, 1546.0, 1924.0, 2247.0]}, {"line": {"color": "steelblue", "width": 2}, "mode": "lines+markers", "name": "Mexico", "type": "scatter", "x": ["2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, 1.0, 4.0, 5.0, 5.0, 5.0, 5.0, 5.0, 6.0, 6.0, 7.0, 7.0, 7.0, 8.0, 12.0, 12.0, 26.0, 41.0, 53.0, 82.0, 93.0, 118.0, 164.0, 203.0, 251.0, 316.0, 367.0]}, {"line": {"color": "darkgreen"}, "mode": "lines+markers", "name": "Chile", "type": "scatter", "x": ["2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, 1.0, 1.0, 4.0, 4.0, 4.0, 8.0, 8.0, 13.0, 23.0, 23.0, 43.0, 61.0, 74.0, 155.0, 201.0, 238.0, 238.0, 434.0, 537.0, 632.0, 746.0, 922.0]}],
                        {"annotations": [{"arrowhead": 7, "showarrow": true, "text": "2247", "x": "2020-03-24", "xref": "x", "y": 2247.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "367", "x": "2020-03-24", "xref": "x", "y": 367.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "922", "x": "2020-03-24", "xref": "x", "y": 922.0, "yref": "y"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Confirmed cases in Mexico, Brazil and Chile"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('e9380a86-7289-4ea3-b8da-bff0a3621c06');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


**We can automate the annotations with the code below**

As we saw in the examples above, we can put annotations in the chart, but note in the last code cell we actually coded a lot of lines only to annotate the last values of the time series.

So in the code cell below is an example of how to write annotations in the chart with a **``for loop``**.


```python
# consolidate 3 time series
# see func 'consolidate_time_series(array)'
ts1 = Country('US').get_confirmed_ts()
ts2 = Country('China').get_confirmed_ts()
ts3 = Country('Iran').get_confirmed_ts()
ts4 = Country('Italy').get_confirmed_ts()
conso = consolidate_time_series([ts1,ts2,ts3,ts4])

# get values
y1 = conso.loc['US']
y2 = conso.loc['China']
y3 = conso.loc['Iran']
y4 = conso.loc['Italy']
y_list = [y1,y2,y3,y4]
x = conso.columns.to_list()

fig = go.Figure()

# add lines
fig.add_trace(go.Scatter(name='China',x=x,y=y2,mode='lines+markers',line=dict(color='firebrick'),
                        text=y2))
fig.add_trace(go.Scatter(name='US',x=x,y=y1,mode='lines+markers',line=dict(color='steelblue',width=2)))
fig.add_trace(go.Scatter(name='Iran',x=x,y=y3,mode='lines+markers',line=dict(color='darkgreen',width=2)))
fig.add_trace(go.Scatter(name='Italy',x=x,y=y4,mode='lines+markers',line=dict(width=2)))

# automate annotations
annotations = []
for yi in y_list:
    annotations.append(
        dict(
            x=x[-1],
            y=yi[-1],
            xref="x",
            yref="y",
            text='{}'.format(int(yi[-1])),
            showarrow=True,
            arrowhead=7
        )
    )
    
# layout
fig.update_layout(
    title='Confirmed cases in US, China and Iran',
    xaxis_title="Date",
    yaxis_title="Confirmed Cases",
    font=dict(
        family="Arial, bold",
        size=14,
        #color="auto"
    ),
    annotations=annotations
)

fig.show()
```


<div>


            <div id="d84c5101-8dbe-4ac4-8a08-8c6f34515a08" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("d84c5101-8dbe-4ac4-8a08-8c6f34515a08")) {
                    Plotly.newPlot(
                        'd84c5101-8dbe-4ac4-8a08-8c6f34515a08',
                        [{"line": {"color": "firebrick"}, "mode": "lines+markers", "name": "China", "text": [548.0, 643.0, 920.0, 1406.0, 2075.0, 2877.0, 5509.0, 6087.0, 8141.0, 9802.0, 11891.0, 16630.0, 19716.0, 23707.0, 27440.0, 30587.0, 34110.0, 36814.0, 39829.0, 42354.0, 44386.0, 44759.0, 59895.0, 66358.0, 68413.0, 70513.0, 72434.0, 74211.0, 74619.0, 75077.0, 75550.0, 77001.0, 77022.0, 77241.0, 77754.0, 78166.0, 78600.0, 78928.0, 79356.0, 79932.0, 80136.0, 80261.0, 80386.0, 80537.0, 80690.0, 80770.0, 80823.0, 80860.0, 80887.0, 80921.0, 80932.0, 80945.0, 80977.0, 81003.0, 81033.0, 81058.0, 81102.0, 81156.0, 81250.0, 81305.0, 81435.0, 81498.0, 81591.0], "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [548.0, 643.0, 920.0, 1406.0, 2075.0, 2877.0, 5509.0, 6087.0, 8141.0, 9802.0, 11891.0, 16630.0, 19716.0, 23707.0, 27440.0, 30587.0, 34110.0, 36814.0, 39829.0, 42354.0, 44386.0, 44759.0, 59895.0, 66358.0, 68413.0, 70513.0, 72434.0, 74211.0, 74619.0, 75077.0, 75550.0, 77001.0, 77022.0, 77241.0, 77754.0, 78166.0, 78600.0, 78928.0, 79356.0, 79932.0, 80136.0, 80261.0, 80386.0, 80537.0, 80690.0, 80770.0, 80823.0, 80860.0, 80887.0, 80921.0, 80932.0, 80945.0, 80977.0, 81003.0, 81033.0, 81058.0, 81102.0, 81156.0, 81250.0, 81305.0, 81435.0, 81498.0, 81591.0]}, {"line": {"color": "steelblue", "width": 2}, "mode": "lines+markers", "name": "US", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [1.0, 1.0, 2.0, 2.0, 5.0, 5.0, 5.0, 5.0, 5.0, 7.0, 8.0, 8.0, 11.0, 11.0, 11.0, 11.0, 11.0, 11.0, 11.0, 11.0, 12.0, 12.0, 13.0, 13.0, 13.0, 13.0, 13.0, 13.0, 13.0, 13.0, 15.0, 15.0, 15.0, 51.0, 51.0, 57.0, 58.0, 60.0, 68.0, 74.0, 98.0, 118.0, 149.0, 217.0, 262.0, 402.0, 518.0, 583.0, 959.0, 1281.0, 1663.0, 2179.0, 2727.0, 3499.0, 4632.0, 6421.0, 7783.0, 13677.0, 19100.0, 25489.0, 33276.0, 43847.0, 53740.0]}, {"line": {"color": "darkgreen", "width": 2}, "mode": "lines+markers", "name": "Iran", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 2.0, 5.0, 18.0, 28.0, 43.0, 61.0, 95.0, 139.0, 245.0, 388.0, 593.0, 978.0, 1501.0, 2336.0, 2922.0, 3513.0, 4747.0, 5823.0, 6566.0, 7161.0, 8042.0, 9000.0, 10075.0, 11364.0, 12729.0, 13938.0, 14991.0, 16169.0, 17361.0, 18407.0, 19644.0, 20610.0, 21638.0, 23049.0, 24811.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Italy", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 20.0, 62.0, 155.0, 229.0, 322.0, 453.0, 655.0, 888.0, 1128.0, 1694.0, 2036.0, 2502.0, 3089.0, 3858.0, 4636.0, 5883.0, 7375.0, 9172.0, 10149.0, 12462.0, 12462.0, 17660.0, 21157.0, 24747.0, 27980.0, 31506.0, 35713.0, 41035.0, 47021.0, 53578.0, 59138.0, 63927.0, 69176.0]}],
                        {"annotations": [{"arrowhead": 7, "showarrow": true, "text": "53740", "x": "2020-03-24", "xref": "x", "y": 53740.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "81591", "x": "2020-03-24", "xref": "x", "y": 81591.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "24811", "x": "2020-03-24", "xref": "x", "y": 24811.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "69176", "x": "2020-03-24", "xref": "x", "y": 69176.0, "yref": "y"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Confirmed cases in US, China and Iran"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('d84c5101-8dbe-4ac4-8a08-8c6f34515a08');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


**What if we liked to display a lot of time series in one chart?**

We likely code a lot of code lines as in the examples above, but if we manage to make a function to do all of that code for us, will be awesome!

In the cell below is a function that automates the plot's of multiple time series.


```python
def graph_lines(countries,kind='Confirmed'):
    """Graph a line chart for every time serie of the countries passed in.
    
    Args:
        countries:
            list or array like object, list of countries to plot time series
        
        kind:
            str default 'Confirmed', kind of data we want to plot.
    
    Returns:
        nothing, it displays the plot and save it to a local path called figs/file_name.png
    """
    
    # store every time series for each country
    # for the kind specified
    ts = []
    if kind.lower() == 'confirmed':
        for country in countries:
            ts.append(Country(country).get_confirmed_ts())
    
    elif kind.lower() == 'deaths':
            for country in countries:
                ts.append(Country(country).get_death_ts())
                
    elif kind.lower() == 'recovered':
            for country in countries:
                ts.append(Country(country).get_rec_ts())
    
    else:
        error ="""kind not valid, please enter one of this:
        
        valid kinds:
            ['Confirmed', 'Deaths', 'Recovered']
            
        """
        print(error)
        raise
    
    # consolidate the time series in a dataframe
    conso = consolidate_time_series(ts)
    
    # get x and y values for the plot
    x = conso.columns.to_list()
    y = []
    for country in conso.index.to_list():
        y.append(conso.loc[country])
    
    # initialize plotly figure and
    # add the lines for each country
    fig = go.Figure()
    for i in range(len(countries)):
        fig.add_trace(go.Scatter(name=countries[i],x=x,y=y[i],mode='lines+markers',line=dict(width=2)))
    
    # store the annotations
    annotations = []
    for yi in y:
        annotations.append(
            dict(
                x=x[-1],
                y=yi[-1],
                xref="x",
                yref="y",
                text='{}'.format(int(yi[-1])),
                showarrow=True,
                arrowhead=7
            )
        )
    
    # store strings for title and file name
    strings = [] # for title
    strings2 = [] # for file name
    if len(countries)!=1:
        for i in range(len(countries)):
            if i==(len(countries)-1):
                strings.append('and '+countries[i])
                strings2.append(countries[i])

            else:
                strings.append(countries[i]+', ')
                strings2.append(countries[i])
    
    else:
        strings.append(countries[0])
        strings2.append(countries[0])
    
    # title for layout
    title = 'Confirmed cases in '+ ''.join(strings)
    
    # layout
    fig.update_layout(
        title=title,
        xaxis_title="Date",
        yaxis_title="Confirmed Cases",
        font=dict(
            family="Arial, bold",
            size=14,
            #color="auto"
        ),
        annotations=annotations
    )
    
    # filename for saving plot
    file_name = 'lines_'+kind+'_'.join(strings2)+'.png'
    
    fig.write_image("figs/"+file_name)
    fig.show()
```


```python
graph_lines(['US','Spain','China','Italy'],kind='deaths')
```


<div>


            <div id="412b1efe-4642-4a6e-a2b4-0ca753c8ea4f" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("412b1efe-4642-4a6e-a2b4-0ca753c8ea4f")) {
                    Plotly.newPlot(
                        '412b1efe-4642-4a6e-a2b4-0ca753c8ea4f',
                        [{"line": {"width": 2}, "mode": "lines+markers", "name": "US", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 1.0, 1.0, 6.0, 7.0, 11.0, 12.0, 14.0, 17.0, 21.0, 22.0, 28.0, 36.0, 40.0, 47.0, 54.0, 63.0, 85.0, 108.0, 118.0, 200.0, 244.0, 307.0, 417.0, 557.0, 706.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Spain", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 1.0, 2.0, 3.0, 5.0, 10.0, 17.0, 28.0, 35.0, 54.0, 55.0, 133.0, 195.0, 289.0, 342.0, 533.0, 623.0, 830.0, 1043.0, 1375.0, 1772.0, 2311.0, 2808.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "China", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [17.0, 18.0, 26.0, 42.0, 56.0, 82.0, 131.0, 133.0, 171.0, 213.0, 259.0, 361.0, 425.0, 491.0, 563.0, 633.0, 718.0, 805.0, 905.0, 1012.0, 1112.0, 1117.0, 1369.0, 1521.0, 1663.0, 1766.0, 1864.0, 2003.0, 2116.0, 2238.0, 2238.0, 2443.0, 2445.0, 2595.0, 2665.0, 2717.0, 2746.0, 2790.0, 2837.0, 2872.0, 2914.0, 2947.0, 2983.0, 3015.0, 3044.0, 3072.0, 3100.0, 3123.0, 3139.0, 3161.0, 3172.0, 3180.0, 3193.0, 3203.0, 3217.0, 3230.0, 3241.0, 3249.0, 3253.0, 3259.0, 3274.0, 3274.0, 3281.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Italy", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 1.0, 2.0, 3.0, 7.0, 10.0, 12.0, 17.0, 21.0, 29.0, 34.0, 52.0, 79.0, 107.0, 148.0, 197.0, 233.0, 366.0, 463.0, 631.0, 827.0, 827.0, 1266.0, 1441.0, 1809.0, 2158.0, 2503.0, 2978.0, 3405.0, 4032.0, 4825.0, 5476.0, 6077.0, 6820.0]}],
                        {"annotations": [{"arrowhead": 7, "showarrow": true, "text": "706", "x": "2020-03-24", "xref": "x", "y": 706.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "2808", "x": "2020-03-24", "xref": "x", "y": 2808.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "3281", "x": "2020-03-24", "xref": "x", "y": 3281.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "6820", "x": "2020-03-24", "xref": "x", "y": 6820.0, "yref": "y"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Confirmed cases in US, Spain, China, and Italy"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('412b1efe-4642-4a6e-a2b4-0ca753c8ea4f');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>



```python
graph_lines(['US','Italy','Iran','Germany','China','Canada','Poland','Spain','Netherlands', 'United Kingdom'])
```


<div>


            <div id="016562b8-42b0-4158-9b9c-07ffbc8f630a" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("016562b8-42b0-4158-9b9c-07ffbc8f630a")) {
                    Plotly.newPlot(
                        '016562b8-42b0-4158-9b9c-07ffbc8f630a',
                        [{"line": {"width": 2}, "mode": "lines+markers", "name": "US", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [1.0, 1.0, 2.0, 2.0, 5.0, 5.0, 5.0, 5.0, 5.0, 7.0, 8.0, 8.0, 11.0, 11.0, 11.0, 11.0, 11.0, 11.0, 11.0, 11.0, 12.0, 12.0, 13.0, 13.0, 13.0, 13.0, 13.0, 13.0, 13.0, 13.0, 15.0, 15.0, 15.0, 51.0, 51.0, 57.0, 58.0, 60.0, 68.0, 74.0, 98.0, 118.0, 149.0, 217.0, 262.0, 402.0, 518.0, 583.0, 959.0, 1281.0, 1663.0, 2179.0, 2727.0, 3499.0, 4632.0, 6421.0, 7783.0, 13677.0, 19100.0, 25489.0, 33276.0, 43847.0, 53740.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Italy", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 20.0, 62.0, 155.0, 229.0, 322.0, 453.0, 655.0, 888.0, 1128.0, 1694.0, 2036.0, 2502.0, 3089.0, 3858.0, 4636.0, 5883.0, 7375.0, 9172.0, 10149.0, 12462.0, 12462.0, 17660.0, 21157.0, 24747.0, 27980.0, 31506.0, 35713.0, 41035.0, 47021.0, 53578.0, 59138.0, 63927.0, 69176.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Iran", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 2.0, 5.0, 18.0, 28.0, 43.0, 61.0, 95.0, 139.0, 245.0, 388.0, 593.0, 978.0, 1501.0, 2336.0, 2922.0, 3513.0, 4747.0, 5823.0, 6566.0, 7161.0, 8042.0, 9000.0, 10075.0, 11364.0, 12729.0, 13938.0, 14991.0, 16169.0, 17361.0, 18407.0, 19644.0, 20610.0, 21638.0, 23049.0, 24811.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Germany", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, 1.0, 4.0, 4.0, 4.0, 5.0, 8.0, 10.0, 12.0, 12.0, 12.0, 12.0, 13.0, 13.0, 14.0, 14.0, 16.0, 16.0, 16.0, 16.0, 16.0, 16.0, 16.0, 16.0, 16.0, 16.0, 16.0, 16.0, 16.0, 16.0, 17.0, 27.0, 46.0, 48.0, 79.0, 130.0, 159.0, 196.0, 262.0, 482.0, 670.0, 799.0, 1040.0, 1176.0, 1457.0, 1908.0, 2078.0, 3675.0, 4585.0, 5795.0, 7272.0, 9257.0, 12327.0, 15320.0, 19848.0, 22213.0, 24873.0, 29056.0, 32986.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "China", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [548.0, 643.0, 920.0, 1406.0, 2075.0, 2877.0, 5509.0, 6087.0, 8141.0, 9802.0, 11891.0, 16630.0, 19716.0, 23707.0, 27440.0, 30587.0, 34110.0, 36814.0, 39829.0, 42354.0, 44386.0, 44759.0, 59895.0, 66358.0, 68413.0, 70513.0, 72434.0, 74211.0, 74619.0, 75077.0, 75550.0, 77001.0, 77022.0, 77241.0, 77754.0, 78166.0, 78600.0, 78928.0, 79356.0, 79932.0, 80136.0, 80261.0, 80386.0, 80537.0, 80690.0, 80770.0, 80823.0, 80860.0, 80887.0, 80921.0, 80932.0, 80945.0, 80977.0, 81003.0, 81033.0, 81058.0, 81102.0, 81156.0, 81250.0, 81305.0, 81435.0, 81498.0, 81591.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Canada", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, 1.0, 1.0, 2.0, 2.0, 2.0, 4.0, 4.0, 4.0, 4.0, 4.0, 5.0, 5.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 8.0, 8.0, 8.0, 8.0, 9.0, 9.0, 9.0, 10.0, 11.0, 11.0, 13.0, 14.0, 20.0, 24.0, 27.0, 30.0, 33.0, 37.0, 49.0, 54.0, 64.0, 77.0, 79.0, 108.0, 117.0, 193.0, 198.0, 252.0, 415.0, 478.0, 657.0, 800.0, 943.0, 1277.0, 1469.0, 2088.0, 2790.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Poland", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 1.0, 1.0, 5.0, 5.0, 11.0, 16.0, 22.0, 31.0, 49.0, 68.0, 103.0, 119.0, 177.0, 238.0, 251.0, 355.0, 425.0, 536.0, 634.0, 749.0, 901.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Spain", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, null, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 6.0, 13.0, 15.0, 32.0, 45.0, 84.0, 120.0, 165.0, 222.0, 259.0, 400.0, 500.0, 673.0, 1073.0, 1695.0, 2277.0, 2277.0, 5232.0, 6391.0, 7798.0, 9942.0, 11748.0, 13910.0, 17963.0, 20410.0, 25374.0, 28768.0, 35136.0, 39885.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Netherlands", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 1.0, 1.0, 6.0, 10.0, 18.0, 24.0, 38.0, 82.0, 128.0, 188.0, 265.0, 321.0, 382.0, 503.0, 503.0, 806.0, 962.0, 1138.0, 1416.0, 1711.0, 2058.0, 2467.0, 3003.0, 3640.0, 4217.0, 4764.0, 5580.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "United Kingdom", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 8.0, 8.0, 9.0, 9.0, 9.0, 9.0, 9.0, 9.0, 9.0, 9.0, 9.0, 9.0, 9.0, 9.0, 13.0, 13.0, 13.0, 15.0, 20.0, 23.0, 36.0, 40.0, 51.0, 86.0, 116.0, 164.0, 207.0, 274.0, 322.0, 384.0, 459.0, 459.0, 802.0, 1144.0, 1145.0, 1551.0, 1960.0, 2642.0, 2716.0, 4014.0, 5067.0, 5745.0, 6726.0, 8164.0]}],
                        {"annotations": [{"arrowhead": 7, "showarrow": true, "text": "53740", "x": "2020-03-24", "xref": "x", "y": 53740.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "69176", "x": "2020-03-24", "xref": "x", "y": 69176.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "24811", "x": "2020-03-24", "xref": "x", "y": 24811.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "32986", "x": "2020-03-24", "xref": "x", "y": 32986.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "81591", "x": "2020-03-24", "xref": "x", "y": 81591.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "2790", "x": "2020-03-24", "xref": "x", "y": 2790.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "901", "x": "2020-03-24", "xref": "x", "y": 901.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "39885", "x": "2020-03-24", "xref": "x", "y": 39885.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "5580", "x": "2020-03-24", "xref": "x", "y": 5580.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "8164", "x": "2020-03-24", "xref": "x", "y": 8164.0, "yref": "y"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Confirmed cases in US, Italy, Iran, Germany, China, Canada, Poland, Spain, Netherlands, and United Kingdom"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('016562b8-42b0-4158-9b9c-07ffbc8f630a');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>



```python
# we also can plot only 1 time series
graph_lines(['Mexico'])
```


<div>


            <div id="b36aaebe-46b3-4338-97e9-f0510e58a5a8" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("b36aaebe-46b3-4338-97e9-f0510e58a5a8")) {
                    Plotly.newPlot(
                        'b36aaebe-46b3-4338-97e9-f0510e58a5a8',
                        [{"line": {"width": 2}, "mode": "lines+markers", "name": "Mexico", "type": "scatter", "x": ["2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [1.0, 4.0, 5.0, 5.0, 5.0, 5.0, 5.0, 6.0, 6.0, 7.0, 7.0, 7.0, 8.0, 12.0, 12.0, 26.0, 41.0, 53.0, 82.0, 93.0, 118.0, 164.0, 203.0, 251.0, 316.0, 367.0]}],
                        {"annotations": [{"arrowhead": 7, "showarrow": true, "text": "367", "x": "2020-03-24", "xref": "x", "y": 367.0, "yref": "y"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Confirmed cases in Mexico"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('b36aaebe-46b3-4338-97e9-f0510e58a5a8');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


### Confirmed cases

All this plots that we have done are from the confirmed cases, but what are the confirmed cases?

**Confirmed cases are all the cumulative positive coronavirus cases that a country oficialy reports**

We can use de method ``get_confirmed_ts()`` in the class ``Country()`` to get the confirmed cases time series of a country, or you can simply use the method ``get_actual_confirmed()`` to get the confirmed cases at the last update date.


```python
# we can get the global time series
# see get_global in functions.
g_confirmed = get_global('Confirmed')
g_confirmed
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
      <th>2020-01-22</th>
      <th>2020-01-23</th>
      <th>2020-01-24</th>
      <th>2020-01-25</th>
      <th>2020-01-26</th>
      <th>2020-01-27</th>
      <th>2020-01-28</th>
      <th>2020-01-29</th>
      <th>2020-01-30</th>
      <th>2020-01-31</th>
      <th>...</th>
      <th>2020-03-15</th>
      <th>2020-03-16</th>
      <th>2020-03-17</th>
      <th>2020-03-18</th>
      <th>2020-03-19</th>
      <th>2020-03-20</th>
      <th>2020-03-21</th>
      <th>2020-03-22</th>
      <th>2020-03-23</th>
      <th>2020-03-24</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Global</th>
      <td>555</td>
      <td>654</td>
      <td>941</td>
      <td>1434</td>
      <td>2118</td>
      <td>2927</td>
      <td>5578</td>
      <td>6166</td>
      <td>8234</td>
      <td>9927</td>
      <td>...</td>
      <td>167454</td>
      <td>181574</td>
      <td>197102</td>
      <td>214821</td>
      <td>242500</td>
      <td>272035</td>
      <td>304396</td>
      <td>336953</td>
      <td>378235</td>
      <td>418045</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 63 columns</p>
</div>




```python
# or we can get the actual global confirmed cases
last_update_date = cols_to_datetime(g_confirmed).max()
g_confirmed[last_update_date]
```




    Global    418045
    Name: 2020-03-24 00:00:00, dtype: int64




```python
# we can plot thi global time series

y = g_confirmed.to_numpy()[0]
x = g_confirmed.columns.to_list()

fig = go.Figure(
    go.Scatter(
        name='Global',
        mode='lines+markers',
        x=x,
        y=y,
        line=dict(
            color='firebrick'
        )
    )
)

fig.update_layout(
    title='Global Confirmed Cases',
    xaxis_title="Date",
    yaxis_title="Confirmed Cases",
    font=dict(
        family="Arial, bold",
        size=14,
        #color="auto"
    ),
    annotations=[
        dict(
            x=x[-1],
            y=y[-1],
            xref="x",
            yref="y",
            text='{}'.format(int(y[-1])),
            showarrow=True,
            arrowhead=7
        )
    ]
)
fig.write_image("figs/line_global_confirmed.png")
fig.show()
```


<div>


            <div id="8a53153d-52a3-4f58-982d-bfbd65537f62" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("8a53153d-52a3-4f58-982d-bfbd65537f62")) {
                    Plotly.newPlot(
                        '8a53153d-52a3-4f58-982d-bfbd65537f62',
                        [{"line": {"color": "firebrick"}, "mode": "lines+markers", "name": "Global", "type": "scatter", "x": ["2020-01-22T00:00:00", "2020-01-23T00:00:00", "2020-01-24T00:00:00", "2020-01-25T00:00:00", "2020-01-26T00:00:00", "2020-01-27T00:00:00", "2020-01-28T00:00:00", "2020-01-29T00:00:00", "2020-01-30T00:00:00", "2020-01-31T00:00:00", "2020-02-01T00:00:00", "2020-02-02T00:00:00", "2020-02-03T00:00:00", "2020-02-04T00:00:00", "2020-02-05T00:00:00", "2020-02-06T00:00:00", "2020-02-07T00:00:00", "2020-02-08T00:00:00", "2020-02-09T00:00:00", "2020-02-10T00:00:00", "2020-02-11T00:00:00", "2020-02-12T00:00:00", "2020-02-13T00:00:00", "2020-02-14T00:00:00", "2020-02-15T00:00:00", "2020-02-16T00:00:00", "2020-02-17T00:00:00", "2020-02-18T00:00:00", "2020-02-19T00:00:00", "2020-02-20T00:00:00", "2020-02-21T00:00:00", "2020-02-22T00:00:00", "2020-02-23T00:00:00", "2020-02-24T00:00:00", "2020-02-25T00:00:00", "2020-02-26T00:00:00", "2020-02-27T00:00:00", "2020-02-28T00:00:00", "2020-02-29T00:00:00", "2020-03-01T00:00:00", "2020-03-02T00:00:00", "2020-03-03T00:00:00", "2020-03-04T00:00:00", "2020-03-05T00:00:00", "2020-03-06T00:00:00", "2020-03-07T00:00:00", "2020-03-08T00:00:00", "2020-03-09T00:00:00", "2020-03-10T00:00:00", "2020-03-11T00:00:00", "2020-03-12T00:00:00", "2020-03-13T00:00:00", "2020-03-14T00:00:00", "2020-03-15T00:00:00", "2020-03-16T00:00:00", "2020-03-17T00:00:00", "2020-03-18T00:00:00", "2020-03-19T00:00:00", "2020-03-20T00:00:00", "2020-03-21T00:00:00", "2020-03-22T00:00:00", "2020-03-23T00:00:00", "2020-03-24T00:00:00"], "y": [555, 654, 941, 1434, 2118, 2927, 5578, 6166, 8234, 9927, 12038, 16787, 19881, 23892, 27635, 30794, 34391, 37120, 40150, 42762, 44802, 45221, 60368, 66885, 69030, 71224, 73258, 75136, 75639, 76197, 76819, 78572, 78958, 79561, 80406, 81388, 82746, 84112, 86011, 88369, 90306, 92840, 95120, 97886, 101801, 105847, 109821, 113590, 118620, 125875, 128352, 145205, 156101, 167454, 181574, 197102, 214821, 242500, 272035, 304396, 336953, 378235, 418045]}],
                        {"annotations": [{"arrowhead": 7, "showarrow": true, "text": "418045", "x": "2020-03-24T00:00:00", "xref": "x", "y": 418045, "yref": "y"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Global Confirmed Cases"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('8a53153d-52a3-4f58-982d-bfbd65537f62');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


### Distribution of confirmed cases per country

**Which are the countries with the most cases in the world?**

To answer this, we can do the following:


```python
# get the top values, see functions for details
top10,df = get_top_values(kind='Confirmed',top=10,ascending=False)
```


```python
top10
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
      <th>Confirmed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>China</th>
      <td>81591</td>
    </tr>
    <tr>
      <th>Italy</th>
      <td>69176</td>
    </tr>
    <tr>
      <th>US</th>
      <td>53740</td>
    </tr>
    <tr>
      <th>Spain</th>
      <td>39885</td>
    </tr>
    <tr>
      <th>Germany</th>
      <td>32986</td>
    </tr>
    <tr>
      <th>Iran</th>
      <td>24811</td>
    </tr>
    <tr>
      <th>France</th>
      <td>22622</td>
    </tr>
    <tr>
      <th>Switzerland</th>
      <td>9877</td>
    </tr>
    <tr>
      <th>Korea, South</th>
      <td>9037</td>
    </tr>
    <tr>
      <th>United Kingdom</th>
      <td>8164</td>
    </tr>
  </tbody>
</table>
</div>




```python
# we can plot this in a bar plot
y = top10['Confirmed'][::-1]
x = top10.index.to_list()[::-1]

fig = go.Figure(
    data=[
        go.Bar(
            x=y,
            y=x,
            orientation='h'
        )
    ]
)

fig.update_traces(
    marker_color='rgb(172,24,24)',
    marker_line_color='rgb(33,35,59)',
    marker_line_width=1.5,
    opacity=0.8
)

annotations=[]
for yi,xi in zip(y,x):
    annotations.append(
        dict(
            x=yi,
            y=xi,
            #xref="x",
            #yref="y",
            text='{}'.format(int(yi)),
            xshift=23,
            showarrow=False,
            #arrowhead=7
        )
    )

fig.update_layout(
    title='Global Confirmed Cases',
    xaxis_title="Date",
    yaxis_title="Confirmed Cases",
    font=dict(
        family="Arial, bold",
        size=14,
        #color="auto"
    ),
    annotations=annotations
)
fig.write_image("figs/bar_top10_confirmed.png")
fig.show()
```


<div>


            <div id="d3498bda-cd7c-48df-b31e-e551ccfce5f1" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("d3498bda-cd7c-48df-b31e-e551ccfce5f1")) {
                    Plotly.newPlot(
                        'd3498bda-cd7c-48df-b31e-e551ccfce5f1',
                        [{"marker": {"color": "rgb(172,24,24)", "line": {"color": "rgb(33,35,59)", "width": 1.5}}, "opacity": 0.8, "orientation": "h", "type": "bar", "x": [8164, 9037, 9877, 22622, 24811, 32986, 39885, 53740, 69176, 81591], "y": ["United Kingdom", "Korea, South", "Switzerland", "France", "Iran", "Germany", "Spain", "US", "Italy", "China"]}],
                        {"annotations": [{"showarrow": false, "text": "8164", "x": 8164, "xshift": 23, "y": "United Kingdom"}, {"showarrow": false, "text": "9037", "x": 9037, "xshift": 23, "y": "Korea, South"}, {"showarrow": false, "text": "9877", "x": 9877, "xshift": 23, "y": "Switzerland"}, {"showarrow": false, "text": "22622", "x": 22622, "xshift": 23, "y": "France"}, {"showarrow": false, "text": "24811", "x": 24811, "xshift": 23, "y": "Iran"}, {"showarrow": false, "text": "32986", "x": 32986, "xshift": 23, "y": "Germany"}, {"showarrow": false, "text": "39885", "x": 39885, "xshift": 23, "y": "Spain"}, {"showarrow": false, "text": "53740", "x": 53740, "xshift": 23, "y": "US"}, {"showarrow": false, "text": "69176", "x": 69176, "xshift": 23, "y": "Italy"}, {"showarrow": false, "text": "81591", "x": 81591, "xshift": 23, "y": "China"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Global Confirmed Cases"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('d3498bda-cd7c-48df-b31e-e551ccfce5f1');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>



With this information we could answer the question: **How fast the virus is spreading globally?**

The answer is easy, supose that we have defined $\delta$ as the increment in % from a value in time respect the previous values, i.e:

$$
\delta_i = \frac{t_{i}-t_{i-1}}{t_i}
$$

where $t_i$ is the value of our time series in the time $i$.


```python
deltas = get_deltas(g_confirmed)
#get the mean of the deltas
deltas_mean = deltas.loc['Global'].mean()

print("In average, the confirmed cases are increasing in: {0:.2f}% globally every day.".format(deltas_mean*100))
```

    In average, the confirmed cases are increasing in: 9.51% globally every day.
    

Now, **What Country is increasing the most in average daily?**


```python
top10,df = get_top_values(kind='deltas-mean',top=10,ascending=False)
```


```python
top10
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
      <th>deltas-mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Laos</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>Libya</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>Mozambique</th>
      <td>0.555556</td>
    </tr>
    <tr>
      <th>Belize</th>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>Dominica</th>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>Uganda</th>
      <td>0.472222</td>
    </tr>
    <tr>
      <th>Turkey</th>
      <td>0.430178</td>
    </tr>
    <tr>
      <th>Madagascar</th>
      <td>0.408824</td>
    </tr>
    <tr>
      <th>Kyrgyzstan</th>
      <td>0.402211</td>
    </tr>
    <tr>
      <th>Mauritius</th>
      <td>0.393991</td>
    </tr>
  </tbody>
</table>
</div>




```python
graph_lines(['Mozambique','Laos','Belize'])
```


<div>


            <div id="ec717c7b-8f67-42d1-948a-85f0e14e4a25" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("ec717c7b-8f67-42d1-948a-85f0e14e4a25")) {
                    Plotly.newPlot(
                        'ec717c7b-8f67-42d1-948a-85f0e14e4a25',
                        [{"line": {"width": 2}, "mode": "lines+markers", "name": "Mozambique", "type": "scatter", "x": ["2020-03-22", "2020-03-23", "2020-03-24"], "y": [1.0, 1.0, 3.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Laos", "type": "scatter", "x": ["2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, 2.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Belize", "type": "scatter", "x": ["2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, 1.0, 1.0]}],
                        {"annotations": [{"arrowhead": 7, "showarrow": true, "text": "3", "x": "2020-03-24", "xref": "x", "y": 3.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "2", "x": "2020-03-24", "xref": "x", "y": 2.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "1", "x": "2020-03-24", "xref": "x", "y": 1.0, "yref": "y"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Confirmed cases in Mozambique, Laos, and Belize"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('ec717c7b-8f67-42d1-948a-85f0e14e4a25');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


As we can see above, the countries with almost no data is making our data dirty, so we need to exclude those countries with little data.


```python
excluded_countries = list(Confirmed[Confirmed["3/23/20"]<200]['Country/Region'].unique())
df.drop(excluded_countries,axis=0,inplace=True)
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
      <th>deltas-mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Algeria</th>
      <td>0.185115</td>
    </tr>
    <tr>
      <th>Argentina</th>
      <td>0.253490</td>
    </tr>
    <tr>
      <th>Armenia</th>
      <td>0.208247</td>
    </tr>
    <tr>
      <th>Austria</th>
      <td>0.259671</td>
    </tr>
    <tr>
      <th>Bahrain</th>
      <td>0.148995</td>
    </tr>
    <tr>
      <th>Belgium</th>
      <td>0.147293</td>
    </tr>
    <tr>
      <th>Brazil</th>
      <td>0.244209</td>
    </tr>
    <tr>
      <th>Bulgaria</th>
      <td>0.238361</td>
    </tr>
    <tr>
      <th>Chile</th>
      <td>0.274982</td>
    </tr>
    <tr>
      <th>Colombia</th>
      <td>0.287727</td>
    </tr>
    <tr>
      <th>Croatia</th>
      <td>0.202524</td>
    </tr>
    <tr>
      <th>Czechia</th>
      <td>0.252994</td>
    </tr>
    <tr>
      <th>Diamond Princess</th>
      <td>0.065345</td>
    </tr>
    <tr>
      <th>Dominican Republic</th>
      <td>0.217697</td>
    </tr>
    <tr>
      <th>Ecuador</th>
      <td>0.217810</td>
    </tr>
    <tr>
      <th>Egypt</th>
      <td>0.130700</td>
    </tr>
    <tr>
      <th>Estonia</th>
      <td>0.189531</td>
    </tr>
    <tr>
      <th>Finland</th>
      <td>0.112372</td>
    </tr>
    <tr>
      <th>Germany</th>
      <td>0.162449</td>
    </tr>
    <tr>
      <th>Greece</th>
      <td>0.213403</td>
    </tr>
    <tr>
      <th>Iceland</th>
      <td>0.235374</td>
    </tr>
    <tr>
      <th>India</th>
      <td>0.104574</td>
    </tr>
    <tr>
      <th>Indonesia</th>
      <td>0.242514</td>
    </tr>
    <tr>
      <th>Iran</th>
      <td>0.241206</td>
    </tr>
    <tr>
      <th>Iraq</th>
      <td>0.180339</td>
    </tr>
    <tr>
      <th>Ireland</th>
      <td>0.255378</td>
    </tr>
    <tr>
      <th>Israel</th>
      <td>0.222591</td>
    </tr>
    <tr>
      <th>Italy</th>
      <td>0.164022</td>
    </tr>
    <tr>
      <th>Japan</th>
      <td>0.090602</td>
    </tr>
    <tr>
      <th>Korea, South</th>
      <td>0.114990</td>
    </tr>
    <tr>
      <th>Lebanon</th>
      <td>0.171572</td>
    </tr>
    <tr>
      <th>Luxembourg</th>
      <td>0.259462</td>
    </tr>
    <tr>
      <th>Malaysia</th>
      <td>0.108178</td>
    </tr>
    <tr>
      <th>Mexico</th>
      <td>0.213111</td>
    </tr>
    <tr>
      <th>Norway</th>
      <td>0.247172</td>
    </tr>
    <tr>
      <th>Pakistan</th>
      <td>0.205822</td>
    </tr>
    <tr>
      <th>Panama</th>
      <td>0.330402</td>
    </tr>
    <tr>
      <th>Peru</th>
      <td>0.279381</td>
    </tr>
    <tr>
      <th>Philippines</th>
      <td>0.111533</td>
    </tr>
    <tr>
      <th>Poland</th>
      <td>0.291398</td>
    </tr>
    <tr>
      <th>Portugal</th>
      <td>0.290086</td>
    </tr>
    <tr>
      <th>Qatar</th>
      <td>0.192432</td>
    </tr>
    <tr>
      <th>Romania</th>
      <td>0.224960</td>
    </tr>
    <tr>
      <th>Russia</th>
      <td>0.100956</td>
    </tr>
    <tr>
      <th>Saudi Arabia</th>
      <td>0.252812</td>
    </tr>
    <tr>
      <th>Serbia</th>
      <td>0.270832</td>
    </tr>
    <tr>
      <th>Singapore</th>
      <td>0.104403</td>
    </tr>
    <tr>
      <th>Slovenia</th>
      <td>0.254042</td>
    </tr>
    <tr>
      <th>South Africa</th>
      <td>0.292607</td>
    </tr>
    <tr>
      <th>Spain</th>
      <td>0.175678</td>
    </tr>
    <tr>
      <th>Sweden</th>
      <td>0.129360</td>
    </tr>
    <tr>
      <th>Switzerland</th>
      <td>0.267045</td>
    </tr>
    <tr>
      <th>Thailand</th>
      <td>0.084519</td>
    </tr>
    <tr>
      <th>Turkey</th>
      <td>0.430178</td>
    </tr>
    <tr>
      <th>US</th>
      <td>0.141665</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.sort_values(by='deltas-mean',ascending=False,inplace=True)
df.head(10)
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
      <th>deltas-mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Turkey</th>
      <td>0.430178</td>
    </tr>
    <tr>
      <th>Panama</th>
      <td>0.330402</td>
    </tr>
    <tr>
      <th>South Africa</th>
      <td>0.292607</td>
    </tr>
    <tr>
      <th>Poland</th>
      <td>0.291398</td>
    </tr>
    <tr>
      <th>Portugal</th>
      <td>0.290086</td>
    </tr>
    <tr>
      <th>Colombia</th>
      <td>0.287727</td>
    </tr>
    <tr>
      <th>Peru</th>
      <td>0.279381</td>
    </tr>
    <tr>
      <th>Chile</th>
      <td>0.274982</td>
    </tr>
    <tr>
      <th>Serbia</th>
      <td>0.270832</td>
    </tr>
    <tr>
      <th>Switzerland</th>
      <td>0.267045</td>
    </tr>
  </tbody>
</table>
</div>




```python
graph_lines(['Mexico','Turkey','Poland'])

# as we can see now the data is more reliable
```


<div>


            <div id="c0856bfd-360b-4089-b4a8-997518ca423c" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("c0856bfd-360b-4089-b4a8-997518ca423c")) {
                    Plotly.newPlot(
                        'c0856bfd-360b-4089-b4a8-997518ca423c',
                        [{"line": {"width": 2}, "mode": "lines+markers", "name": "Mexico", "type": "scatter", "x": ["2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [1.0, 4.0, 5.0, 5.0, 5.0, 5.0, 5.0, 6.0, 6.0, 7.0, 7.0, 7.0, 8.0, 12.0, 12.0, 26.0, 41.0, 53.0, 82.0, 93.0, 118.0, 164.0, 203.0, 251.0, 316.0, 367.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Turkey", "type": "scatter", "x": ["2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, null, null, null, null, null, null, null, 1.0, 1.0, 5.0, 5.0, 6.0, 18.0, 47.0, 98.0, 192.0, 359.0, 670.0, 1236.0, 1529.0, 1872.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Poland", "type": "scatter", "x": ["2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23", "2020-03-24"], "y": [null, null, null, null, null, 1.0, 1.0, 5.0, 5.0, 11.0, 16.0, 22.0, 31.0, 49.0, 68.0, 103.0, 119.0, 177.0, 238.0, 251.0, 355.0, 425.0, 536.0, 634.0, 749.0, 901.0]}],
                        {"annotations": [{"arrowhead": 7, "showarrow": true, "text": "367", "x": "2020-03-24", "xref": "x", "y": 367.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "1872", "x": "2020-03-24", "xref": "x", "y": 1872.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "901", "x": "2020-03-24", "xref": "x", "y": 901.0, "yref": "y"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Confirmed cases in Mexico, Turkey, and Poland"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('c0856bfd-360b-4089-b4a8-997518ca423c');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


### Actual infected people
Okay, we have ploted the cumulated confirmed cases for any country with confirmed cases, but how about of computing the actual infected people in the world or in a country.

**How can we compute the actual infected people?**

$$
Actual Infected People = Confirmed - Deaths - Recovered
$$


```python
confirmed = Country('Italy').get_confirmed_ts()
recovered = Country('Italy').get_rec_ts()
deaths = Country('Italy').get_death_ts()
actual = confirmed - deaths - recovered
actual = consolidate_time_series([actual])
actual
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
      <th>2020-01-31</th>
      <th>2020-02-01</th>
      <th>2020-02-02</th>
      <th>2020-02-03</th>
      <th>2020-02-04</th>
      <th>2020-02-05</th>
      <th>2020-02-06</th>
      <th>2020-02-07</th>
      <th>2020-02-08</th>
      <th>2020-02-09</th>
      <th>...</th>
      <th>2020-03-14</th>
      <th>2020-03-15</th>
      <th>2020-03-16</th>
      <th>2020-03-17</th>
      <th>2020-03-18</th>
      <th>2020-03-19</th>
      <th>2020-03-20</th>
      <th>2020-03-21</th>
      <th>2020-03-22</th>
      <th>2020-03-23</th>
    </tr>
    <tr>
      <th>Country/Region</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Italy</th>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>3.0</td>
      <td>3.0</td>
      <td>...</td>
      <td>17750.0</td>
      <td>20603.0</td>
      <td>23073.0</td>
      <td>26062.0</td>
      <td>28710.0</td>
      <td>33190.0</td>
      <td>38549.0</td>
      <td>42681.0</td>
      <td>46638.0</td>
      <td>50826.0</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 53 columns</p>
</div>




```python
def graph_lines_actual_cases(countries):
    """Graph a line chart for every time serie of the countries passed in.
    
    Args:
        countries:
            list or array like object, list of countries to plot time series
    
    Returns:
        nothing, it displays the plot and save it to a local path called figs/file_name.png
    """
    # store every time series for each country
    ts = []
    for country in countries:
        confirmed = Country(country).get_confirmed_ts()
        recovered = Country(country).get_rec_ts()
        deaths = Country(country).get_death_ts()
        actual = confirmed - recovered - deaths
        ts.append(actual)
    
    # consolidate the time series in a dataframe
    conso = consolidate_time_series(ts)
    
    # get x and y values for the plot
    x = conso.columns.to_list()
    y = []
    for country in conso.index.to_list():
        y.append(conso.loc[country])
    
    # initialize plotly figure and
    # add the lines for each country
    fig = go.Figure()
    for i in range(len(countries)):
        fig.add_trace(go.Scatter(name=countries[i],x=x,y=y[i],mode='lines+markers',line=dict(width=2)))
    
    # store the annotations
    annotations = []
    for yi in y:
        annotations.append(
            dict(
                x=x[-1],
                y=yi[-1],
                xref="x",
                yref="y",
                text='{}'.format(int(yi[-1])),
                showarrow=True,
                arrowhead=7
            )
        )
    
    # store strings for title and file name
    strings = [] # for title
    strings2 = [] # for file name
    if len(countries)!=1:
        for i in range(len(countries)):
            if i==(len(countries)-1):
                strings.append('and '+countries[i])
                strings2.append(countries[i])

            else:
                strings.append(countries[i]+', ')
                strings2.append(countries[i])
    
    else:
        strings.append(countries[0])
        strings2.append(countries[0])
    
    # title for layout
    title = 'No. of actual infected people in '+ ''.join(strings)
    
    # layout
    fig.update_layout(
        title=title,
        xaxis_title="Date",
        yaxis_title="Confirmed Cases",
        font=dict(
            family="Arial, bold",
            size=14,
            #color="auto"
        ),
        annotations=annotations
    )
    
    # filename for saving plot
    file_name = 'lines_actual_infected_'+'_'.join(strings2)+'.png'
    
    fig.write_image("figs/"+file_name)
    fig.show()
```


```python
graph_lines_actual_cases(['China','US','Italy'])
```


<div>


            <div id="6c1fcae9-f402-4bfd-a18e-065200177422" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("6c1fcae9-f402-4bfd-a18e-065200177422")) {
                    Plotly.newPlot(
                        '6c1fcae9-f402-4bfd-a18e-065200177422',
                        [{"line": {"width": 2}, "mode": "lines+markers", "name": "China", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23"], "y": [503.0, 595.0, 858.0, 1325.0, 1970.0, 2737.0, 5277.0, 5834.0, 7835.0, 9375.0, 11357.0, 15806.0, 18677.0, 22373.0, 25762.0, 28477.0, 31393.0, 33413.0, 35705.0, 37424.0, 38638.0, 38560.0, 52309.0, 56860.0, 57452.0, 57992.0, 58108.0, 58002.0, 56541.0, 54825.0, 54608.0, 51859.0, 51390.0, 49631.0, 47413.0, 45365.0, 42924.0, 39809.0, 37199.0, 34898.0, 32368.0, 29864.0, 27402.0, 25230.0, 23702.0, 22159.0, 20335.0, 18933.0, 17567.0, 16116.0, 14859.0, 13569.0, 12124.0, 10783.0, 9906.0, 9030.0, 8106.0, 7372.0, 6731.0, 6189.0, 5799.0, 5410.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "US", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23"], "y": [1.0, 1.0, 2.0, 2.0, 5.0, 5.0, 5.0, 5.0, 5.0, 7.0, 8.0, 8.0, 11.0, 11.0, 11.0, 11.0, 11.0, 11.0, 8.0, 8.0, 9.0, 9.0, 10.0, 10.0, 10.0, 10.0, 10.0, 10.0, 10.0, 10.0, 10.0, 10.0, 10.0, 46.0, 45.0, 51.0, 52.0, 53.0, 60.0, 66.0, 85.0, 104.0, 131.0, 198.0, 241.0, 378.0, 490.0, 554.0, 923.0, 1237.0, 1611.0, 2120.0, 2661.0, 3424.0, 4530.0, 6296.0, 7665.0, 13477.0, 18856.0, 25182.0, 32859.0, 43112.0]}, {"line": {"width": 2}, "mode": "lines+markers", "name": "Italy", "type": "scatter", "x": ["2020-01-22", "2020-01-23", "2020-01-24", "2020-01-25", "2020-01-26", "2020-01-27", "2020-01-28", "2020-01-29", "2020-01-30", "2020-01-31", "2020-02-01", "2020-02-02", "2020-02-03", "2020-02-04", "2020-02-05", "2020-02-06", "2020-02-07", "2020-02-08", "2020-02-09", "2020-02-10", "2020-02-11", "2020-02-12", "2020-02-13", "2020-02-14", "2020-02-15", "2020-02-16", "2020-02-17", "2020-02-18", "2020-02-19", "2020-02-20", "2020-02-21", "2020-02-22", "2020-02-23", "2020-02-24", "2020-02-25", "2020-02-26", "2020-02-27", "2020-02-28", "2020-02-29", "2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05", "2020-03-06", "2020-03-07", "2020-03-08", "2020-03-09", "2020-03-10", "2020-03-11", "2020-03-12", "2020-03-13", "2020-03-14", "2020-03-15", "2020-03-16", "2020-03-17", "2020-03-18", "2020-03-19", "2020-03-20", "2020-03-21", "2020-03-22", "2020-03-23"], "y": [null, null, null, null, null, null, null, null, null, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 19.0, 59.0, 150.0, 221.0, 311.0, 438.0, 593.0, 821.0, 1053.0, 1577.0, 1835.0, 2263.0, 2706.0, 3296.0, 3916.0, 5061.0, 6387.0, 7985.0, 8794.0, 10590.0, 10590.0, 14955.0, 17750.0, 20603.0, 23073.0, 26062.0, 28710.0, 33190.0, 38549.0, 42681.0, 46638.0, 50826.0]}],
                        {"annotations": [{"arrowhead": 7, "showarrow": true, "text": "5410", "x": "2020-03-23", "xref": "x", "y": 5410.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "43112", "x": "2020-03-23", "xref": "x", "y": 43112.0, "yref": "y"}, {"arrowhead": 7, "showarrow": true, "text": "50826", "x": "2020-03-23", "xref": "x", "y": 50826.0, "yref": "y"}], "font": {"family": "Arial, bold", "size": 14}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "No. of actual infected people in China, US, and Italy"}, "xaxis": {"title": {"text": "Date"}}, "yaxis": {"title": {"text": "Confirmed Cases"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('6c1fcae9-f402-4bfd-a18e-065200177422');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>



```python
actual_infected = Country('China').get_confirmed_ts() - Country('China').get_death_ts() -Country('China').get_rec_ts()
actual_infected.dropna(axis=1, inplace=True)

wti = pd.read_csv('data\\petro\\wti-daily_csv.csv')
wti['Date'] = pd.to_datetime(wti['Date'])
wti = wti[wti['Date']>='2020-01-22']
actual_infected.loc['China']

wti.set_index(keys='Date',inplace=True)
dates = pd.to_datetime(actual_infected.columns.to_list())
actual_infected.rename(dict(zip(actual_infected.columns.to_list(),dates)),axis=1,inplace=True)

wti['Infected in China'] = actual_infected.loc['China']
wti
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
      <th>Price</th>
      <th>Infected in China</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2020-01-22</th>
      <td>56.76</td>
      <td>503.0</td>
    </tr>
    <tr>
      <th>2020-01-23</th>
      <td>55.51</td>
      <td>595.0</td>
    </tr>
    <tr>
      <th>2020-01-24</th>
      <td>54.09</td>
      <td>858.0</td>
    </tr>
    <tr>
      <th>2020-01-27</th>
      <td>53.09</td>
      <td>2737.0</td>
    </tr>
    <tr>
      <th>2020-01-28</th>
      <td>53.33</td>
      <td>5277.0</td>
    </tr>
    <tr>
      <th>2020-01-29</th>
      <td>53.29</td>
      <td>5834.0</td>
    </tr>
    <tr>
      <th>2020-01-30</th>
      <td>52.19</td>
      <td>7835.0</td>
    </tr>
    <tr>
      <th>2020-01-31</th>
      <td>51.58</td>
      <td>9375.0</td>
    </tr>
    <tr>
      <th>2020-02-03</th>
      <td>50.06</td>
      <td>18677.0</td>
    </tr>
    <tr>
      <th>2020-02-04</th>
      <td>49.59</td>
      <td>22373.0</td>
    </tr>
    <tr>
      <th>2020-02-05</th>
      <td>50.87</td>
      <td>25762.0</td>
    </tr>
    <tr>
      <th>2020-02-06</th>
      <td>50.94</td>
      <td>28477.0</td>
    </tr>
    <tr>
      <th>2020-02-07</th>
      <td>50.34</td>
      <td>31393.0</td>
    </tr>
    <tr>
      <th>2020-02-10</th>
      <td>49.59</td>
      <td>37424.0</td>
    </tr>
    <tr>
      <th>2020-02-11</th>
      <td>50.00</td>
      <td>38638.0</td>
    </tr>
    <tr>
      <th>2020-02-12</th>
      <td>51.13</td>
      <td>38560.0</td>
    </tr>
    <tr>
      <th>2020-02-13</th>
      <td>51.41</td>
      <td>52309.0</td>
    </tr>
    <tr>
      <th>2020-02-14</th>
      <td>52.03</td>
      <td>56860.0</td>
    </tr>
    <tr>
      <th>2020-02-18</th>
      <td>52.10</td>
      <td>58002.0</td>
    </tr>
    <tr>
      <th>2020-02-19</th>
      <td>53.31</td>
      <td>56541.0</td>
    </tr>
    <tr>
      <th>2020-02-20</th>
      <td>53.77</td>
      <td>54825.0</td>
    </tr>
    <tr>
      <th>2020-02-21</th>
      <td>53.36</td>
      <td>54608.0</td>
    </tr>
    <tr>
      <th>2020-02-24</th>
      <td>51.36</td>
      <td>49631.0</td>
    </tr>
    <tr>
      <th>2020-02-25</th>
      <td>49.78</td>
      <td>47413.0</td>
    </tr>
    <tr>
      <th>2020-02-26</th>
      <td>48.67</td>
      <td>45365.0</td>
    </tr>
    <tr>
      <th>2020-02-27</th>
      <td>47.17</td>
      <td>42924.0</td>
    </tr>
    <tr>
      <th>2020-02-28</th>
      <td>44.83</td>
      <td>39809.0</td>
    </tr>
    <tr>
      <th>2020-03-02</th>
      <td>46.78</td>
      <td>32368.0</td>
    </tr>
    <tr>
      <th>2020-03-03</th>
      <td>47.27</td>
      <td>29864.0</td>
    </tr>
    <tr>
      <th>2020-03-04</th>
      <td>46.78</td>
      <td>27402.0</td>
    </tr>
    <tr>
      <th>2020-03-05</th>
      <td>45.90</td>
      <td>25230.0</td>
    </tr>
    <tr>
      <th>2020-03-06</th>
      <td>41.14</td>
      <td>23702.0</td>
    </tr>
    <tr>
      <th>2020-03-09</th>
      <td>31.05</td>
      <td>18933.0</td>
    </tr>
    <tr>
      <th>2020-03-10</th>
      <td>34.47</td>
      <td>17567.0</td>
    </tr>
    <tr>
      <th>2020-03-11</th>
      <td>33.13</td>
      <td>16116.0</td>
    </tr>
    <tr>
      <th>2020-03-12</th>
      <td>31.56</td>
      <td>14859.0</td>
    </tr>
    <tr>
      <th>2020-03-13</th>
      <td>31.72</td>
      <td>13569.0</td>
    </tr>
    <tr>
      <th>2020-03-16</th>
      <td>28.96</td>
      <td>9906.0</td>
    </tr>
    <tr>
      <th>2020-03-18</th>
      <td>26.96</td>
      <td>8106.0</td>
    </tr>
    <tr>
      <th>2020-03-19</th>
      <td>20.48</td>
      <td>7372.0</td>
    </tr>
    <tr>
      <th>2020-03-20</th>
      <td>25.09</td>
      <td>6731.0</td>
    </tr>
    <tr>
      <th>2020-03-23</th>
      <td>19.48</td>
      <td>5410.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
sns.lmplot(x='Price',y='Infected in China',data=wti)
```




    <seaborn.axisgrid.FacetGrid at 0x2013329b188>




![png](output_55_1.png)



```python
wti.corr()
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
      <th>Price</th>
      <th>Infected in China</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Price</th>
      <td>1.000000</td>
      <td>0.377296</td>
    </tr>
    <tr>
      <th>Infected in China</th>
      <td>0.377296</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



### Global Mortality rates


```python
nones = [None]*len(Countries)
d = dict(zip(Countries,nones))

for country in Countries:    
    d[country]= Pais(country).get_actual_mortality_rate()
```

    C:\ProgramData\Anaconda3\lib\site-packages\ipykernel_launcher.py:88: RuntimeWarning: invalid value encountered in true_divide
    


```python
excluded_countries = list(Confirmed[Confirmed["3/23/20"]<200]['Country/Region'].unique())
```


```python
global_mr = pd.DataFrame(data={"Mortality Rate":list(d.values())},
                         index=list(d.keys()))
#global_mr = global_mr.sort_values(ascending=False)

#for i in range(len(global_mr)):
#    print(str(i+1)+".- "+global_mr.index.to_list()[i] + ": " + str(global_mr[i]))

global_mr = global_mr.sort_values(by='Mortality Rate', ascending=False)
global_mr.drop(excluded_countries,axis=0,inplace=True)
global_mr.head(15)
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
      <th>Mortality Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Italy</th>
      <td>0.098589</td>
    </tr>
    <tr>
      <th>Iraq</th>
      <td>0.085443</td>
    </tr>
    <tr>
      <th>Indonesia</th>
      <td>0.080175</td>
    </tr>
    <tr>
      <th>Iran</th>
      <td>0.077949</td>
    </tr>
    <tr>
      <th>Algeria</th>
      <td>0.071970</td>
    </tr>
    <tr>
      <th>Spain</th>
      <td>0.070402</td>
    </tr>
    <tr>
      <th>Philippines</th>
      <td>0.063406</td>
    </tr>
    <tr>
      <th>Egypt</th>
      <td>0.049751</td>
    </tr>
    <tr>
      <th>Japan</th>
      <td>0.036044</td>
    </tr>
    <tr>
      <th>Belgium</th>
      <td>0.028578</td>
    </tr>
    <tr>
      <th>Greece</th>
      <td>0.026918</td>
    </tr>
    <tr>
      <th>Ecuador</th>
      <td>0.024954</td>
    </tr>
    <tr>
      <th>Turkey</th>
      <td>0.023504</td>
    </tr>
    <tr>
      <th>Brazil</th>
      <td>0.020472</td>
    </tr>
    <tr>
      <th>Dominican Republic</th>
      <td>0.019231</td>
    </tr>
  </tbody>
</table>
</div>




```python
global_mr['Mortality Rate'].mean()
```




    0.019868207098475234

