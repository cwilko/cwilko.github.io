---
layout: post
title:  "A Financial Data pipeline in Python"
date:   2017-03-01
category: Quant Finance
---

# Preparing Data

Arguably the most important step in any data science or machine learning task is the preparation of the data. Taking the raw data and and turning it into a set of "features", in a particular format that a algorithm can work with is, is crucial to getting accurate and optimal results.

Often we find that there are a set of common steps that we need to apply to all datasets, regardless of the problem at hand. Other times we find we need to write specific logic to process the data for a particular problem.

It is useful to extract the common steps into a library or toolset. These operations can be resused against each new problem. We can think of passing the raw data through a preprocessing pipeline. In one end we add the raw data, and configure the pipeline to apply various operations to the data. The output of the pipeline is the processed data, ready to be fed into the algorithm of choice. Incidentally, the pipeline concept is a well known [design pattern](https://www.cise.ufl.edu/research/ParallelPatterns/PatternLanguage/AlgorithmStructure/Pipeline.htm) in software engineering. 

# An Example

This blog is primarily concerned with Financial Time Series data. We can imagine that there are a standard set of operation we wish to apply to a raw data file. For example, we may wish to convert the time series to a common timezone (UTC, GMT, etc), we may wish to crop between certain dates, or we may wish to remove missing data.

As well as a library of operations, we also need the infrastructure to controll the flow through the pipeline, and a mechanism to configure the pipeline (preferably via a config file).

The following snippet shows how we may loop through a set of operations and apply them to the input data. Once a number of operations have been applied to the data, we save the results as a CSV file ready to be fed into a subsequent algorithm (machine learning, backtest, etc) :

```python   
CONFIG_FILE = "config.json"

with open(CONFIG_FILE) as data_file:    
    config = json.load(data_file)

for dataset in config["datasets"]:
    
    # Load Dataframe from store
    storedData = hdfStore[dataset["name"]]

    ### INTRADAY DATA PIPELINE ##############
    
    ## Localise date to timezone of the market
    ## Crop to configured dates
    ## Resample to 5 min periods (to introduce nans in all missing periods)
    ## Crop on time (e.g. only keep 9am to 5pm each day)

    data = localize(storedData, dataset["timezone")

    data = cropDate(dafa, dataset["crop"]["start"], dataset["crop"]["end"])
  
    data = resample(data, dataset["sample_unit"])
    
    data = cropTime(data, dataset["start_time"], dataset["end_time"])
  
    ########################################
    
    ## Save output
    save_csv(data, dataset["name"] + ".csv")
```

The above snippet loads in a JSON configuration file, which contains the variables in this instance of this pipeline. We may want to change these as we experiment with the data so it's important to abstract these out into a file rather than in the code.

For example, the JSON file may contain the following data :

```json
"datasets": [
				{
					"name":"WallSt-5min",
					"timezone":"US/Eastern",
					"sample_unit":"5min",
					"crop": {
						"start": "2016-09-09",
						"end": "2017-09-09"
					}
				},
				{
					"name":"FTSE-5min",
					"timezone":"Europe/London",
					"sample_unit":"5min",
					"crop": {
						"start": "2016-09-09",
						"end": "2017-09-09"
					}
				}
			]

```
# Pipeline Operations

Here are some example snippet of the operations that we apply to the raw data :

```python 
##
## Crop - take a Pandas timeseries dataframe and returns a time cropped dataframe.
##

def cropDate(data, start, end):
    return data[start:end]

def cropTime(data, start, end):
    return data.between_time(start, end, include_start=True, include_end=False)

##
## Localise - take a GMT pandas timeseries dataframe and convert to a given timezone
##
import pytz
def localize(data, datasource, dataset):
    print "Converting " + dataset["name"] + " from " + datasource["timezone"] + " to " + dataset["timezone"]
    timezone = pytz.timezone(datasource["timezone"])
    data.index = data.index.tz_localize("Europe/London").tz_convert(timezone)
    return data
```

Specifically for Machine Learning problems, We can create operations for normalising the data (between 0 and 1, or -1 and +1, encoding the classification of the data (e.g. One Hot encoding, binary encoding), and synthesizing features from the raw OHLC data. For this last item for example, we may wish to translate into close data, japanese candlesticks, an indicator of some sort. Other operations may include shuffling the data, or separating into training, test, and validation sets.

Here are some more example snippets :

```python 
##
## Normalise (Candlesticks)
## Take a Pandas DF of data where each row is training example, e.g. a sequence of 5 min data between 9am and 10am, and output
## a normalisation of that data across the sequence.
##
def normaliseCandlesticks(data):
    X = data.values
    Xmax = X.max(axis=1)[numpy.newaxis].T
    Xmin = X.min(axis=1)[numpy.newaxis].T
    scale = Xmax - Xmin
    X = (X - Xmin) / scale
    return pandas.DataFrame(numpy.hstack((X,scale / numpy.nanmax(scale))))

##
## Split (Train/Val/Test)
## Tale a Pandas DF of data and split into proportions given by the parameters.
##
def split(data, train=.6, val=.2, test=.2):
    idx = numpy.arange(0,len(data)) / float(len(data))
    msk1 = data[idx<train]
    msk2 = data[(idx>=train) & (idx<(train + val))]
    msk3 = data[(idx>=(train+val))]
    return [msk1, msk2, msk3]
```
# What next?

Getting the pre-processing step right is cruicial in ensuring we have a clean set of data to feed into our analysis phase. We want to spend most of our time trying out differnet hypotheses on the data, rather than writing code to turn the data into a particular format. It is therefore essential to have a set of tools we can quickly utilise to get our data ready for this next phase.

It is conceivable that the pipeline library could be extended to not only include pre-processing tasks, but also data analysis tasks such as linear regression, logistic regression, neural networks, etc.

All the code, and more, can be found in a [Jupyter notebook](https://github.com/cwilko/Notebooks/blob/master/Dataset_Pipeline.ipynb) on my github pages.












