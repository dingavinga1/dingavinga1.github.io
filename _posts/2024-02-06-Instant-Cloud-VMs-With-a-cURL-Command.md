---
title: Instant Cloud VMs With a cURL Command
date: 2024-02-06 12:29:00 +0500
categories: [Blogs, Cloud Computing]
tags: [cloud computing, cloud, public cloud, IaaS, google cloud, cloud functions, lambda functions]
---

![Google Cloud Functions](https://miro.medium.com/v2/resize:fit:720/format:webp/1*KG0wy_jsLaIAQONWKS0_9g.jpeg)

Cloud computing has been around for a while, and we all know its basics. Some things are easy to navigate around, and some aren’t, but I think we should all agree upon the fact that the consoles provided by Google, Azure and AWS are a bit disappointing when it comes to UX. Google is a king when it comes to the tech-verse but somehow, they managed to mess up web socket integration in their cloud portal.

Let me give an example. Go to [console.cloud.google.com](http://console.cloud.google.com) and try to create a new Compute Engine instance. If you don’t keep reloading the page, the console won’t tell you that your instance is running. This is probably because the state isn’t being reflected in real-time. Even though this is the most nitpicky example to ever exist, there are multiple other areas where the console disappoints but let’s not get into those.

Being a perfectionist, I Googled up a solution for this (ironic, isn’t it?) and came to know about [Cloud Functions](https://cloud.google.com/functions?hl=en)! Yes, serverless compute might just be able to help you with your problem but let’s first see what these functions really are:

> Run your code in the cloud with no servers or containers to manage with our scalable, pay-as-you-go functions as a service (FaaS) product.

This is how Google defines cloud functions but here is what they actually are:

*   Google Cloud Functions are mini-programs that run in the cloud.
*   They respond to events triggered by other Google Cloud services or HTTP requests.
*   You write code to define what happens when these events occur.
*   They scale automatically, meaning they can handle varying workloads without you needing to manage resources.

Yeah… you still don’t get it do you? No judgement, I wouldn’t have either if someone pasted a ChatGPT answer in 4 bullet points. Google Cloud Functions, in the simplest terms possible, allow us to write and run code, in the cloud, without the need to manage or run a server, while heavily integrating with other Google Cloud services.

![Cloud Services](https://miro.medium.com/v2/resize:fit:720/format:webp/1*wFA_0I7Ve1dUHs3hQQdoSQ.png)

## Advantages

With Cloud Functions, you get:

*   **Scalability**: Scale you function endlessly without the need for manual intervention or even a load-balancing setup.
*   **Cost Effectiveness**: Pay per request i.e. you only pay a small amount per hit/call on your function.

## Disadvantages

If you are like me, this must’ve got you thinking that you can host your entire business using Cloud Functions and hey, maybe you can. But first, there are a few things that you cannot do with Cloud Functions:

*   **10 MB** limit on both HTTP request and response sizes.
*   Maximum **32 GB** memory available to a single cloud function
*   **500 MB** source size, including modules.

Now that we’ve covered the basics, how exactly can cloud functions help us get rid of our UX complaints? You might have guessed by now, but if you haven’t, here’s how- by creating your own REST API that helps you start and stop Compute Engine instances! Build your dream UI and integrate Google Cloud into it. Manage your instances without the need to reload the console page!

I don’t know if this classifies as click-bait or not but if you’re here for a tutorial on creating an entire custom web console for Google Cloud, you might be in for a disappointment. I’m just here to give you the Cloud Functions part; the rest is on you. Now, without further ado, let’s dive into how you can create/start/stop VMs, in the cloud, through a cURL command!

## How do we create a Cloud Function?

*   Navigate to [console.cloud.google.com/functions/list](http://console.cloud.google.com/functions/list).
*   Click on the **Create Function** button on the top left.
*   Give your function a name, choose your region and choose the computational limits you want to apply.
*   Click next!

## What now?

Google Cloud Functions allow us to use various programming languages but I, for the purpose of this tutorial, will be using **Python 3.10.**

First of all, we need go ahead and edit the **requirements.txt** file:

```
functions-framework==3.*  
google-api-python-client==1.10.0
```

We need to add the second line to make sure we have the required **pip** module to interact with Google Cloud. Once we’re done with the pre-requisites, we can move on to writing our code in the **main.py** file.

Here’s what we need to start with:

```python
@functions_framework.http  
def hello_http(request):  
    """  
    Driver for cloud function  
    """  
  
    if request.method == 'OPTIONS': # letting CORS know that it's okay to hit GET/POST on this endpoint  
        headers = {  
            'Access-Control-Allow-Origin': '*',  
            'Access-Control-Allow-Methods': 'POST',  
            'Access-Control-Allow-Headers': 'Content-Type',  
            'Access-Control-Max-Age': '3600'  
        }  
  
        return ('', 204, headers)   
  
    headers = { # headers to attach to each response we send  
        'Access-Control-Allow-Methods': 'POST',  
        'Access-Control-Allow-Origin': '*'  
    }  
  
    if request.json['operation']=='START':  
        response = start_vm(request.json['id'], headers)  
    elif request.json['operation']=='STOP':  
        response = stop_vm(request.json['id'], headers)  
    else   
        response = ('Invalid Operation', 400, headers)  
  
    return response
```

An apology in advance for the lack of error handling or any bad practices. You can modify the code according to your requirements as this blog is just to provide you with a basic idea of how everything works. In the driver function, we basically added a few formalities to cater to the “Same origin policy” that our browsers are so fond of (kidding, they’re there for our security).

For interacting with our compute instances, we can use the [Google Cloud Compute SDK Reference](https://googleapis.github.io/google-api-python-client/docs/dyn/compute_v1.html). However, here’s the code for starting and stopping a particular instance.

```python
def start_vm(id, headers):  

    project_id = '<YOUR PROJECT ID HERE>'  
    zone_name = '<THE ZONE OF YOUR CHOICE HERE>'  

    compute = googleapiclient.discovery.build('compute', 'v1')  

    _ = compute.instances().start(project=project_id, zone=zone_name, instance=vm_name).execute()  
    instance_details = compute.instances().get(project=project_id, zone=zone_name, instance=id).execute()  

    return (instance details, 200, headers)  
  
  
def stop_vm(id, headers):  
  
    project_id = '<YOUR PROJECT ID HERE>'  
    zone_name = '<THE ZONE OF YOUR CHOICE HERE>'  

    compute = googleapiclient.discovery.build('compute', 'v1')  

    _ = compute.instances().stop(project=project_id, zone=zone_name, instance=vm_name).execute()  


    return ('', 204, headers)
```

For the above code to work, you need to add your own project id and zone in which your instances are. Given that, if you hit test function and hit the function with the following request, you should see your instance started in the GCP Console.

```
{  
  "operation": "START",  
  "id": <NAME OF YOUR INSTANCE HERE>  
}
```

Now, if you click on **DEPLOY** and try to run the same request with a cURL command, it won’t work; that’s because we haven’t allowed our function to be accessed publicly. To do that, we need to do the following:

*   Go to [https://console.cloud.google.com/run](https://console.cloud.google.com/run)
*   Select the newly created function
*   Click on **Permissions**
*   Click on **Add Principal** and select the “allUsers” principal
*   Assign it the following role: **Cloud Run Invoker**

Now you’re good to go! Use the following cURL command to start your cloud instances:

```
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{  
        "operation": "START",  
        "id": "<NAME OF YOUR INSTANCE HERE>"  
      }' \  
  https://<URL OF YOUR CLOUD FUNCTION>
```

## Goodbye

That’s it for this blog. I hope it helps everyone who is new to Google Cloud and wants to play around with the Compute SDK!
