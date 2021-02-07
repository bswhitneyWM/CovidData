## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/bswhitneyWM/CovidData/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
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

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/bswhitneyWM/CovidData/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
