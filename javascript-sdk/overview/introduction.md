---
id: introduction
title: Symbl SDK (Node.js)
sidebar_label: Introduction
slug: /javascript-sdk/introduction
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


The Programmable Javascript SDK allows you to add Conversational Intelligence directly into your web applications and meeting platforms. With the Javascript SDK, you can generate intelligent insights such as action items, topics and questions. Currently, the Symbl Javascript SDK only works with the [Telephony API](/docs/telephony/introduction) and [Streaming API](/docs/streamingapi/overview/introduction)


### Installation

First, start by installing our Javascript SDK:

```bash
$ npm install --save symbl-node
```


### Initialize



```js
sdk.init({
  // APP_ID and APP_SECRET come from the Symbl Platform: https://platform.symbl.ai
  appId: APP_ID,
  appSecret: APP_SECRET,
  basePath: 'https://api.symbl.ai'
})
.then(() => console.log('SDK Initialized.'))
.catch(err => console.error('Error in initialization.', err));
 ```


 Import the SDK using the ES5 or ES6 way:

<Tabs
  defaultValue="es6"
  values={[
    { label: 'ES6', value: 'es6', },
    { label: 'ES5', value: 'es5', },
  ]
}>

<TabItem value="es5">

```js
var sdk = require('symbl-node').sdk;
```

 </TabItem>

<TabItem value="es6">

```js
import { sdk } from 'symbl-node';
```

</TabItem>
</Tabs>

### Tutorials

We have prepared a list of tutorials to help you understand how to use Symbl's Javascript SDK.

#### Telephony API Tutorials

* [Real-time Output with PSTN Dialing using Symbl's Javascript SDK](/docs/javascript-sdk/tutorials/get-real-time-transcription-js-sdk)
* [How to pass different audio codecs to Symbl endpoint](/docs/javascript-sdk/tutorials/pass-audio-codecs)
* [Using Symbl SDK (Node.js) To Push Speaker Events](/docs/javascript-sdk/tutorials/push-speakerevents-get-summary-url)

#### Streaming API Tutorials

* [How to receive live speech-to-text and AI Insights on your Mac/Windows PC?](/docs/javascript-sdk/tutorials/receive-ai-insights-from-your-computer)


### Code Snippets

#### Telephony API Code Snippets

* [Connect to Endpoints](/docs/javascript-sdk/code-snippets/connect-to-endpoints)
* [Active Speaker Events](/docs/javascript-sdk/code-snippets/active-speaker-events)
* [Set Language and Timezone When Connecting To An Endpoint](/docs/javascript-sdk/code-snippets/use-languages-timezones-with-sdk)

#### Streaming API Code Snippets

* [Subscribe to real-time Events](/docs/javascript-sdk/code-snippets/subscribe-real-time)
* [Streaming Audio in real-time](/docs/javascript-sdk/code-snippets/streaming-audio-real-time)
* [Set Language When Connecting To A Web Socket](/docs/javascript-sdk/code-snippets/use-languages-with-sdk	)



### Javascript SDK Reference

Supported methods and events for the Symbl Javascript SDK are listed below:

* [Public Endpoints](/docs/javascript-sdk/reference#public-methods)
    * [init](/docs/javascript-sdk/reference#init)
    * [startEndpoint](/docs/javascript-sdk/reference#startendpoint)
    * [stopEndpoint](/docs/javascript-sdk/reference#stopendpoint)
    * [startRealtimeRequest](/docs/javascript-sdk/reference#startRealtimeRequest)
    * [subscribeToConnection](/docs/javascript-sdk/reference#subscribetoconnection)
    * [pushEventOnConnection](/docs/javascript-sdk/reference#pusheventonconnection)
* [Event Handlers](/docs/javascript-sdk/reference#event-handlers-1)
    * [onSpeechDetected](/docs/javascript-sdk/reference#onspeechdetected)
    * [onMessageResponse](/docs/javascript-sdk/reference#onmessageresponse)
    * [onInsightResponse](/docs/javascript-sdk/reference#oninsightresponse)
    * [onTopicResponse](/docs/javascript-sdk/reference#ontopicresponse)

