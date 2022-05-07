---
title: "Start Deep Dive to Machine Learning and Action Flow"
date: 2022-05-07T12:54:54+09:00
draft: false
tags: ['Machine Learning', 'Studio','Action Flow']
---

Until last post, I have seen the flow of process discovery through Celonis Process Analytics (or Studio Analysis) and prerequisite ETL (Extraction, Transformation, Load) by Data Integration. I already worked in multiple static process mining projects and found that they are enough functions for static process mining. And other process mining solutions are provided these functions too.

By the way, referring to [Process Mining Data Science in Action by Wil van der Aalst](https://link.springer.com/book/10.1007/978-3-662-49851-4), Process Mining can also make it possible for further actions such as monitoring and predictive analysis. Celonis EMS is ready for these further actions by Machine Learning and Action Flow components I think.

Machine Learning component is not only in the sense of the word, but also Python execution environment powered by [JupyterLab](https://jupyterlab.readthedocs.io/en/stable/) and [Pycelonis](https://celonis.github.io/pycelonis). This component allows me to extend process mining functions in some extent of frexibility. For example, 
- integrate data that is not feasible for Extractor Builder I introduced before
- integrate file based data 
- generate new data calculated by machine learning algorithm
- manipulate Celonis EMS functions from outside of it.

On the other hand, Action Flow (customized version of [Integromat / Make](https://www.make.com/)) component enables me to manipulate and integrate not only Celonis EMS, but almost all SasS solutions with easier development knowledge and relatively higher frexibility. 

I guess these components confuse you because of their high degree of freedom, also require some knowledge of current internet world's programming, e.g. REST API and prerequisite HTTP request/response. When I first tried these functions, I am struggled with the freedom of choosing these components, then finally I can use these components to automate my own work, and I have chance to train them to my colleagues. Now some of my colleagues can utilize these components too. I believe they are ideal components to solve individual (local) requirements in your organization, so I strongly recommend to try them.

Based on the training above, due to your knowledge base is broader, I think it is difficult to write about these components in a series of posts. But I will try my best to explain it step by step. In addition, since Machine Learning and Action Flow are interrelated for me, I will use whichever is necessary in future posts.

Finally, I would like to comment to other Celonis components, Action Engine, and Skill (previously Automation). I believe both are older version of Action Flow and recommend not to take much time to learn them. In this blog, normally I do not mention about them. But I sometimes deal with Skill in case Action Flow does not have function that Skill has. 

Kaz