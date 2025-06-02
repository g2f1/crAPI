# What is crAPI?
crAPI (Completely Ridiculous API) defines an API which is intentionally vulnerable to the OWASP API Top 10 vulnerabilities. It simulates an API-driven, microservice-based web application that is a platform for vehicle owners. All the challenges in crAPI are based on real-life vulnerabilities that were found in APIs of big companies like Facebook, Uber, and Shopify.

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

## Broken User Authentication

It refers to vulnerabilities that allow attackers to compromise authentication mechanisms, enabling them to impersonate users or gain unauthorized access. This can happen due to issues like:
- Weak or missing authentication checks
- Poor session management
- Insecure token handling

### **Challenge 3** - Reset the password of a different user

In this challenge we need to find an endpoint that make users change their password.

On the login page, there is a feature that allows users to reset their password using their email address. However, the issue is that the application does not require re-authentication when users attempt to change their current password. I used a previously discovered email address to initiate a password reset. When doing so, the application prompts for a one-time password (OTP) sent to the owner's email.

![to](assets/images/auth1.png)

After that the app make a call to /identity/api/auth/v3/check-otp with the OTP, email and new password in the body of the request.

![to](assets/images/auth2.png)

I initially attempted to brute-force the OTP, but the application seemed to restrict the number of attempts. So, I decided to check the documentation and discovered that there is a version 2 of the endpoint that verifies the OTP. I then tried the brute-force attack again using this new endpoint, and it worked.

![to](assets/images/auth3.png)

![to](assets/images/auth4.png)

## Excessive Data Exposure

Excessive data exposure occurs when APIs expose all object properties to API calls rather than what the user needs to act on without considering the object’s sensitivity level.

### **Challenge 4** - Find an API endpoint that leaks sensitive information of other users

The goal is to find an API endpoint that reveals more informations than it should. I remember that I previously used the API endpoint that fetches post data to retrieve the carID. This makes it the likely vulnerable API, as it exposes more information than what the user actually needs.

![to](assets/images/excessive.png)

### **Challenge 5** - Find an API endpoint that leaks an internal property of a video

Every user can add a video to their profile. When a user uploads a video, an API call is made to /identity/api/v2/user/videos. The response includes the video’s ID, name, and data. However, the most interesting—and concerning—part is the conversion_params field, which contains the value "-v codec h264". This is sensitive information that should not be exposed to the user, as it reveals internal configuration details about how the server processes uploaded video content.

![to](assets/images/excessive2.png)
