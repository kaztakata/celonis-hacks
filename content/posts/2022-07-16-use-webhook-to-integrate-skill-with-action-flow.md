---
title: "Use Webhook to Integrate Skill with Action Flow"
date: 2022-07-16T09:49:00+09:00
draft: false
tags: ['Studio', 'Action Flow', 'Data Integration']
---

### Email from Celonis EMS 
Until last post [Aggregate Bundles to Minimize Notification](../2022-07-09-aggregate-bundles-to-minimize-notification), first Action Flow scenario is developed as draft. Today I would like to improve notification part of scenario. 

Few days after I started scenario, I found that `Sent` box in my email account is unnecessarily increased due to notification I developed. That is because I used my email account to send it. If this is personal development that is fine, but if it is official one and in some time later you will leave your position, this email account is not available. So that I would like to use system email account instead of personal one.

I found that Action Flow does not have functionarity to do that, but Skill has. Skill is similar function developed by Celonis and its development is almost stopped due to aquiring Action Flow, but there are few functions that is only in Skill. I would like to integrate Action Flow and Skill to replace notification part of my scenario.

### Create Webhook as first step of Skill

OK, I need one more node in my package, so click plus button in package then create skill, then enter its name and key to save it. In the blank cambus I will choose Sensor, means first step of this flow, as Webhook > Catch Hook. [Webhook](https://en.wikipedia.org/wiki/Webhook) is trigger point constructed by HTTP request. From anywhere else I send to HTTP request to URL registered for Webhook then Webhook is triggered to process the following operations.

After that I will add new input field for this Webhook. For the moment I need `to`, `subject` and `body` parameters for following email function.

[![image](https://user-images.githubusercontent.com/67397583/179325285-0c31c420-e21f-435c-a02d-05ba1202b242.png)](https://user-images.githubusercontent.com/67397583/179325285-0c31c420-e21f-435c-a02d-05ba1202b242.png)

### Add following email step

Click `Add Next Step` in the cambus, then choose Celonis > Email by Celonis. As below screen, set corresponding variables in Webhook.

[![image](https://user-images.githubusercontent.com/67397583/179325644-d8ed23d3-f28e-42f6-a7e3-f0182f92409a.png)](https://user-images.githubusercontent.com/67397583/179325644-d8ed23d3-f28e-42f6-a7e3-f0182f92409a.png)

Let's test this Skill. Copy Webhook URL like below, change parameters accoringly (without space for the moment), then paste to your URL bar in browser and input Enter button. 

```
https://your_email.training.celonis.cloud/process-automation-v2/api/public/skills/your_skill_id/hook?to=your_email&subject=Test&body=test
```

Then I can receive email from `noreply@celonismail.com` that is Celonis system.

### Add authorization to Skill

Because Webhook is using public URL, I have risk that someone else is getting Webhook URL then use Skill. To prevent this I will add one more step to validate request. 

I will add one more parameter `authorization` in Webhook. Next I will add step between Webhook and Email, and choose System Actions > Filter. In Filter condition, compare authorization parameter with arbitrary long password like below. To do this, if someone else knows URL, that poeson also need to know password to use Skill.

[![image](https://user-images.githubusercontent.com/67397583/179326625-bf02c785-c0cf-428b-ac09-6847f082f7f8.png)](https://user-images.githubusercontent.com/67397583/179326625-bf02c785-c0cf-428b-ac09-6847f082f7f8.png)

### Integrate Skill with Action Flow

Finally I would like to replace email module in Action flow with HTTP request for Webhook. Delete email module and create new HTTP request module. Instead of previous test, this time URL parameter is not used but use JSON body to hide authorization. So I copy URL until `?` to HTTP module. Request content is JSON body like below. This content should follow JSON format, so line feed or `"` in each value (especially body from previous module) is not allowed.

```json
{
"to":"your_email",
"subject":"Data Job Monitor",
"body":"{{17.text}}",
"authorization":"your_password"
}
```

[![image](https://user-images.githubusercontent.com/67397583/179327196-5133f0de-17ef-41c4-bcc8-2353ac32a184.png)](https://user-images.githubusercontent.com/67397583/179327196-5133f0de-17ef-41c4-bcc8-2353ac32a184.png)

After replacing module, I can run once then see if this Action Flow and following Skill are working correctly. 

Like today's topic, Webhook is the convenient tool to integrate your action flows / skills with another tools. For example, I setup Webhook to my GitHub account to notify my when our GitHub repository is updated.

Kaz
