---
title: "Limit permissions of API token to minimize risk"
date: 2022-05-21T09:31:32+09:00
draft: false
tags: ['Machine Learning', 'Admin and Settings']
---

At last post [Login to Celonis EMS from Jupyter Workbench](../2022-05-14-login-to-celonis-ems-from-jupyter-workbench), I used API key that have same permission as my GUI user. I mentioned that it is too strong and risky against unauthorized access. Imagine your API token accidentally make public, then anyone can operate Celonis instead of you. That is why I segregate API token from Notebook (Notebook may be published to GitHub etc.). Anyway, user API token must be altered to another weaker key especially in production system. 

First step is to access to `Admin and Settings`. If you are using training environment as me, you can see this button because you are admin user. Normally you cannot see `Admin and Settings` if you are not admin user.

[![image](https://user-images.githubusercontent.com/67397583/169627997-2e767f3a-7f40-45ac-8f76-8b73de954d43.png)](https://user-images.githubusercontent.com/67397583/169627997-2e767f3a-7f40-45ac-8f76-8b73de954d43.png)

In the `Admin and Settings`, find `Applications` menu that is target for today. Click `New Application Key` and type name for our purpose. Then I can see Appication key (API token) so copy it to text editor etc. If this key is accidentally published to public space, then click three dots of this Application then `Delete`. This token is not used anymore.

Also click three dots and `View Permissions`. For the moment No permissions found is displayed. So currently this token is useless. So next step is adding permissions to this token. 

In the `Admin and Settings`, click `Permissions` menu then all Celonis EMS modules are displayed.

[![image](https://user-images.githubusercontent.com/67397583/169628815-1775ff9f-902b-4d6b-b63e-124a8f8edda0.png)](https://user-images.githubusercontent.com/67397583/169628815-1775ff9f-902b-4d6b-b63e-124a8f8edda0.png)

For example, click `Data Integration` and go into detail setting. If I would like to handle all of existing Pools from this API token but no need to create new Pool, untick `CREATE DATA POOL` and tick the rest. 

[![image](https://user-images.githubusercontent.com/67397583/169629061-bf371670-e52f-4953-bd84-2db557dc40b4.png)](https://user-images.githubusercontent.com/67397583/169629061-bf371670-e52f-4953-bd84-2db557dc40b4.png)

Or if I would like to limit to only one Pool from this API token, go to `Data Integration` and click three dots of relevant Pool, then tick required permissions as below. 

[![image](https://user-images.githubusercontent.com/67397583/169629286-0af0d303-254c-44ba-99dd-e01fc8565aed.png)](https://user-images.githubusercontent.com/67397583/169629286-0af0d303-254c-44ba-99dd-e01fc8565aed.png)

Please be careful that if both All Pool permissions and One Pool permissions are set, first one prioritize second one.

After that go back to `Applications` and see `View Permissions`. I can review which permissions is assigned to this API token. 

OK, let's use new API token (Application key) in Notebook. Change two points from last post's script.
- File to store API token is changed to `app.json`, and create relevant file that store Application key noted before.
- when calling `get_celonis`, second argument key_type is changed to `KeyType.APP_KEY` to use Application key.

If working correctly, output log is like below.
```python
[2022-05-21 01:42:09] INFO: Initial connect successful! You are using an Application Key. PyCelonis Version: 1.7.0
[2022-05-21 01:42:10] INFO: Your key has following permissions:
[
    {
        "permissions": [],
        "serviceName": "package-manager"
    },
    ...
    {
        "permissions": [
            "USE_ALL_DATA_MODELS",
            "EDIT_ALL_DATA_POOLS"
        ],
        "serviceName": "event-collection"
    },
    ...
]
```

And I can confirm I cannot access to packages by `celonis.default_space.packages` like last post.
```python
...
PyCelonisNotFoundError: No result found for Default.
```

To minimize risk of unauthorized access, I recommend to understand today's topic well. By the way, as test purpose I do not want to raise error every time for explaining functions, so I use full permissions to my API token in this blog posts after that.

Kaz
