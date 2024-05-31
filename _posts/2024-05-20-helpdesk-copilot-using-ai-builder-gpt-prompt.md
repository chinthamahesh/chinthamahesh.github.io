---
title: "Helpdesk Copilot Using AI Builder GPT Prompt"
last_modified_at: 2024-05-20T16:20:02-05:00
categories:
  - Blog
tags:
  - Microsoft Copilot
  - Copilot Studio
  - Open AI GPT
  - AI Builder
  - Power Automate
  - Prompt Engineering
---

IT Helpdesk custom Copilot using Copilot Studio and AI Builder GPT Model:


In this post we see how to create a custom copilot which acts as IT Helpdesk support using Copilot Studio and Power Platform AI Builder 'Create text with GPT using a Prompt'. In this case, copilot will generate solutions for end users or agents to support with IT issues using GPT model. 

This copilot can be published and made it available to users across organization. Please refer the previous blog posts on how to publish a copilot to MS Teams or SharePoint site.

## Power Platform AI Builder:

AI Builder is a Microsoft Power Platform feature that allows you to create and use AI models for your business needs, AI Builder offers a range of AI capabilities without requiring extensive coding or data expertise.

## Create custom Copilot using Microsoft Copilot Studio:

We now navigate to Copilot Studio - https://copilotstudio.microsoft.com and create a custom copilot named 'IT Helpdesk Copilot'


## Create a Power Automate flow from the Copilot Studio:

Navigate to 'Conversation Start' topic -> Add an Action -> Call an Action -> Create a flow

Within the flow, Add a Node -> Select/Search 'AI Builder' and select 'Create text with GPT using a prompt'



Power Automate flow will look like below
                                                   
Add an input parameter to Power Automate flow 'Run a flow from Copilot' node. This is an input parameter to the flow which will be sent from Copilot which is an question or issue description asked by the end user

## Create a Custom Prompt:

Add a prompt and create an input variable as 'issuedescription' in the custom prompt. This variable will be used in the Prompt. Refer the IT Expert prompt from the Github Power Platform prompt library repo. Save the custom prompt

Add output parameter to the flow to GPT response 'Text'. This will be the GPT response for the question/issue asked by the end user

## Call Power Automate flow from Copilot: 

Call the flow created from Copilot Studio from 'Conversation Start' topic. You may need to refresh the copilot studio to show up the newly created flow
                                          
## Publish and test the copilot:

Here is the response generated using the GPT for an issue posted by end user.

