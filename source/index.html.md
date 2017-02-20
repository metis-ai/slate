---
title: botSON Reference

<!--language_tabs:
  - json-->

toc_footers:
  - <a href='https://hubtype.com/'>Hubtype</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

#Introduction

botSON definitions for the bots created and mantained for hubtype.

This folder contains several examples of chatbots defined in our JSON specification language (botSON). Here you'll also find the bots we define for our real clients, for demos or for fun (consumer oriented).

botSON is a JSON structure defined to create chatbots. Bots are finite states machines that are able to comunicate with external resources and do some stuff defined by their transitions. It uses [Jinja](http://jinja.pocoo.org/) to give flexible state interactions.

#Getting Started

##Creating a state

```json
{
  "definition": [
    {
      "label": "helloworld",
      "output": "Hello World!",
      "next_step": "exit"
    }
  ]
}
```

The definition of the bot is composed by states. Each one has a `label`, a `next_step` and at least one `output` or `input`. The state 'exit' indicates the end of execution. There are four additional obligatory states needed for the correct execution: 'initial', 'input_failure', 'external_request_failure' and 'fallback_instruction'.

##First complete definition

```json
{
  "definition": [
    {
      "label": "initial",
      "input": {
        "variable":"first_text_from_user",
        "action":"free_text"
      },
      "next_step": "helloworld"
    },
    {
      "label": "helloworld",
      "output": "Hello World!",
      "next_step": "exit"
    },
    {
      "label": "input_failure",
      "output": {
        "type": "text",
        "data": "This happens when the end user don't put the expected input data"
      },
      "next_step":  "exit"
    },
    {
      "label": "external_request_failure",
      "output": {
        "type": "text",
        "data": "This happens when a response is outside the range 200-299"
      },
      "next_step":  "exit"
    },
    {
      "label": "fallback_instruction",
      "output": {
        "type": "text",
        "data": "This happens when a state is missing"
      },
      "next_step": "exit"
    }
  ]
}
```

Every bot starts it's execution in the 'initial' state when it receives the first message. Then, the automata follows the flow from state to state assigned in the 'next_step' field.

##Getting started 2

#Top level properties

##input_retry

```json
{
  "input_retry": 2
}
```

Specifies the number of input failures before the machine goes to the input_failure state. By default is set to 3.

##name

```json
{
  "name": "The Weather Bot"
}
```

The name of the bot.

##language *(not used)*

Display languaje of the bot, languaje of output messages.

##defaults

```json
{
  "defaults": {
    "requests": {
      "headers": {
        "Authorization": "Bearer mytoken"
      }
    },
    "context": {
      "API_URL": "https://api.example.com",
      "foo": "bar"
    }
  }
}
```

Defines the default values that are used throughout the chatbot life in different features:

* **requests**

  Currently only supports default `headers` to be used in every request.

* **context**

  Variables that will be always available at any state of the bot. They are recalculed in each state change.

##triggers

```json
{
  "triggers": {
    "payload":[
      {
        "match":"^WATCH_VIDEO_(?P<video_id>[a-z0-9-]+)$",
        "context": {
          "video_url": "videos/{{video_id}}.mp4"
        },
        "next_step": "watch_video"
      }
    ],
    "text":[
      {
        "match":"trigger_(?P<id>\\w+)",
        "next_step": "trigger_id"
      },
      {
        "match":"help",
        "next_step": "trigger_help"
      }
    ]
  }
}
```

Commands that are available at any point during the bot execution, regardless of its current state.`text` triggers can be fired by user inputs or payloads, while `payload` triggers can not be fired by user messages. Triggers are matched with regular expressions, if the regexp contains named groups, those will be added to the bot context with the corresponding matched value. Usually triggers are used to capture button clicks (what Facebook calls 'Postbacks') and general commands like "help". When a user input is captured by a trigger, it is not consumed by the current state.

Triggers must have valid `next_step` and `match` attributes. If next step is `null` the bot will consume that input without any change, which is useful if we want to ignore some words. See [next_step](#next_step) reference for more info.

##definition

The array of possible states of the bot. The first state should be named **initial** and should **NOT** have output. When the bot enters a state, the first thing it does is to update the context, then writting all the outputs (if any) and finally wait for a user input (if defined). Either `output` or `input` must be defined.

###label

The state name.

###context 

*(Optional)*

Variables to be saved when entering the state.

###output 

*(Optional)*

The bot will output the data defined here. It has different possible [data types](#output-types).

###input 

*(Optional)*

Input from the user. The field "actions" specifies the expected [type of input](#input-actions).

###next_step

```json
{
  "next_step": "another_state"
}
```

Next state to jump after all steps: **context , output, input**.

#Output types

##Text

```json
{
  "type": "text",
  "data": "Echo!"
}
```

TODO

##Image

```json
{
  "type": "image",
  "data": "http://this.is.the/url/of/the/image.jpg"
}
```

TODO

##Video

```json
{
  "type": "video",
  "data": "http://this.is.the/url/of/the/video.mp4"
}

```

TODO

##Audio

```json
{
  "type": "audio",
  "data": "http://this.is.the/url/of/the/audio.mp3"
}

```

TODO

##Document

```json
{
  "type": "document",
  "data": "http://this.is.the/url/of/the/document.pdf"
}
```

TODO

##Location

```json
{
  "type": "location",
  "latitude": 41.412255,
  "longitude": 2.2079313,
  "title": "My Home",
  "address": "Hollywood Boulevard 32",
  "url": "http://any.url.com"
}
```

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "location" | |
| latitude  | Number      |    |
| longitude | Number      |    |
| title     | String      |  *optional*<br>*32 chars max*  |
| address   | String      |  *optional*  |
| url       | String      |  *optional*  |

<aside class="warning">
IMPORTANT: The Location Output is not available on Facebook.
</aside>

##Contact

```json
{
  "type": "contact",
  "first_name": "John",
  "last_name": "Doe",                   //Optional
  "phone_number": "678909909",          //Optional
  "vcard": "vcard:data"                 //Optional
}
```

TODO

##Carrousel

```json
{
  "type": "carrousel",
  "elements":[
    {
      "title": "Title Element 1",
      "subtitle": "My description",
      "image_url": "https://www.cover.image.jpg",
      "buttons":[
        {
          "type": "postback",
          "title": "My Button",
          "payload": "EXTRA_DATA"
        }
      ]
    },
    {
      "title": "Title Element 2",
      "subtitle": "My Description",
      "image_url": "https://www.cover.image.jpg",
      "buttons":[
        {
          "type": "web_url",
          "title": "Darse de baja",
          "url": "https://www.some.web.com"
        }
      ]
    }
  ]
}
```

A rich interactive message that displays a horizontal scrollable carousel of items, each composed of an image attachment, short description and buttons to request input from the user.

<img width="300" src="/images/carrousel.png">

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "carrousel" | |
| elements  | Array of Elements  | *10 elements max (will truncate)*   |

###Element:

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| title      | String | |
| subtitle  | String  |   |
| image_url  | String  |   |
| buttons  | Array of Buttons  | *3 elements max (will truncate)*  |

###Button:

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "web_url" or "postback" | |
| title  | String  |   |
| url  | String  | *mandatory if type is "web_url"*  |
| payload  | String  | *mandatory if type is "postback"* |

`postback`: this type of buttons act as if the end-user typed the title of the button.

`web_url`: this type of buttons open the `url` in a browser.

<aside class="warning">
IMPORTANT: The carrousel output is only available on Facebook. Currently other platforms will ignore that output.
</aside>

##Quick Replies

```json
{
  "type": "text",
  "data": "What do you need?",
  "keyboard": [
    {
      "label": "Option One",
      "data": "option_1"
    },
    {
      "label": "Option Two",
      "data": "option_2"
    },
    {
      "label": "Option Three",
      "data": "option_3"
    }
  ]
}
```

Quick Replies are a convenient way to let users know the available options to type. Also, they save the end-user the effort to type all words and let them navigate through options much faster.

Any message type accepts a `"keyboard"` field with an array of (`"label"`, `"data"`) pairs.

#Input actions

The data will be stored in the specified variable.

##Free Text

```json
{
  "variable": "text_var_name",
  "action": "free_text"
}
```

##Get Integer

```json
{
  "variable": "name_of_int_var",
  "action": "get_int"
}
```

TODO

##Get in Set

```json
{
  "variable": "name_of_option",
  "action": "get_in_set",
  "action_parameters": ["op1", "op2"]
}
```

Only accepts these parameters as answer. It will ask at most `input_retry` times.

##Get in Set Fuzzy

```json
{
  "variable": "object_user_choice",
  "action": "get_in_set_fuzzy",
  "action_parameters": ["object1", "object2"]
}
```

Same as get in set, but it does not require a perfect match.

##Get yes or no

```json
{
  "variable": "confirm_var_name",
  "action": "get_yes_no"
}
```

TODO

##Get from url

```json
{
  "variable": "var_name_to_save",
  "action": "get_from_url",
  "action_parameters": {
    "url": "{{BASE_URL}}path/search/",
    "params": {
      "param1": "{{_input}}",
      "param2": "val_2"
    }
  }
}
```

TODO - it makes a call to the url and stores the response

##Get name

```json
{

}
```

TODO

##Get email

```json
{
  "variable": "user_email",
  "action": "get_email"
}
```

Gets the email and check if it is well writed. **something@someother**

##Get age

```json
{
  "action":"get_age",
  "variable":"age"
}
```

Saves a number representing the age. Age should match 1 <= age < 120 

##Get location

```json
{
  "action":"get_location",
  "variable":"location"
}
```

The variable location will contain the components: `latitude` , `longitude`, `title`, `address`, `url`. For field info see [location](#location).
IMPORTANT: The get location input is only available on Facebook. 

##Get image
```json
{
  "variable": "the_image_url",
  "action": "get_image"
}
```
It will save in the variable the url given by the provider ( facebook )
IMPORTANT: The get image input is only available on Facebook. 
#Templating

See [Jinja](http://jinja.pocoo.org/).