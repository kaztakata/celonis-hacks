---
title: "Include JSON data in HTTP Request and Response"
date: 2022-06-11T18:58:00+09:00
draft: false
tags: ['Machine Learning', 'Data Integration']
---

### About JSON

Until last post, I did not intentionally mention about detail of input and output data in HTTP request. Actually HTTP request requires not only URL (URI) but header data like `Authorization` (refer to [Observe HTTP request in Pycelonis login script](../2022-05-28-observe-http-request-in-pycelonis-login-script)). Also you can imagine it is possible to attach input form value (e.g. name, address, email). Generally in API world, JSON format is used to attach with HTTP request.

[JSON](https://www.json.org/json-en.html) (JavaScript Object Notation) is text file format to describe structured data. It was derived from JavaScript Programming Language but used for other environment. 

JSON is useful to describe complex structured data, that include 1) collection of Key and Value (name is Kazuhiko, country is Japan etc.) and 2) list of value (today's stock price, yesterday's one etc.). Due to this structure, JSON text is easily converted to data structure in each programming languages. On the other hand, JSON is text data so it is easily communicated with Web server by HTTP protocol.

Focusing on Python environment, JSON file can be converted to nested dictionary and list, both are basic data type in Python and luckily looks of JSON text and Python data are almost same.

### Send JSON data as HTTP Request

Similar to last post [Find out HTTP Request from GUI Function](../2022-06-04-find-out-http-request-from-gui-function), I would like to find API from GUI function. Today I focus on Configura Alert button in Data Job menu as below.

[![image](https://user-images.githubusercontent.com/67397583/173181333-3b58cb59-f41e-4a84-be4c-e07d94a5558c.png)](https://user-images.githubusercontent.com/67397583/173181333-3b58cb59-f41e-4a84-be4c-e07d94a5558c.png)

I opened DevTools and record network in setup Alert configuration. This time I clicked `Payload` tab and I can see configuration I put in my screen like this (format text for readability).

```json
{
    "enabled": true,
    "jobId": "3427660d-6b6f-4d7b-8488-210cff8e7ebb",
    "notifyIfdataJobFails": true,
    "notifyIfdataJobFinishesSuccessfully": null,
    "notifyIfdataJobFinishesSkipped": null,
    "notifyIfdataJobExecutionTimeLongerThan": null
}
```

This is simple JSON data including collection of key and value. In this case, each key name is relevant to field in screen, for example first toggle button is `enabled` and its value is `true`.

OK I will mimic this configuration update in Pycelonis. I reused last post's Python script until login to Celonis, and changed last part as below.

```python
pool = celonis.pools.find('3d8a2621-f9c0-456a-9b2d-9bb8c0a9fcd1') # my Pool ID
datajob = pool.data_jobs.find('3427660d-6b6f-4d7b-8488-210cff8e7ebb') # my Job ID
celonis.api_request(datajob.url + '/alerts', {"enabled":True, "jobId":datajob.id, "notifyIfdataJobFails":True})
```

URL is picked up from DevTools, combination of Job URL and `/alerts`. Second argument `{"enabled":True, "jobId":datajob.id, "notifyIfdataJobFails":True}` is mimic of Payload from DevTools. This is Python dictionary data and it is internally converted to JSON text and send to Celonis. Please keep in mind Python boolean flag is `True`, even JSON it is `true` ([Ref](https://stackoverflow.com/questions/33486100/why-cant-i-use-true-false-null-in-a-python-object-fed-to-json-dumps)).

This time the purpose of HTTP request is to update alert configuration, so after running script, move to Data Job screen and confirm whether configuration is updated.

### Receive JSON data as HTTP Response

Additionally I would like to confirm response of HTTP request is also JSON data. Using last post example, I would like to receive response (export pool) data by `export_data = celonis.api_request(pool.url + '/export')`. Variable `export_data` is originally JSON but internally converted to Python structured data describing pool information. I will check the data content by below script.

```python
# JSON is converted to Python dictionary object
print(type(export_data)) 

# see collection of key
print(export_data.keys()) 

# 'transformations' is one of the key, including list of transformation 
print(export_data['transformations']) 
```

Actually some of Pycelonis method is to get JSON data from Celonis, format some text and return as dictionary like object (or list of that).

Kaz
