
Author: Venkat Kaushik
=======

Created: May 8, 2019
========

Presentation Link: 
==================
https://web.microsoftstream.com/video/c71da57e-a6d2-475c-9980-59199b160904

Duration: ~40 mins
---------

Pre-requisites
==============

* Install Jupyter Notebook

https://jupyter.readthedocs.io/en/latest/install.html

* Install Slack Client
https://pypi.org/project/slackclient/

If you need to setup new app use the link below and sign in with your account information. 
You may request your app to be published 
https://api.slack.com/apps 

* References: 

  * Slack API methods
https://api.slack.com/methods/chat.postMessage

  * Building Great Slack Integration
http://www.heyupdate.com/blog/building-a-slack-integration/

## libraries we are going to need
import os
import slack
import pprint
from datetime import datetime, timedelta
from pandas.io.json import json_normalize## environment variable needed to use SLACK API's
print( os.environ['SLACK_API_TOKEN_BOT'][0:5], os.environ['SLACK_API_TOKEN_USER'][0:5])                              

```python
## instantiate a client using the API token
clbot = slack.WebClient(token=os.environ['SLACK_API_TOKEN_BOT'])
clusr = slack.WebClient(token=os.environ['SLACK_API_TOKEN_USER'])
pprint.pprint (dir(clbot)[-140:-1])
```


```python
response = clbot.chat_postMessage(channel='#noise_scraper_test', text = "hi")
assert(response['ok'])
```


```python
response = clbot.channels_list()
```


```python
print (response.api_url, response.status_code)
print (response.data.keys())
response.data['ok']
#response.data['channels']
```


```python
df = json_normalize(response.data['channels'])
```


```python
pprint.pprint(df.columns)
df.head()
```


```python
## select channel details whose column "name" equals "techops-data-alerts"
df.loc[ df['name'] == 'techops-data-alerts']
```


```python
tnow = datetime.now()
tm24 = tnow - timedelta( hours = 24 )
print (tnow.strftime('%s.%f'), tm24.strftime('%s.%f'))
```


```python
response = clusr.channels_history( channel = 'CA9CS2MNG', latest = tnow.strftime('%s.%f'), oldest = tm24.strftime('%s.%f') )
```


```python
print (response.api_url, response.status_code)
print (response.data.keys())
response.data['ok'], response.data['has_more']
```


```python
dm = json_normalize(response.data['messages'])
```


```python
pprint.pprint(dm.columns)
dm.head()
```


```python
for row in dm.iterrows():
    pprint.pprint(row[1]['attachments'][0])
    break
```


```python
import pandas as pd

def select_columns(data_frame, column_names):
    new_frame = data_frame.loc[:, column_names]
    return new_frame

td = select_columns(dm, 'attachments').apply(lambda x: x[0]['title'])
td = pd.DataFrame(td) 
print ( td[td['attachments'].str.contains('PROBLEM') == True])
print ( td[td['attachments'].str.contains('PROBLEM') == True].count())
```
