# What is crAPI?
crAPI (Completely Ridiculous API) defines an API which is intentionally vulnerable to the OWASP API Top 10 vulnerabilities. It simulates an API-driven, microservice-based web application that is a platform for vehicle owners. 

crAPI can be tested using docker, vagrant or other deployement options.

For more informations check this repo by the OWASP team : [crAPI](https://github.com/OWASP/crAPI)

>After completing my preparation for the ACP certification, I started looking for a vulnerable application to test the knowledge I had acquired before taking the exam.
>Fortunately, I came across crAPI, which helped me a lot by allowing me to practice in real-world scenarios.
# The Action
## Recon
Every Penetration testing typically begins with the reconnaissance phase. In the case of API pentest, reviewing the available documentation is often an effective starting point. Many API providers publish detailed specifications using OpenAPI to support developers and consumers. So, API documentation is frequently accessible and can offer valuable insights for testing.

In the case of crAPI, a JSON file containing the OpenAPI documentation for all API endpoints can be found in the crAPI repository. This file can be imported into the Swagger Editor to generate a fancy documentation of the api.

Here is a list of all available endpoints.

![to](assets/images/endpoints.png)

This is all the methods associated with the auth endpoint

![to](assets/images/methods_auth.png)

The documentation also includes detailed information on request and response formats.

![to](assets/images/param_auth.png)

![to](assets/images/response_auth.png)

We’ll refer to this documentation at different stages during the API's exploitation

# Challenges

There are two approaches to hack crAPI - the first is to look at it as a complete black box test, where you get no directions, but just try to understand the app from scratch and hack it. The second approach is using this page, which will give you an idea about which vulnerabilities exist in crAPI and will direct you on how to exploit them.

Due to time constraints, I decided to tackle the API using the second approach. The exploitation is therefore broken down into challenges, each highlighting the presence of a vulnerability from the OWASP Top 10.

## BOLA Vulnerabilities
BOLA or Broken object level authorization is a vuln that occurs when an API does not properly verify whether the user is authorized to access a specific object or resource
### **Challenge 1** - Access details of another user’s vehicle

We need to leak sensitive information of another user’s vehicle.

![to](assets/images/login.png)

After creating an account, I log in to the app to get the following page

![to](assets/images/dashboard.png)

I try to add vehicle, but first I need to get the vehicle details sent to my email address in the mailHog web portal running in the port 8025

![to](assets/images/mail.png)

After adding the vehicle, I notice a refresh button by inspecting it using burpsuite, I found a call to an api that receives a vehicle GUID and returns information about it. 

![to](assets/images/refresh.png)

![to](assets/images/vuln1.png)

My next step is to figure out how to access other users’ GUIDs. In the community section, people publish posts, and when the app fetches these posts, it calls an API that, as revealed by Burp Suite, returns extra information, including the vehicle ID

![to](assets/images/vuln2.png)

We manage to get sensitve informations about other users' car

![to](assets/images/vuln3.png)

### **Challenge 2** - Access mechanic reports of other users

crAPI allows vehicle owners to contact their mechanics by submitting a “contact mechanic” form. The goal is to access others' reports.

When attempting to submit a report, a POST request is made to the /workshop/api/merchant/contact_mechanic endpoint, with all the report details included in the request body. The interesting part lies in the response, which contains a link to the submitted report along with its ID. I then sent a request to this endpoint using id=2 which give me access to report of other user.

![to](assets/images/report.png)

![to](assets/images/report2.png)

