## Project Description

In this project I grab data from [covidtracking.com](https://api.covidtracking.com/v1/states/daily.csv) to explore the current status of COVID-19 in a variety of different states. The data is updated daily, and the data, such as number of deaths or cases, is separated on a state by state basis.

### Functions

Below are the functions I used to either update the data or load the most recent copy, and plot a given metric (such as # of deaths) for a given state.

```python
def UpdateCovidTracking():
    """ Function will download current copy of the historic state data from
    the covid tracking project"""
    import requests
    from datetime import datetime as dt
    import pytz
    import os
    
    # Url for data
    url = "https://api.covidtracking.com/v1/states/daily.csv"
    
    # Define file name for saving the data
    if not os.path.exists('./data'):
        os.makedirs("./data")
    # Now construct the file name
    file_name = "./data/ctp_" + str(dt.now(tz=pytz.utc)).replace(" ", "_") + ".csv"
    
    # Download the file
    r = requests.get(url)
    with open(file_name, "wb") as f:
        f.write(r.content)
    
    print('Date updated')
    
    
    
    
def LoadCTPData(update_first=False):
    """Return the most recent copy of our CTP data as a DataFrame and apply
        necessary preprocessing
    
        If update_first = True, then call updateCovidTracking() first..."""
    import glob
    import pandas as pd
    from datetime import datetime as dt
    import pytz
    
    if update_first:
        updateCovidTracking()
    
    # Find all data files that start with CTP
    files = glob.glob('./data/ctp*')
    
    # Sort list of file names, take the most recent one
    files.sort()
    file = files[-1]
    
    # Import this file using pandas
    df = pd.read_csv(file)
    
    # Extract Columns we want
    cols = ['date', 'state', 'positive', 'death']
    df_filtered = df[cols].copy()
    
    # Fix Dates and set them as the index
    date_format = '%Y%m%d'
    df_filtered.date = df_filtered.date.apply(lambda d: dt.strptime(str(d), date_format))
    df_filtered.index = df_filtered.date
    df_filtered = df_filtered.drop(columns=['date'])
    
    return df_filtered


def ShowCovidTracking(state, metric, update_first=False):
    """ Call LoadCTPData to load the data and then plot the results for the
        state and metric ("death" or "positive") of deaths or positive cases
        since data started being collected
    """
    
    import matplotlib.pyplot as plt
    
    # Load the data
    df = LoadCTPData(update_first)
    
    # get state data
    idx_state = df.state == state
    df_current_state = df[idx_state].copy()
    
    # Calculate new and total cases
    new_cases = df_current_state[metric][0] - df_current_state[metric][1]
    total_cases = df_current_state[metric][0]
    
    # Make the plot!
    plt.figure(figsize=(12,6))
    plt.plot(df_current_state[metric])
    plt.xticks(rotation=45)
    if metric == "death":
        plt.ylabel("Deaths")
        plt.title("Covid-19 related deaths in " + state)
    else:
        plt.ylabel("Positive Cases")
        plt.title("Covid-19 positive cases in " + state)
    plt.show()
```

Using these functions I am able to produce graphs of positive cases or deaths for any state I want. One example in Massachuetts is ![Here]({{site.url}}/CovidMA.png)
