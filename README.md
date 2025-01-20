# Others

# ECB database API

This notebook presents several functions for data fetching through the ECB database API.

**Summary**
1. Extracting one key : *get_data(key), get_series(datasets)*
2. Extracting several keys : *get_datasets(keys)*

ECB API help webpage : https://data.ecb.europa.eu/help/api/data

import requests
from io import StringIO
import pandas as pd
import matplotlib.pyplot as plt

# 1. Extracting one key

# Extraction of one dataset
def get_data(key):
    
    base_url = "https://data-api.ecb.europa.eu/service/data/"
    
    key_parts = key.split('.',1)
    
    url = f"{base_url}{key_parts[0]}/{key_parts[1]}?format=csvdata"
    
    # Replace USER and PASSWORD with your own username/password.
    proxy = {'https':'https://USER:PASSWORD@proxyusers.intranatixis.com:8080'} 
    
    r = requests.get(url, proxies=proxy, verify=False)

    if r.status_code == 200:
        data = pd.read_csv(StringIO(r.content.decode('utf-8')))
        
        print("GENERAL INFORMATIONS",f"({key})","\n" + "=" * 100)
        print('TITLE:',data['TITLE_COMPL'].iloc[0],"\n"+"-"*100)
        print("START:",data['TIME_PERIOD'].iloc[0],"END:",data['TIME_PERIOD'].iloc[-1],"\n"+"-"*100)
        print(", ".join(data.columns.tolist()))
        print("-" * 100)
              
        return data
    else:
        print('Not connected')

# Extraction of the time series of the dataset
def get_series(dataset):
    series = pd.Series(data=dataset['OBS_VALUE'].tolist(), index=dataset['TIME_PERIOD'].tolist())
    series.name = dataset['TITLE_COMPL'].iloc[0] 
    series.index.name = 'Time'
    return series

### Example

key = 'EXR.D.USD.EUR.SP00.A'

raw_data = get_data(key)

# Display the first 3 lines of the extracted dataset
print(raw_data[0:3])

# Extraction of the observed values of the dataset
series = get_series(raw_data)

print(series[0:3])

# 2. Extracting several keys

def get_datasets(keys):
    datasets = []
    index = 1
    for k in keys:
        print('DATASET N°',index)
        datasets.append(get_data(k))
        index += 1
    return datasets

### Example

keys = [
    'EXR.D.USD.EUR.SP00.A',
    'SHSS.Q.N.U2.W0.S12Q.S12P.N.A.LE.F3.S._Z.XDC._T.F.V.N._T',
    'AME.A.DNK.1.0.0.0.OVGD'
]

datasets = get_datasets(keys)

# Display the first 3 lines of the dataset n°1
print(datasets[0][0:3])
