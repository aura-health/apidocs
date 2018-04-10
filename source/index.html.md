---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - javascript
  - ruby
  - java

toc_footers:
  - <a href='mailto:developers@aurahealth.com.br'>Request us for a Developer Key</a>
  - <a href='https://www.aurahealth.com.br'>Documentation Powered by Aura</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to Aura API! You can use our API to access Aura API endpoints, which can start a new Interview session and subscribe to their updates.

We have language bindings in Shell but you can use in all other languages by consuming our HTTPS endpoints! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

This API documentation page was created with [Slate](https://github.com/lord/slate), an open source project used by a lot of big companies to share their documentations.

# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: Bearer YOUR_API_KEY"
```

> Make sure to replace `YOUR_API_KEY` with your API key.

Aura uses API keys to allow access to the API. You can register a new Aura API key by requesting from email [developers@aurahealth.com.br](mailto:developers@aurahealth.com.br).

Aura expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer YOUR_API_KEY`

<aside class="notice">
You must replace <code>YOUR_API_KEY</code> with your personal API key.
</aside>

# Interview Session

The core feature of Aura chatbot is to interview a patient before being taken care of by the Doctor. This endpoints is to start, get information and finish an interview.

The Interview Session data structure will appear several times around here. It's because every time you interact with a session, the response will be the same session but, in its new state. The session data structure is incremental, always!

That's why you'll realize just how simple it is to interact with Aura APIs!

## Start a new Interview Session

```shell
curl -X POST \
  https://api.aurahealth.com.br/v1/interview \
  -H 'Authorization: Bearer YOUR_API_TOKEN' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
    "cellphone": "+5531988880000",
    "company_name": "Clínica High Tech",
    "doctor_name": "Dr. Igor",
    "sync_webhook_url": "https://webhooks.clinicahightech.com.br/aura/patient_id_sample?myparam=sample&foo=bar"
  }'
```

```javascript
const request = require("request");

const options = { method: 'POST',
  url: 'https://api.aurahealth.com.br/v1/interview',
  headers: 
   { 'Cache-Control': 'no-cache',
     Authorization: 'Bearer YOUR_API_TOKEN',
     'Content-Type': 'application/json' },
  body: 
   { cellphone: '+5531988880000',
     company_name: 'Clínica High Tech',
     doctor_name: 'Dr. Igor',
     sync_webhook_url: 'https://webhooks.clinicahightech.com.br/aura/patient_id_sample?myparam=sample&foo=bar' },
  json: true };

request(options, (error, response, body) => {
  if (error) throw new Error(error);

  console.log(body);
});
```

```ruby
require 'uri'
require 'net/http'

url = URI("https://api.aurahealth.com.br/v1/interview")

http = Net::HTTP.new(url.host, url.port)

request = Net::HTTP::Post.new(url)
request["Content-Type"] = 'application/json'
request["Authorization"] = 'Bearer YOUR_API_TOKEN'
request["Cache-Control"] = 'no-cache'
request.body = "{\n\t\"cellphone\": \"+5531988880000\",\n\t\"company_name\": \"Clínica High Tech\",\n\t\"doctor_name\": \"Dr. Igor\",\n\t\"sync_webhook_url\": \"https://webhooks.clinicahightech.com.br/aura/patient_id_sample?myparam=sample&foo=bar\"\n}"

response = http.request(request)
puts response.read_body
```

```java
OkHttpClient client = new OkHttpClient();

MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "{\n\t\"cellphone\": \"+5531988880000\",\n\t\"company_name\": \"Clínica High Tech\",\n\t\"doctor_name\": \"Dr. Igor\",\n\t\"sync_webhook_url\": \"https://webhooks.clinicahightech.com.br/aura/patient_id_sample?myparam=sample&foo=bar\"\n}");
Request request = new Request.Builder()
  .url("https://api.aurahealth.com.br/v1/interview")
  .post(body)
  .addHeader("Content-Type", "application/json")
  .addHeader("Authorization", "Bearer YOUR_API_TOKEN")
  .addHeader("Cache-Control", "no-cache")
  .build();

Response response = client.newCall(request).execute();
```

> The above command returns JSON structured like this:

```json
{
  "id": "7gHyd6f31",
  "status": "pristine",
  "chat_url": "https://app.aurahealth.com.br/s/7gHyd6f31"
}
```

This endpoint start a new interview session in ``pristine`` state.

<aside class="notice">
<strong>Pristine</strong> is the untouched/first state of an interview.
</aside>

### HTTP Request

`POST https://api.aurahealth.com.br/v1/interview`

### Body Parameters

Parameter | Type | Description
--------- | ------- | -----------
cellphone | ``String`` | ***Required*** to send SMS message to interviewee with chat link. The pattern is ``+5531988880000``.
company_name | ``String`` | ***Optional*** in case you want to label the interview with your Company Name.
doctor_name | ``String`` | ***Optional*** if you want ensure the Doctors name to build confidence with interviewee.
sync_webhook_url | ``String`` | ***Optional*** if you want to subscribe on changes, set here your webhook HTTPS URL and you will receive a payload at every change on Interview state, as [described below](#sync-webhook-of-interview).
medical_report_format | ``String`` | ***Optional (Default: markdown)*** Available options for Medical Report format: ``"markdown"``, ``"plain_text"``, ``"html"``
interview_language | ``String`` | ***Optional (Default: en)*** Available options for the Interview language: ``"en"``, ``"pt"``
medical_report_language | ``String`` | ***Optional (Default: en)*** Available options for the Medical Report language: ``"en"``, ``"pt"``

<aside class="success">
Our Sync Webhook is great for simplifying your implementation! We strongly recommend their usage!
</aside>

## Sync Webhook of Interview

When you [start a new Interview Session](#start-a-new-interview-session), you can set up a webhook to subscribe on Interview state changes. It is useful if you want to simplify your implementation without loose the power of Aura data at the right time.

All of Interview session data is incrementally retrieved based on conversation of Aura chatbot with interviewee. Based on that, we will describe [full json data structure](#interview-structure-state) below.

## Interview Structure State

You will deal with this structure at every response from Interview API.

> Full Interview data structure:

```json
{
  "id": "9DIGITSID",
  "status": "answering",
  "chat_url": "https://app.aurahealth.com.br/s/9DIGITSID",
  "medical_report": "
    ## Aura Medical Report

    **Interviewee**: Full Name of Interviewee, 25 yo, male.

    **Evidences**:
      * **Headache** pulsing with high intensity
      * **Abdominal pain** ...
  "
}
```

Attribute | Description
--------- | -------
id | ***String*** 9 digits id that identifies the Interview session
status | ***String*** Statuses list: ``"pristine"``, ``"answering"``, ``"inactive"``, ``"finished"``
chat_url | ***String*** Link that interviewee interact with our chatbot.
medical_report | ***String*** The medical report formatted accordingly to [start params](#start-a-new-interview-session). The **default** format is ``"markdown"``.

## Get a Specific Interview state

```shell
curl -X GET \
  https://api.aurahealth.com.br/v1/interview/9DIGITSID \
  -H 'Authorization: Bearer YOUR_API_TOKEN' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json'
```

```javascript
const request = require("request");

const options = { method: 'GET',
  url: 'https://api.aurahealth.com.br/v1/interview/9DIGITSID',
  headers: 
   { 'Cache-Control': 'no-cache',
     Authorization: 'Bearer YOUR_API_TOKEN',
     'Content-Type': 'application/json' } };

request(options, (error, response, body) => {
  if (error) throw new Error(error);

  console.log(body);
});

```

```ruby
require 'uri'
require 'net/http'

url = URI("https://api.aurahealth.com.br/v1/interview/9DIGITSID")

http = Net::HTTP.new(url.host, url.port)

request = Net::HTTP::Get.new(url)
request["Content-Type"] = 'application/json'
request["Authorization"] = 'Bearer YOUR_API_TOKEN'
request["Cache-Control"] = 'no-cache'

response = http.request(request)
puts response.read_body
```

```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
  .url("https://api.aurahealth.com.br/v1/interview/9DIGITSID")
  .get()
  .addHeader("Content-Type", "application/json")
  .addHeader("Authorization", "Bearer YOUR_API_TOKEN")
  .addHeader("Cache-Control", "no-cache")
  .build();

Response response = client.newCall(request).execute();
```

> The above command returns JSON structured like this:

```json
{
  "id": "9DIGITSID",
  "status": "inactive",
  "chat_url": "https://app.aurahealth.com.br/s/9DIGITSID",
  "medical_report": "
    ## Aura Medical Report

    **Interviewee**: Full Name of Interviewee, 25 yo, male.

    **Evidences**:
      * **Headache** pulsing with high intensity
      * **Abdominal pain** ...
  "
}
```

This endpoint retrieves a specific Interview Session current state.

### HTTP Request

`GET https://api.aurahealth.com.br/v1/interview/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the Interview Session to retrieve

## Finish a Specific Interview

```shell
curl -X DELETE \
  https://api.aurahealth.com.br/v1/interview/9DIGITSID \
  -H 'Authorization: Bearer YOUR_API_TOKEN' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json'
```

```javascript
const request = require("request");

cpnst options = { method: 'DELETE',
  url: 'https://api.aurahealth.com.br/v1/interview/9DIGITSID',
  headers: 
   { 'Cache-Control': 'no-cache',
     Authorization: 'Bearer YOUR_API_TOKEN',
     'Content-Type': 'application/json' } };

request(options, (error, response, body) => {
  if (error) throw new Error(error);

  console.log(body);
});
```

```ruby
require 'uri'
require 'net/http'

url = URI("https://api.aurahealth.com.br/v1/interview/9DIGITSID")

http = Net::HTTP.new(url.host, url.port)

request = Net::HTTP::Delete.new(url)
request["Content-Type"] = 'application/json'
request["Authorization"] = 'Bearer YOUR_API_TOKEN'
request["Cache-Control"] = 'no-cache'

response = http.request(request)
puts response.read_body
```

```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
  .url("https://api.aurahealth.com.br/v1/interview/9DIGITSID")
  .delete(null)
  .addHeader("Content-Type", "application/json")
  .addHeader("Authorization", "Bearer YOUR_API_TOKEN")
  .addHeader("Cache-Control", "no-cache")
  .build();

Response response = client.newCall(request).execute();
```

> The above command returns JSON structured like this:

```json
{
  "id": "9DIGITSID",
  "status": "finished",
  "chat_url": "https://app.aurahealth.com.br/s/9DIGITSID",
  "medical_report": "
    ## Aura Medical Report

    **Interviewee**: Full Name of Interviewee, 25 yo, male.

    **Evidences**:
      * **Headache** pulsing with high intensity
      * **Abdominal pain** with low intensity, appearing after eat
  "
}
```

This endpoint finishes a specific Interview Session.

### HTTP Request

`DELETE https://api.aurahealth.com.br/v1/interview/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the Interview Session to finish

