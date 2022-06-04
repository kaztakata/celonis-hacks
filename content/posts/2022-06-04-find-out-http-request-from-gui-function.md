---
title: "Find out HTTP Request from GUI Function"
date: 2022-06-04T09:25:43+09:00
draft: false
tags: ['Machine Learning', 'Data Integration']
---

In the previous post [Observe HTTP request in Pycelonis login script](../2022-05-28-observe-http-request-in-pycelonis-login-script), I showed how to observe HTTP request under Pycelonis API. In this observation I found HTTP request requires at least `Authorization` header and of course URL to reach to resource in Celonis EMS. Also I found that I can manage to investigate which HTTP request is sent when calling Pycelonis class method.

By the way, not all HTTP requests are implemented in Pycelonis. And sometimes I would like to use HTTP request that in not in Pycelonis. Even in this situation, if I can find GUI (browser) function, it may be possible to find HTTP request under GUI function.

Today I would like to show how to observe HTTP request in Celonis GUI (browser) function.

For example, I am aware about `Export` button in Data Pool three dots. In Pycelonis [Pool Class](https://celonis.github.io/pycelonis/1.7.0/reference/celonis_api/event_collection/data_pool/#celonis_api.event_collection.data_pool.Pool), there is not such export method. If I can easily call export API, it is convenient to backup Data Pool contents and to integrate with version control system like Git.

By the way, I will demonstrate in my Google Chrome browser from now on. If you use other browser, you can switch to Google Chrome or find similar function in your browser.

OK, assuming I click Data > Data Integration in left side menu, and now I am in Data Pool list screen. At that time enter F12 key, or enter Ctrl + Shift + I to open DevTools. If it is first time for you to open it, your screen may be similar to my screen below. This tool show many aspects in this web page, for example web page `Elements` that is structured HTML file content.

[![image](https://user-images.githubusercontent.com/67397583/171999882-0bcd3dc9-85c5-46d8-9b84-a3e6a826d773.png)](https://user-images.githubusercontent.com/67397583/171999882-0bcd3dc9-85c5-46d8-9b84-a3e6a826d773.png)

My interest is which HTTP requests are sent in this web page. So I click `>>` button at right side of `Elements` and choose `Network`, then I can see communication between this browser and Celonis EMS including HTTP request.

After that I clicked three dots and `Export`. Immediately some of rows are appened in Network tab like below.

[![image](https://user-images.githubusercontent.com/67397583/172000308-e27685d3-14f7-4f20-82cb-b3e298acf1a4.png)](https://user-images.githubusercontent.com/67397583/172000308-e27685d3-14f7-4f20-82cb-b3e298acf1a4.png)

I should look for HTTP request relevant to Pool Export from Network log. It is lucky that third log is exactly `export`. I click this log to see detail.

First I check `Headers > General` section and I can see `Request URL` in the first attribute. RequestURL is `https://your_email.training.celonis.cloud/integration//api/pools/your_pool_id/export` (`your_email` and `your_pool_id` are actually my email and Pool ID).

Using this URL pattern (called endpoint), I think export function can be reproduced in Pycelonis too. I will switch to Machine Learning Workbench, open notebook and run below script.

```python
###### import packages
import json
from pycelonis import get_celonis
from pycelonis.celonis_api.utils import KeyType

# open credential file
with open('/home/jovyan/pycelonis-profiles/user.json') as f:
    credential = json.loads(f.read())

# login to Celonis EMS
celonis = get_celonis(**credential, key_type=KeyType.USER_KEY, total_retry=10, permissions=False)

# Export Pool
pool_id = '3d8a2621-f9c0-456a-9b2d-9bb8c0a9fcd1' # my Pool ID
celonis.api_request(celonis.url + '/integration//api/pools/' + pool_id + '/export')
```

The output is quite similar to what I exported from GUI.

[![image](https://user-images.githubusercontent.com/67397583/172001206-928170e9-0698-4bf7-9202-c5a5e42f2200.png)](https://user-images.githubusercontent.com/67397583/172001206-928170e9-0698-4bf7-9202-c5a5e42f2200.png)

I need to explain the last line of Python script. This is what I emulate to send HTTP Request `celonis.api_request(celonis.url + '/integration//api/pools/' + pool_id + '/export')`.

- `celonis.api_request` is the way to send HTTP request using `Authorization` header that is from `get_celonis` method (see [Observe HTTP request in Pycelonis login script](../2022-05-28-observe-http-request-in-pycelonis-login-script)).

- `celonis.url` is team URL I am login by API token.


URL string can be shortened using Pycelonis Pool Class.
```python
pool = celonis.pools.find('3d8a2621-f9c0-456a-9b2d-9bb8c0a9fcd1')
celonis.api_request(pool.url + '/export')
```

This time `pool` object has its URL in `pool.url`, so just add '/export' is enough.


I believe today's topic is not so difficult. And I am happy if you know that you can extend Pycelonis method to automate some of GUI functions.

Kaz
