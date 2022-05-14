---
title: "Login to Celonis EMS from Jupyter Workbench"
date: 2022-05-14T10:40:29+09:00
draft: false
tags: ['Machine Learning']
---

At last post [Start Deep Dive to Machine Learning and Action Flow](../2022-05-07-start-deep-dive-to-machine-learning-and-action-flow), I introduced overview of these two functions that manipulate Celonis EMS functions without using GUI (browser) for automatation. 

From today I would like to share basic functions in Machine Learning (Jupyter Workbench). Before staring I will setup Jupyter Workbench. If you do not have your Workbench, read [Share my Analysis by Content-CLI](../2021-08-14-share-my-analysis-by-content-cli) then follow until creating Workbench.

I will go to Launcher in Workbench then select Others > Terminal, then enter below command after $ sign (output is as of 2022-05-14). First command is to upgrade latest version of Pycelonis 1.7.0. Second command is to confirm installed version.

```bash
(base) jovyan@workbench-d46f8977-mtfw6:~$ pip install pycelonis --upgrade
Looking in indexes: https://pypi.celonis.cloud, https://pypi.org/simple
...
Successfully installed ddsketch-2.0.2 ddtrace-1.1.2 pycelonis-1.7.0
```
```bash
(base) jovyan@workbench-d46f8977-mtfw6:~$ pip show pycelonis
Name: pycelonis
Version: 1.7.0
...
Required-by: celonis-ml
```

Next I will click New Folder button in left pane, then name it as `pycelonis-profiles`, that is for storing credeintial files later. Then move to this folder and create new file `user.json`. Just copy below content and modify `your_team`,`your_realm`,`your_api_token` to your value. For the moment I use API token from Edit Profile (see [Share my Analysis by Content-CLI](../2021-08-14-share-my-analysis-by-content-cli)).

```json
{
    "url": "https://your_team.your_realm.celonis.cloud", 
    "api_token": "your_api_token"
}
```

OK, it is time to run first script in Jupyter Workbench. As below screen, create new Nootbook (Python 3 ipykernel) and copy below script to first cell in notebook, then click Start button (or enter shift + Enter).

```python
# import packages
import json
from pycelonis import get_celonis
from pycelonis.celonis_api.utils import KeyType

# open credential file
with open('/home/jovyan/pycelonis-profiles/user.json') as f:
    credential = json.loads(f.read())

# login to Celonis EMS
celonis = get_celonis(**credential, key_type=KeyType.USER_KEY, total_retry=10, permissions=True)
```

[![image](https://user-images.githubusercontent.com/67397583/168415145-a4c02cc3-abbd-4340-88fc-c29323d15db4.png)](https://user-images.githubusercontent.com/67397583/168415145-a4c02cc3-abbd-4340-88fc-c29323d15db4.png)

This script is firstly import packages required for next step, then open credential file and use it to login to Celonis EMS. Login method `get_celonis` has [documentation]( https://celonis.github.io/pycelonis/1.7.0/reference/get_celonis/) to see each parameters. In this method, this notebook tried to login to Celonis EMS using API token, and showed avaiable permissions if URL and API token are valid.

Two points to learn from this operation are 
1. it is possible to login to any Celonis environment that is different from ML notebook location
2. Permission granted to this API token is same as GUI user.

Regarding 1, even if I am using different Jupyter environment e.g. my local machine, I just pass URL and API token then I can login to Celonis EMS. 

About 2, I am Admin user and it is too strong permissions for especially production environment. Considering unauthorized access, I will create one more API key that is weaker than Admin but enough for my operation in next post.

As today's last thing, I would like to show one example to use pycelonis. Below is displaying Celonis Packages under login enviornment which I created at [Validate Data Model by Studio Analysis](../2022-04-23-validate-data-model-by-studio-analysis).

```python
celonis.default_space.packages
```

[![image](https://user-images.githubusercontent.com/67397583/168414778-c211ab89-77c8-4933-ac6a-c3d405aad3de.png)](https://user-images.githubusercontent.com/67397583/168414778-c211ab89-77c8-4933-ac6a-c3d405aad3de.png)

In the next post I will create application and assign appropriate permissions to it.

Kaz
