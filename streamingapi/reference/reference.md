---
id: api-reference
title: Streaming API Reference
slug: /streaming-api/api-reference
sidebar_label: Request Parameters
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


## Request Parameters

#### Endpoint

`wss://api.symbl.ai/v1/realtime/insights/MEETING_ID?access_token=ACCESS_TOKEN`


#### <a name="message-body"></a>Main Message Body

Field  | Required | Supported Value | Description
---------- | ------- |  ------- |  -------
```type``` | true | start_request, stop_request | Type of message
```insightTypes``` | false | action_item, question | Types of insights to return. If not provided, no insights will be returned.
```config``` | false | | Configuration for this request. [See the config section below for more details](#config).
```speaker``` | false  | | Speaker identity to use for audio in this WebSocket connection. If omitted, no speaker identification will be used for processing. [See the speaker section below for more details])(#speaker).

##### Code Example

```js
{
  "type": "start_request",
  "insightTypes": ["question", "action_item"],
  "config": {},  // See Config section below.
  "speaker": {}  // See Speaker section below.
}
```


#### <a name="config"></a>Config

Field | Required | Supported value | Default Value | Description
---------- | ------- | ------- |  ------- |  ------- |
```confidenceThreshold``` | false  | 0.0 - 1.0 | 0.5 | Minimum Confidence score that should be met for API to consider it as valid insight, if not provided defaults to 0.5 i.e. 50% or more
```languageCode``` | false | | en-US | The language code as per the BCP 47 specification
```speechRecognition``` | false | | | Speaker identity to use for audio in this WebSocket connection. If omitted, no speaker identification will be used for processing. See below.
```meetingTitle``` | false | | | The name of the meeting.

##### <a name="config-example"></a>Code Example

```js
{
  "config": {
    "confidenceThreshold": 0.9,
    // "timezoneOffset": 480, // Your timezone offset from UTC in minutes
    "languageCode": "en-US",
    "speechRecognition": {}  // See Speech Recognition section below.
    "meetingTitle": "Client Meeting"
  }
}
```


####  <a name="speech-recognition"></a>Speech Recognition

Field | Required | Supported value | Default Value | Description
---------- | ------- | ------- |  ------- |  -------
```encoding``` | false  | LINEAR16, FLAC, MULAW | LINEAR16 | Audio Encoding in which the audio will be sent over the WebSocket.
```sampleRateHertz	``` | false  |  | 16000 | The rate of the incoming audio stream.


##### Code Example

```js
{
  "speechRecognition": {
    "encoding": "LINEAR16",
    "sampleRateHertz": 44100 // Make sure the correct sample rate is provided for best results
  }
}
```

####  <a name="speaker"></a>Speaker

Field  | Required | Supported Value
---------- | ------- |  -------
```userId``` | false | Any user identifier for the user.
```name``` | false | Display name of the user.

##### Code Example

```js
{
  "speaker": {
    "userId": "jane.doe@example.com",
    "name": "Jane"
  }
}
```

### Full Code Example

```js
{
  "type": "start_request",
  "insightTypes": ["question", "action_item"],
  "config": {
    "confidenceThreshold": 0.9,
    // "timezoneOffset": 480, // Your timezone offset from UTC in minutes
    "languageCode": "en-US",
    "speechRecognition": {
      "encoding": "LINEAR16",
      "sampleRateHertz": 44100 // Make sure the correct sample rate is provided for best results
    },
    "meetingTitle": "Client Meeting"
  },
  "speaker": {
    "userId": "jane.doe@example.com",
    "name": "Jane"
  }
}
 ```


## Connection Establishment
This is a WebSocket endpoint, and hence it starts as an HTTP request that contains HTTP headers that indicate the client's desire to upgrade the connection to a WebSocket instead of using HTTP semantics. The server indicates its willingness to participate in the WebSocket connection by returning an HTTP 101 Switching Protocols response. After the exchange of this handshake, both client and service keep the socket open and begin using a message-based protocol to send and receive information. Please refer to [WebSocket Specification RFC 6455](https://tools.ietf.org/html/rfc6455) for a more in-depth understanding of the Handshake process.

### Start Request


```js
{
  "type": "start_request",
  "insightTypes": ["question", "action_item"],
  "config": {
    "confidenceThreshold": 0.9,
    "languageCode": "en-US",
    "speechRecognition": {
      "encoding": "LINEAR16",
      "sampleRateHertz": 16000
    }
  },
  "speaker": {
    "userId": "jane.doe@example.com",
    "name": "Jane"
  }
}
```

This is a request to start the processing after the connection is established. Right after this message has been sent, the audio should be streamed, any binary audio streamed before the receipt of this message will be ignored.


To get direct access to the mic, we're going to use an API in the WebRTC specification called `getUserMedia()`.

Once the code is running, start speaking and you should see the message_response and insight_response messages getting printed on the console.


<Tabs
  defaultValue="javascript"
  values={[
    { label: 'Javascript', value: 'javascript', }
  ]
}>

<TabItem value="javascript">

```js
const handleSuccess = function(stream) {
  const context = new AudioContext();
  const source = context.createMediaStreamSource(stream);
  const processor = context.createScriptProcessor(1024, 1, 1);
  source.connect(processor);
  processor.connect(context.destination);
  processor.onaudioprocess = function(e) {
    // convert to 16-bit payload
    const inputData = e.inputBuffer.getChannelData(0) || new Float32Array(this.options.bufferSize);
    const targetBuffer = new Int16Array(inputData.length);
    for (let index = inputData.length; index > 0; index--)
        targetBuffer[index] = 32767 * Math.min(1, inputData[index]);
    // Send to websocket
    if(ws.readyState === WebSocket.OPEN){
        ws.send(targetBuffer.buffer);
    }
  };
};

navigator.mediaDevices.getUserMedia({ audio: true, video: false })
  .then(handleSuccess);

// Schedule the stop of the client after 2 minutes (120 sec)
setTimeout(() => {
  // Send stop request
  ws.send(JSON.stringify({
    "type": "stop_request"
  }));
  ws.close();
}, 120000);
```

</TabItem>
<TabItem value="curl">
</TabItem>
</Tabs>

### Stop Message
```js
{
  "type": "stop_request"
}
```

## Messages

### Message Formats
Client and Server both can send messages after the connection is established. According to RFC 6455, WebSocket messages can have either text or a binary encoding. The two encodings use different on-the-wire formats. Each format is optimized for efficient encoding, transmission, and decoding of the message payload.

#### Text Message
Text messages over WebSocket must use UTF-8 encoding. Text Message is the serialized JSON message. Every text message has a type field to specify the type or the purpose of the message.

#### Binary Message
Binary WebSocket messages carry a binary payload. For the Real-time API, audio is transmitted to the service by using binary messages. All other messages are the Text messages.

#### Client Messages
This section describes the messages that originate from the client and are sent to service. The types of messages sent by the client are `start_request`, `stop_request` and binary messages containing audio.

This is a request to stop the processing. After the receipt of this message, the service will stop any processing and close the WebSocket connection.

### Sending Binary Messages with Audio
The client needs to send the audio to Service by converting the audio stream into a series of audio chunks. Each chunk of audio carries a segment of audio that needs to be processed. The maximum size of a single audio chunk is 8,192 bytes.

### Service Messages
This section describes the messages that originate in Service and are sent to the client.

Service sends mainly two types of messages (`message_response`, `insight_response`) to the client as soon as they're available.

### Message Response
The `message_response` contains the processed messages as soon as they're ready and available, in the processing of the continuous audio stream. This message does not contain any insights.



Example of the `message_response` object


```js
{
  "type": "message_response",
  "messages": [
    {
      "from": {
        "name": "Jane",
        "userId": "jane.doe@example.com"
      },
      "payload": {
        "content": "I was very impressed by your profile, and I am excited to know more about you.",
        "contentType": "text/plain"
      }
    },
    {
      "from": {
        "name": "Jane",
        "userId": "jane.doe@example.com"
      },
      "payload": {
        "content": "So tell me, what is the most important quality that you acquired over all of your professional career?",
        "contentType": "text/plain"
      }
    }
  ]
}
```

### Insight Response
The `insight_response` contains the insights from the ongoing conversation as soon as they are available. This message does not contain any messages.



Example of the `insight_response` object

```js
{
  "type": "insight_response",
  "insights": [
    {
      "type": "question",
      "text": "So tell me, what is the most important quality that you acquired over all of your professional career?",
      "confidence": 0.9997962117195129,
      "hints": [],
      "tags": []
    },
    {
      "type": "action_item",
      "text": "Jane will look into the requirements on the hiring for coming financial year.",
      "confidence": 0.9972074778643447,
      "hints": [],
      "tags": [
        {
          "type": "person",
          "text": "Jane",
          "beginOffset": 0,
          "value": {
            "value": {
              "name": "Jane",
              "alias": "Jane",
              "userId": "jane.doe@symbl.ai"
            }
          }
        }
      ]
    }
  ]
}
```

## Using Trackers

:::note In Beta Phase
This feature is in the Beta phase. If you have any questions, ideas or suggestions please reach out to us at devrelations@symbl.ai.
:::

Symbl provides a real-time Streaming API for processing audio content in real time. You can also capture Trackers for a conversation in real time. For more details on Trackers, refer to the documentation [here](/docs/concepts/trackers). 

Below is an example that shows how to pass Trackers in the config object for the `startRealtimeRequest` of the Symbl’s JS SDK. This example also shows how to consume the results of the detected Trackers in real-time.

Below is the code snippet:

```js
const connection = await sdk.startRealtimeRequest({
    id,
    insightTypes: ['action_item', 'question'],
    trackers: [
        {
            name: "COVID-19",
            vocabulary: [
                "social distancing",
                "cover your face with mask",
                "vaccination"
            ]
        }
    ],
    config: {
        meetingTitle: "My Awesome Meeting",
        confidenceThreshold: 0.7,
        languageCode: "en-US",
        sampleRateHertz: 48000,
        trackers: {
            "interimResults": true
        }
    },
    speaker: {
        // Optional, if not specified, will simply not send an email in the end.
        userId: "john@example.com", // Update with valid email
        name: "John",
    },
    handlers: {
        onTrackerResponse: (data) => {
            // When a tracker is detected in real-time
            console.log('onTrackerResponse', JSON.stringify(data, null, 2));
            if (!!data) {
                data.forEach((tracker) => {
                    console.log(`Detected Tracker Name: ${tracker.name}`);
                    console.log(`Detected Matches`);
                    tracker.matches.forEach((match) => {
                        console.log(`Tracker Value: ${match.value}`);
                        console.log(`Messages detected against this Tracker`);
                        match.messageRefs.forEach((messageRef) => {
                            console.log(`Message ID: ${messageRef.id}`);
                            console.log(`Message text for which the match was detected: ${messageRef.text}`);
                            console.log(`\n`);
                        });
                        console.log(`\n\n`);
                        
                        console.log(`Insights detected against this Tracker`);
                        match.messageRefs.forEach((insightRef) => {
                            console.log(`Insight ID: ${insightRef.id}`);
                            console.log(`Insight text for which the match was detected: ${insightRef.text}`);
                            console.log(`Insight Type: ${insightRef.type}`);
                            console.log(`\n`);
                        });
                        console.log(`\n\n`);
                    });
                });
            }
        },
    },
})
```

Let’s go over all the parameters passed in the configuration object in the above function:

1. `id`: A unique UUID that represents this WebSocket API's session.

2. `insightType`: The types of **Insights** to be detected for this conversation.

3. `trackers`: The **Trackers** to be detected in real-time for that conversation.<!--  Follows the same structure as shown in the Trackers section. -->

4. `config`: The config object encapsulates the metadata for the WebSocket API’s session.

  a. `meetingTitle`: The title for this conversation or meeting.

  b. `confidenceThreshold`:  The **Insights** having confidence scores greater than this threshold will be the ones detected for the conversation.

  c. `languageCode`: The language-code in BCP-47 format. For a list of supported languages by Symbl, please take a look [here](https://docs.symbl.ai/docs/streaming-api/api-reference/#supported-languages).

  d. `sampleRateHertz`: The sample rate of the incoming audio data which is being pushed to Symbl.

  e. `trackers`: `{ interimResults }`: The `interimResults` flag tells Symbl to send the `tracker` results as soon as they are detected. If false, the `tracker` results are detected for the finalized transcription responses.

5. `speaker`: The details of the speaker in this conversation.

  a. `userId`: Unique identifier to represent the user.

  b. `name`: The name of the user.

6. `handlers`: The object encapsulating the call-back functions to be invoked on detection of those specific entities. For more information on various other handlers, check out the [Javascript SDK Reference](/docs/javascript-sdk/reference#event-handlers-1).

  a. `onTrackerResponse`: This function is invoked when Symbl detects a Tracker in real-time. The structure of the **Tracker** object is shown in the above code snippet.

## Supported Languages

Language is specified by language code in the `languageCode` key of the configuration object.

:::info
Currently, transcription punctuation is not supported for French (`fr-CA`) or Dutch (`nl-NL`) languages.
:::

The following list of languages(with their [BCP-47](https://en.wikipedia.org/wiki/IETF_language_tag) language-codes) are currently supported:

 | Supported Languages          | Code    |
 |------------------------------|---------|
 | English (United States)      | `en-US` |
 | English (United Kingdom)     | `en-GB` |
 | English (Australia)          | `en-AU` |
 | English (Ireland)            | `en-IE` |
 | English (India)              | `en-IN` |
 | English (South Africa)       | `en-ZA` |
 | Russian (Russian Federation) | `ru-RU` |
 | French (Canada)              | `fr-CA` |
 | French (France)              | `fr-FR` |
 | German (Germany)             | `de-DE` |
 | Italian (Italy)              | `it-IT` |
 | Dutch (Netherlands)          | `nl-NL` |
 | Japanese (Japan)             | `ja-JP` |
 | Spanish (United States)      | `es-US` |
 | Spanish (Spain)              | `es-ES` |
 | Arabic (Saudi Arabia)        | `ar-SA` |
 | Hindi (India)                | `hi-IN` |
 | Portuguese (Brazil)          | `pt-BR` |
 | Portuguese (Portugal)        | `pt-PT` |
 | Persian (Iran)               | `fa-IR` |   


:::caution
 1. If the language is not specified then `en-US`(English - United States) is used as the default language.
 2. Insights like Action items, follow-ups, topics, etc are detected for the English language only.
:::


### Code Example

Below is an example of a request payload specifying that the meeting’s language should be in Spanish.

```js
{
  type: 'start_request',
  meetingTitle: 'Websockets How-to', // Conversation name
  insightTypes: ['question', 'action_item'], // Will enable insight generation
  config: {
    confidenceThreshold: 0.5,
    languageCode: 'es-ES',
    speechRecognition: {
      encoding: 'LINEAR16',
      sampleRateHertz: 44100,
    }
  },
  speaker: {
    userId: 'example@symbl.ai',
    name: 'Example Sample',
  }
}
```

:::info
If a language is not specified, it will currently fall back to English (en-US).
:::
