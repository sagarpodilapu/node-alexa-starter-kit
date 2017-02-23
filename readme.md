# Simple Amazon Alexa Starter Kit

This starter kit allows you to add multiple new skills for Amazon Alexa easily. <br>
Configuration for <a href="https://developer.amazon.com/alexa">Amazon</a>
is generated automatically and saved to **amazonConfig.txt** file,
so all you have to do is to copy it to <a href="https://developer.amazon.com/alexa">Amazon Alexa website</a><br>
**This kit uses <a href="https://www.npmjs.com/package/alexa-app">alexa-app</a> module, visit the page
for more information about functionality and customizing server if needed.**

## Initialization
- ```git clone git@github.com:AlexSapoznikov/alexa-express-starter-kit.git```
- ```cd alexa-express-starter-kit/```
- ```npm install```

## Commands
- ```npm start``` - builds and starts server
- ```npm run start-dev``` - starts and restarts server on code change
- ```npm run build``` - builds without starting server
- ```npm run create-skill [-- --name=anyname]``` - creates new sample skill file in *skills* folder
- ```npm run expose``` - exposes your localhost to the internet
- ```npm run eslint``` - lints the code for errors
- ```npm run test``` - runs tests

## The Structure

- Development is being done in *./src* folder.
- The builded code is generated to *./public* folder.
- Configuration files are located in *./config*.
- Skills are located in *./src/skills/* folder.
- Skills are included in *./src/skills.js* file.
- Server creates intents (endpoints) automatically using merged skills in *./src/skills.js* file.
- Scripts for generating amazon configuration and new skill files are located in *./src/scripts* folder.
- Error messages are located in *./src/errorMessages.js* file

## How to add new skills

- Add new skill file via terminal
```
npm run create-skill -- --name=myNewSkill
```
- Edit the <a href="#skillfile">skill file</a> (*./src/skills/myNewSkill.js*).
- Start the server
```
    npm run start
    // or
    npm run start-dev
```

- For development, expose your localhost to the internet.
    - Start ngrok
    ```
        npm run expose
    ```
    - Look for the output to find <a name="alexaendpoint">alexa endpoint</a>, it looks something like this: https://1234ccb1.ngrok.io/alexa

- Do a setup in <a href="https://developer.amazon.com/alexa">Amazon Alexa website</a>
    - Sign in to <a href="https://developer.amazon.com/alexa">Amazon Alexa website</a>
    - Add new skill
    - Copy-paste generated configuration from <a href="#amazonconf">**./amazonConfig.txt**</a> to required fields.
    - Use <a href="#alexaendpoint">https url generated by ngrok</a> in previous step
    
Alexa is now ready for testing.

- To invoke custom skill, say: *Alexa, ask [invocationName] [question using utterances]*
- For more information about invoking custom skills, visit <a href="https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/supported-phrases-to-begin-a-conversation">Amazon Developer commands page</a>

## <a name="skillfile">Skill file</a>

Skill file looks like this:

```
export default {
  skillName: 'dog',
  invocationName: 'dog',
  intents: [
    {
      intentName: 'dogNumber',
      slots: {
        number: 'AMAZON.NUMBER'
      },
      utterances: [
        'say the number {-|number}',
        'find {-|number}'
      ],
      response: (request, response) => {
        const number = request.slot('number');
        response.say(`The number you asked to say is ${number}`);
      }
    }
  ]
};
```

- skillName - name of skill that is displayed to customers in the Alexa app. Must be between 2-50 characters.
- invocationName - name that customers use to activate a skill.
- intents - array on intents
- intentName - name of intent
- slots - variables
- utterances - these are what people say to interact with skill
- response - response from alexa

### Handler

Request and response objects that can be used.<br>
Documentation about handler objects is copied from <a href="https://www.npmjs.com/package/alexa-app">alexa-app</a> npm module,
because current starter kit uses it's functionality.

#### Request

```
// return the type of request received (LaunchRequest, IntentRequest, SessionEndedRequest)
String request.type()

// return the value passed in for a given slot name
String request.slot("slotName")

// check if you can use session (read or write)
Boolean request.hasSession()

// return the session object
Session request.getSession()

// return the request context
request.context

// the raw request JSON object
request.data
```

#### Response

```
// tell Alexa to say something; multiple calls to say() will be appended to each other
// all text output is treated as SSML
response.say(String phrase)

// empty the response text
response.clear()

// tell Alexa to re-prompt the user for a response, if it didn't hear anything valid
response.reprompt(String phrase)

// return a card to the user's Alexa app
// for Object definition @see https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/alexa-skills-kit-interface-reference#card-object
// skill supports card(String title, String content) for backwards compat of type "Simple"
response.card(Object card)

// return a card instructing the user how to link their account to the skill
// this internally sets the card response
response.linkAccount()

// play audio stream (send AudioPlayer.Play directive) @see https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/custom-audioplayer-interface-reference#play-directive
// skill supports stream(String url, String token, String expectedPreviousToken, Integer offsetInMilliseconds)
response.audioPlayerPlayStream(String playBehavior, Object stream)

// stop playing audio stream (send AudioPlayer.Stop directive)
response.audioPlayerStop()

// clear audio player queue (send AudioPlayer.ClearQueue directive)
// clearBehavior is "CLEAR_ALL" by default
response.audioPlayerClearQueue([ String clearBehavior ])

// tell Alexa whether the user's session is over; sessions end by default
// you can optionally pass a reprompt message
response.shouldEndSession(boolean end [, String reprompt] )

// send the response to the Alexa device (success)
// this is not required for synchronous handlers
// you must call this from asynchronous handlers
response.send()

// trigger a response failure
// the internal promise containing the response will be rejected, and should be handled by the calling environment
// instead of the Alexa response being returned, the failure message will be passed
response.fail(String message)

// calls to response can be chained together
response.say("OK").send()
```


#### Session

```
// check if you can use session (read or write)
Boolean request.hasSession()

// get the session object
var session = request.getSession()

// set a session variable
// by defailt, Alexa only persists session variables to the next request
// the alexa-app module makes session variables persist across multiple requests
// Note that you *must* use `.set` or `.clear` to update
// session properties. Updating properties of `attributeValue`
// that are objects will not persist until `.set` is called
session.set(String attributeName, String attributeValue)

// return the value of a session variable
String session.get(String attributeName)

// session details, as passed by Amazon in the request
session.details = { ... }
```

## Server

Look ./src/server.js file in the code. <br>
Following handlers are added for editing:
- app.launch
- app.pre
- app.post
- app.sessionEnded
- app.error

For more functionality and options visit <a href="https://www.npmjs.com/package/alexa-app">alexa-app</a> npm module.

## <a name="amazonconf">Configuration for amazon</a>

When starting server, Amazon configuration is written to **amazonConfig.txt** file.<br>
All you need for configuring Alexa in Amazon is to copy name, invocation name, intent schema and sample utterances to corresponding fields.<br>
The file looks like this:

```

Amazon configs:

--------------------------------
SKILL 1: dog
--------------------------------
Name: dog
Invocation Name: dog
Intent Schema:
{
  "intents": [
    {
      "intent": "dogDate",
      "slots": [
        {
          "name": "date",
          "type": "AMAZON.DATE"
        }
      ]
    },
    {
      "intent": "dogNumber",
      "slots": [
        {
          "name": "number",
          "type": "AMAZON.NUMBER"
        }
      ]
    },
    {
      "intent": "dogFood",
      "slots": null
    }
  ]
}
Sample Utterances:
dogDate	{date}
dogNumber	say the number {number}
dogNumber	find {number}
dogFood	give him food
dogFood	give her food
dogFood	she wants food
dogFood	he wants food
dogFood	she needs food
dogFood	he needs food


--------------------------------
SKILL 2: anyone
--------------------------------
Name: anyone
Invocation Name: anyone
Intent Schema:
{
  "intents": [
    {
      "intent": "askanyone",
      "slots": null
    }
  ]
}
Sample Utterances:
askanyone	if
```

## Licence

Copyright (c) 2017 Aleksandr Sapožnikov, NodeSWAT

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
