---
id: messages
title: GET Speech to Text
sidebar_label: GET Speech to Text
slug: /conversation-api/messages
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

The Messages API returns a list of all the messages in a conversation. You can use this for providing **Speech to Text data (also known as transcription sometimes)** for video conference, meeting or telephone call.

Here message refer to a continuous sentence spoken by a speaker.

### Sentiment Analysis in messages <font color="orange"> BETA</font>

Here you can enable sentiment analysis over each message which is being spoken in the conversation.

All you need to do is pass `sentiment=true` as a query parameter. [Read more about it](/docs/concepts/sentiment-analysis).


### HTTP Request

`GET https://api.symbl.ai/v1/conversations/{conversationId}/messages`

### Example API Call



:::info
Before using the Conversation API you must get the authentication token (`AUTH_TOKEN`) from [our authentication process](/docs/developer-tools/authentication).
:::

<Tabs
  defaultValue="cURL"
  values={[
    { label: 'cURL', value: 'cURL', },
    { label: 'Node.js', value: 'nodejs', },
    { label: 'Python', value: 'python' }
  ]
}>
<TabItem value="cURL">

```sh
curl "https://api.symbl.ai/v1/conversations/$CONVERSATION_ID/messages" \
    -H "Authorization: Bearer $AUTH_TOKEN"
```

</TabItem>

<TabItem value="nodejs">

```js
const request = require('request');
const authToken = AUTH_TOKEN;
const conversationId = CONVERSATION_ID;

request.get({
    url: `https://api.symbl.ai/v1/conversations/${conversationId}/messages`,
    headers: { 'Authorization': `Bearer ${authToken}` },
    json: true
}, (err, response, body) => {
    console.log(body);
});
```

</TabItem>
<TabItem value="python">

```py
import requests

baseUrl = "https://api.symbl.ai/v1/conversations/{conversationId}/messages"
conversationId = 'your_conversation_id'  # Generated using Submit text end point

url = baseUrl.format(conversationId=conversationId)

# set your access token here. See https://docs.symbl.ai/docs/developer-tools/authentication
access_token = 'your_access_token'

headers = {
    'Authorization': 'Bearer ' + access_token,
    'Content-Type': 'application/json'
}

params = {
    'verbose': True,  # <Optional, boolean| Gives you word level timestamps of each sentence.>
    'sentiment': True  # <Optional, boolean| Give you sentiment analysis on each message.>
}

responses = {
    401: 'Unauthorized. Please generate a new access token.',
    404: 'The conversation and/or it\'s metadata you asked could not be found, please check the input provided',
    500: 'Something went wrong! Please contact support@symbl.ai'
}

response = requests.request("GET", url, headers=headers)

if response.status_code == 200:
    # Successful API execution
    print("messages => " + str(response.json()['messages']))  # messages is a list of id, text, from, startTime, endTime, conversationId, words, phrases, sentiment
elif response.status_code in responses.keys():
    print(responses[response.status_code])  # Expected error occurred
else:
    print("Unexpected error occurred. Please contact support@symbl.ai" + ", Debug Message => " + str(response.text))

exit()
```
</TabItem>
</Tabs>


### Query Params

Parameter | Required | Value |Description|
--------- | --------- | ------- | -------
```verbose``` | No | true |Gives you word level timestamps of each sentence.
```sentiment```| No | true | Give you [Sentiment Analysis](/docs/concepts/sentiment-analysis) on each message.

### Response

```javascript
{
    "messages": [
         {
             "id": "6412283618000896",
             "text": "Best package for you is $69.99 per month.",
             "from": {
                 "name": "Roger",
                 "email": "Roger@example.com"
             },
             "startTime": "2020-07-10T11:16:21.024Z",
             "endTime": "2020-07-10T11:16:26.724Z",
             "conversationId": "6749556955938816",
             "phrases": [
                {
                    "type": "action_phrase",
                    "text": "$69.99 per month"
                }
             ],
             "sentiment": {
                "polarity": {
                    "score": 0.6
                },
                "suggested": "positive"
              },
              "words": [
                 {
                     "word": "Best",
                     "startTime": "2020-08-18T11:10:14.536Z",
                     "endTime": "2020-08-18T11:10:15.536Z",
                 },
                 {
                     "word": "package",
                     "startTime": "2020-08-18T11:10:16.536Z",
                     "endTime": "2020-08-18T11:10:17.536Z",
                 },
                 {
                     "word": "for",
                     "startTime": "2020-08-18T11:10:18.536Z",
                     "endTime": "2020-08-18T11:10:19.536Z",
                 },
                 {
                     "word": "you",
                     "startTime": "2020-08-18T11:10:20.536Z",
                     "endTime": "2020-08-18T11:10:22.536Z",
                 },
                 {
                     "word": "is",
                     "startTime": "2020-08-18T11:10:22.536Z",
                     "endTime": "2020-08-18T11:10:25.536Z",
                 },
                 {
                     "word": "$69.99",
                     "startTime": "2020-08-18T11:10:25.536Z",
                     "endTime": "2020-08-18T11:10:27.536Z",
                 },
                 {
                     "word": "per",
                     "startTime": "2020-08-18T11:10:27.536Z",
                     "endTime": "2020-08-18T11:10:29.536Z",
                 },
                 {
                     "word": "month.",
                     "startTime": "2020-08-18T11:10:30.536Z",
                     "endTime": "2020-08-18T11:10:32.536Z",
                 }]
          },
         {
             "id": "5661493169225728",
             "text": "Okay, Where is the file?",
             "from": {
                 "name": "John",
                 "email": "John@example.com"
             }
             "startTime": "2020-08-18T11:11:14.536Z",
             "endTime": "2020-08-18T11:11:18.536Z",
             "conversationId": "5139780136337408",
             "phrases": [],
             "sentiment": {
                    "polarity": {
                        "score": 0.2
                    },
                    "suggested": "neutral"
              },
             "words": [
                 {
                     "word": "Okay,",
                     "startTime": "2020-08-18T11:11:14.536Z",
                     "endTime": "2020-08-18T11:11:14.936Z"
                 },
                 {
                     "word": "Where",
                     "startTime": "2020-08-18T11:11:14.936Z",
                     "endTime": "2020-08-18T11:11:15.436Z"
                 },
                 {
                     "word": "is",
                     "startTime": "2020-08-18T11:11:16.236Z",
                     "endTime": "2020-08-18T11:11:16.536Z"
                 },
                 {
                     "word": "the",
                     "startTime": "2020-08-18T11:11:16.536Z",
                     "endTime": "2020-08-18T11:11:16.936Z"
                 },
                 {
                     "word": "file?",
                     "startTime": "2020-08-18T11:11:16.936Z",
                     "endTime": "2020-08-18T11:11:17.236Z"
                 }
    ]
}
```

### Response Object

Field  | Description
---------- | ------- |
```id``` | Unique message identifier.
```text``` | Message text.
```from``` | User object with name and email.
```startTime``` | DateTime value.
```endTime``` | DateTime value.
```conversationId``` | Unique conversation identifier.
```words``` | Words object with properties `word`, `startTime` and `endTime`.
```phrases``` | It shows the most important action phrases in each sentence. It's enabled when you pass `detectPhrases=true` during submiting the request in Async and Websocket API.
```sentiment```| Shows the sentiment polarity(intensity of negativity or positivity of a sentence) and suggested sentiment type (positive, negative and neutral).
