# Fredknows.it expert API

[Fredknows.it](https://www.fredknows.it) lets you build your own AI expert.

This repository contains the documentation for the expert API.

A reference jquery implementation is also available on our public [jquery expert repository](https://github.com/fredknowsit/jquery-expert).

## Quickstart

The base API URL for an expert is:
`https://admin.fredknows.it/session/api/expert-id/<your-expert-id>/`,
where `<your-expert-id>` is the identifier of your expert.

For the purpose of this tutorial, we will use the sample [zooki animal expert](https://expert.fredknows.it/5887544647657bc7145ea94c). Its expert id is: `5887544647657bc7145ea94c`.

Let's start a new session by providing a unique session identifier (`session_id`) using [HTTPie](https://httpie.org):

```shell
http POST https://admin.fredknows.it/session/api/expert-id/5887544647657bc7145ea94c/query/ session_id="123456789"
```

The expert response will look like this:

```
{
    "session_id": "123456789",
    "result": {
        "id": "5887584904460c2223a449d3",
        "type": "feature",
        "title": "How many arms & legs does your animal have?",
        "description": "Please note: wings, tentacles or fins do not count. Only the sum of arms and legs.",
        "media_url": "https://s3-eu-west-1.amazonaws.com/fred-media/5887544647657bc7145ea94c/featuregroup-5887584904460c2223a449d3-1488538503.png",
        "options": [
            {
                "label": "0",
                "payload": "feature-57c82a4d04460c3ca6350ef7-y"
            },
            {
                "label": "2",
                "payload": "feature-57c82a4d04460c3ca6350ef8-y"
            },
            {
                "label": "4",
                "payload": "feature-57c82a4d04460c3ca6350ef9-y"
            },
            {
                "label": "6",
                "payload": "feature-57c82a4d04460c3ca6350efa-y"
            },
            {
                "label": "8",
                "payload": "feature-57c82a4d04460c3ca6350efb-y"
            },
            {
                "label": "more than 8",
                "payload": "feature-57c82a4d04460c3ca6350efc-y"
            }
        ],
        "custom_texts": "...",
        "feedback_forms": null
    },
    "progress": 0.0
}
```

The response is a JSON dictionary. The most relevant keys are:

1. `session_id`: The session identifier provided when starting the session. You
   will reuse this id in subsequent requests.
2. `result`: The result of your request.
  1. `type`: typically a **feature** (a question) or a **class** (a solution).
  2. `options`: a list of **payloads** to reply with.

Let's tell our _animal_ expert that the animal we are thinking has _0 legs_. For that, we'll send the payload `feature-57c82a4d04460c3ca6350ef7-y`:

```shell
http POST https://admin.fredknows.it/session/api/expert-id/5887544647657bc7145ea94c/query/ \
  session_id="123456789" \
  postback:='{"payload": "feature-57c82a4d04460c3ca6350ef7-y"}'
```

Which returns:

```json
{
    "session_id": "123456789",
    "result": {
        "id": "57c82a4d04460c3ca6350efe",
        "type": "feature"
        "title": "Is the animal a predator?",
        "description": null,
        "media_url": null,
        "options": [
            {
                "label": "button_yes",
                "payload": "feature-57c82a4d04460c3ca6350efe-y"
            },
            {
                "label": "button_no",
                "payload": "feature-57c82a4d04460c3ca6350efe-n"
            },
            {
                "label": "button_idk",
                "payload": "feature-57c82a4d04460c3ca6350efe-idk"
            }
        ],
        "custom_texts": "...",
        "feedback_forms": null
    },
    "progress": 0.04
}
```

As you can see, the expert replied with another **feature** (_question_), and three possible **payloads**.

Our expert will keep on asking **features** until it:

- Finds a **class** (a _solution_)
- Runs out of questions.

A **class** response will look like this:

```json
{
    "session_id": "123456789",
    "result": {
        "id": "5811bca73a641c69dd85ac0d", 
        "type": "class"
        "title": "I think you're looking for a piranha, right?", 
        "media_url": "https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/Pirhana06.jpg/250px-Pirhana06.jpg", 
        "options": [
            {
                "label": "button_correct_solution", 
                "payload": "class-5811bca73a641c69dd85ac0d-y"
            }, 
            {
                "label": "button_wrong_solution", 
                "payload": "class-5811bca73a641c69dd85ac0d-n"
            }
        ], 
        "custom_texts": "...",
        "feedback_forms": "..."
    }, 
    "progress": 0.95
}
```

We could confirm the class proposed by our expert sending the payload `class-5811bca73a641c69dd85ac0d-y`:

```shell
http POST https://admin.fredknows.it/session/api/expert-id/5887544647657bc7145ea94c/query/ \
  session_id="123456789" \
  postback:='{"payload": "class-5811bca73a641c69dd85ac0d-y"}'
```

Which will return the following response:

```json
{
    "session_id": "lep1zlh78fd2kked66ep45"
    "result": {
        "id": "5811bca73a641c69dd85ac0d", 
        "type": "end of game"
        "title": "Class feedback was correctly submitted", 
        "description": "", 
        "media_url": "", 
        "options": [], 
        "custom_texts": "...",
        "feedback_forms": null
    }, 
    "progress": 1.0
}
```

As you can see, the result _type_ of this response is `end of game`. We will find out more about possible response types in the next section.





```
```

## Workflow diagram
![expert_flow](https://s3.eu-central-1.amazonaws.com/expert-flow-img/exper_flow.png "Expert flow")

## API Reference

### Base URL
The base URL is `https://admin.fredknows.it/session/api/expert-id/{your-expert-id}/`.  You need to replace `your-expert-id` with the ID of your expert.

### Authentication
Currently the endpoint is open to everyone as long as your expert page is activated. 

## `GET /`

Sending a **GET** request to the **base URL** retrives the **custom texts** configured in the Admin panel:
#### Response schema
```sh
curl -X GET "https://admin.fredknows.it/session/api/expert-id/your-expert-id/"
```
```json
{
  "custom_texts": {
    "button_correct_solution": "Yes, that was helpful",
    "button_idk": "I don't know",
    "button_no": "No",
    "button_wrong_solution": "No, it didn't help",
    "button_yes": "Yes",
    "class_found": "To me it sounds like the following issue:",
    "class_show": "This should help you solving the issue:",
    "error": "Houston, we have problem! Seems like something went wrong. Please try again in a couple of minutes or contact my human colleauges.",
    "feedback": "I’d like to learn and improve. Would you be so kind and give me some feedback?",
    "feedback_thanks": "Thank you for taking the time and give me feedback. My human colleagues are working hard to improve my solution skills.",
    "introduction": "Hey Ho Let's Go!. She was literally amazed!!!",
    "out_of_questions": "I don't have anymore smart question to ask. Thus I stop here. I sincerely apologize for the inconvenience.",
    "restart": "We've reached the end. If you like, you can restart. Otherwise: Goodbye & have a great time!",
    "solved_no": "I sincerely apologize for the inconvenience.",
    "solved_yes": "Great! In case you were happy with my service, feel free to spread the word."
  },
  "title": "your_project_title",
  "media_url": "https://s3-eu-west-1.amazonaws.com/fred-media/57bc52f351dcc257e954080e/class-5878d83204460c71ba3351bc-1484837262.png"
}
```
You should be familiar with these texts from the interaction with the expert in the Admin panel. If you wish to integrate these, retrieve them once at start-up.
## `POST /query`
The **query** endpoint is used to interact with your expert. It returns a **feature** (question) or a **class** (solution) and a list of options how to proceed. 

On the **first call** you only need to provide a unique session ID. As response you will retrieve the first **feature** with a list of options. On every further call you submit the `payload` of the selected option as `postback`.

### Request schemas
**Session start**

```json
POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id/"

Headers:
"Content-Type: application/json; charset=utf-8"

POST body:
{
	"session_id": "1234567890"
}
```
**Question/Solution reply**

```json
POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id/"

Headers:
"Content-Type: application/json; charset=utf-8"

POST body:
{
	"session_id": "1234567890",
	"postback": {
		"payload": "feature-58763fb804460c787949f25e-y"
	}
}
```
Note: subsequent calls with the same reply will return an error.

**Go back one step**

```json
POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id/"

Headers:
"Content-Type: application/json; charset=utf-8"

POST body:
{
	"session_id": "1234567890",
	"postback": {
		"payload": "goback"
	}
}
```
Notes: 

+ will return an error if requested before session start
+ will return an error if requested before the 1st question is answered
+ will return an error if requested after user feedback was sumbitted

**Feedback**

```json
POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id/"

Headers:
"Content-Type: application/json; charset=utf-8"

POST body:
{
	"session_id": "1234567890",
	"message": {
		"text": "The text typed in by the user."
	}
}
```
**Alternative solutions**

```json
POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id/"

Headers:
"Content-Type: application/json; charset=utf-8"

POST body:
{
	"session_id": "1234567890",
	"postback": {
		"payload": "alternatives"
	}
}
```
Note: will return an error if requested before a solution has been found.

### Parameters

| Name | Type | Description |
|------|------|-------------|
| session_id | String | A unique session ID that you have to generate |
| postback | Object | `postback` object, *not on first call* |
| message | Object | `message` object, *only at end of session* | 
| dry_run | Boolean | Set to `true` if session should *NOT* be stored for the expert's training | 

### Postback Object

| Name | Type | Description |
|------|------|-------------|
| payload | String | Payload of the option that the user choose |


### Message Object

| Name | Type | Description |
|------|------|-------------|
| text | String | Text input from user that is saved as user feebdack |

### Response schemas

**Question/Solution**

```json
{
  "result": {
    "type": "feature"   
    "title": "Can you log in to your computer?", 
    "description": null, 
    "id": "123456", 
    "media_url": null, 
    "options": [
      {
        "label": "Yes", 
        "payload": "feature-58763fb804460c787949f25e-y"
      }, 
      {
        "label": "No", 
        "payload": "feature-58763fb804460c787949f25e-n"
      }, 
      {
        "label": "I don't know", 
        "payload": "feature-58763fb804460c787949f25e-idk"
      }
    ], 

  }, 
  "session_id": "1234567890",
  "progress": 0.26
}
```
**Alternatives**
 
```json
{
  "classes":[
  {
  		"type": "class"   
	   	"title": "Solution 1", 
	   	"description": null, 
	   	"id": "123456", 
	  	"media_url": null, 
   },
   {
   		"type": "class"   
	    "title": "Solution 2", 
	    "description": null, 
	    "id": "123457", 
	    "media_url": null, 
	},
	{
		"type": "class"   
		"title": "Solution 2", 
		"description": null, 
		"id": "123458", 
		"media_url": null, 
	},
  ] 
  "session_id": "1234567890"
}
```


### Response fields

| Name | Type | Description |
|------|------|-------------|
| type | String | Entity type `feature` or `class` or one of the status types `no-question-no-solution` if there's nothing to ask anymore or `closed` if the session is over |
| title | String | Title of entity |
| description | String | Description of entity |
| id | String | ID of entity |
| media_url | String | Image URL of entity |
| options | Array | Array with `option` objects |
| progress | Float | Progress to successfully completing a session |

### Option Object
| Name | Type | Description |
|------|------|-------------|
| label | String | Label text of payload, e.g. `Yes` |
| payload | String | Value to submit as `postback` if an option was selected |

