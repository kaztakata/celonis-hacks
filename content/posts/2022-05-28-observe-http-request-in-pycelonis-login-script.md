---
title: "Observe HTTP request in Pycelonis login script"
date: 2022-05-28T07:59:48+09:00
draft: false
tags: ['Machine Learning']
---

I showed how to login to Celonis EMS using Pycelonis in previous posts. Authentication topic I mentioned there is the most annoying when using API, but after mastering this I can transfer this knowledge to another areas easily. 

To master authentication, I would like to show the mechanism of HTTP request in the internet (web) programming. Even I did not know HTTP request well at the beginning of using Celonis, now I got some of basic knowledge and it is enough to use HTTP request.

Today I will continue using ML notebook and stop at the timing of HTTP request, then show what is passed to Celonis EMS and what is returned. Using below script (modify from [Login to Celonis EMS from Jupyter Workbench](../2022-05-14-login-to-celonis-ems-from-jupyter-workbench)) I ran script, so that stop program at `pdb.set_trace()` point.

```python
###### import packages
import json
from pycelonis import get_celonis
from pycelonis.celonis_api.utils import KeyType

# open credential file
with open('/home/jovyan/pycelonis-profiles/user.json') as f:
    credential = json.loads(f.read())

# stop program
import pdb 
pdb.set_trace()

# login to Celonis EMS
celonis = get_celonis(**credential, key_type=KeyType.USER_KEY, total_retry=10, permissions=False)
```

[![image](https://user-images.githubusercontent.com/67397583/170803914-b9f06c6a-00b7-44a2-92d6-aa248cf4d02f.png)](https://user-images.githubusercontent.com/67397583/170803914-b9f06c6a-00b7-44a2-92d6-aa248cf4d02f.png)

Now it is possible to slowly run script using this debugger. At the input window at the bottom of output, command as below (ignore `ipdb>` when enter command).

```
ipdb> b pycelonis/celonis_api/celonis.py:626
Breakpoint 1 at /home/jovyan/.local/lib/python3.8/site-packages/pycelonis/celonis_api/celonis.py:626
```

```
ipdb> c
...
> /home/jovyan/.local/lib/python3.8/site-packages/pycelonis/celonis_api/celonis.py(626)api_request()
    624 
    625         try:
1-> 626             response = self._session.request(**kwargs, **extra)
    627         except ReadTimeout as e:
    628             raise PyCelonisHTTPError(
```

First command makes new breakpoint to stop program at HTTP request. Second command just go until new breakpoint. After that I am at `response = self._session.request(**kwargs, **extra)` line, where famous python library [requests](https://requests.readthedocs.io/en/latest/) is used to execute HTTP request. Before executing request, I would like to see some of parameters in request. Command as below.

```
ipdb>  url
'https://your_team.your_realm.celonis.cloud/api/cloud'
```

```
ipdb>  self._session.headers
{'User-Agent': 'pycelonis/1.7.0', 'Accept-Encoding': 'gzip, deflate, br', 'Accept': '*/*', 'Connection': 'keep-alive', 'authorization': 'Bearer your_api_token'}
```

URL is also called endpoint, to access to resource in Web server. In this case this URL will retun login information. On the other hand header information is attached with URL, especially if authentication is required it is meaningful. Authorization attribute is stored my api_token with `Bearer` text (`AppKey` if Application Key is used). 

OK, let's run HTTP requst by command `n`.

```
ipdb> n
> /home/jovyan/.local/lib/python3.8/site-packages/pycelonis/celonis_api/celonis.py(637)api_request()
    635             )
    636 
--> 637         if response.status_code in [401, 403]:
    638             message = f"You don't have permission to perform {kwargs['method']} -> {url}."
    639             if self.permissions:
```

Program is moved to next line, after executing HTTP request. I can see the result in `response`, in this case `response.content` returns your account information if authentication is successful.
```
ipdb>  response.content
b'{your_account_information}'
```

Finally command `c` and run the rest of script to complete.
```
ipdb>  c
[2022-05-28 01:35:00] INFO: Initial connect successful! Hello your_email. PyCelonis Version: 1.7.0
```

In Pycelonis, `get_celonis` method returns authentication information to `celonis` object, so after that `celonis` object can access to another endpoints in various Pycelonis methods. For example after first cell in notebook is completed, input below script. I can stop script same as previous cell.

```python
celonis.default_space
```

Then enter command same as below.
```
ipdb>  url
'https://your_team.your_realm.celonis.cloud/package-manager/api/spaces'
ipdb>  n
...
ipdb>  response.content
b'[{"permissions":["CREATE_PACKAGE","DELETE_SPACE","USE","$ACCESS_CHILD","CREATE_SPACE","DELETE_ALL_PACKAGES","USE_ALL_PACKAGES","DELETE_PACKAGE","EDIT_PACKAGE","EDIT_ALL_PACKAGES","EDIT_ALL_SPACES","MANAGE_PERMISSIONS","EDIT_SPACE","DELETE_ALL_SPACES","USE_PACKAGE"],"id":"3c895d6e-38c1-4ea7-9451-13ee5c55b67a","name":"Default","iconReference":"earth","creationDate":1624615392821,"changeDate":1624615392821,"objectId":"3c895d6e-38c1-4ea7-9451-13ee5c55b67a"}]'
ipdb>  c
<Space, id 3c895d6e-38c1-4ea7-9451-13ee5c55b67a, name Default>
```

This is first step to understand HTTP request. Please do it by your hand, it is required for further instruction in my posts.

Kaz
