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

# Getting Started

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
    "^WATCH_VIDEO_(?P<video_id>[a-z0-9-]+)$": {
      "context": {
        "video_url": "videos/{{video_id}}.mp4"
      },
      "next_step": "watch_video"
    },
  "^(some|bad|words|to|ignore)$": null
  }
}
```

Commands that are available to the enduser at any point during the bot execution, regardless of its current state. Triggers are matched with regular expressions, if the regexp contains named groups, those will be added to the bot context with the corresponding matched value. Usually triggers are used to capture button clicks (what Facebook calls 'Postbacks') and general commands like "help". When a user input is captured by a trigger, it is not consumed by the current state.

Triggers must have a valid `next_step` attribute. If next step is `null` the bot will consume that input without any change, which is useful if we want to ignore some words. See [next_step](#next_step) reference for more info.

##definition

The array of possible states of the bot. The first state should be named **initial** and should **NOT** have output. When the bot enters a state, the first thing it does is to update the context, then writting all the outputs (if any) and finally wait for a user input (if defined). Either `output` or `input` must be defined.

###context 

*(Optional)*

TODO

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

Next state of the bot.

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
  "title": "My Home",                   //Optional
  "address": "Hollywood Boulevard 32",  //Optional
  "url": "http://any.url.com"           //Optional
}
```

TODO

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

They support 2 types of buttons:

`postback`: this type of buttons act as if the end-user typed the title of the button.

`web_url`: this type of buttons open the `url` in a browser.

<aside class="warning">
IMPORTANT: The carrousel type is only available on Facebook. Currently other platforms will ignore that output.
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

TODO

##Get age

```json
{

}
```

TODO

#Templating

See [Jinja](http://jinja.pocoo.org/).