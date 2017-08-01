# Fredknows.it expert API

[Fredknows.it](https://www.fredknows.it) lets you build your own AI expert.

This repository contains the documentation for the expert API.

A reference jquery implementation is also available on our public [jquery expert repository](https://github.com/fredknowsit/jquery-expert).

## Quickstart

Start a new session by providing a unique session ID and receive a **feature** (question) or a **class** (solution) and a list of options to reply with:

*Note: You can try it out using the ID of the "Zooki" animal expert:* **5887544647657bc7145ea94c**

### Request

```json
POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id/query/"

Headers:
"Content-Type: application/json; charset=utf-8"

POST body:
{
	"session_id": "1234567890"
}
```
```bash
curl -X POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id/query/" -H "Content-Type: application/json" -d '{
    "session_id": "1234567890"
}'
```
### Response

```json
{
  "result": {
    "type": "feature"   
    "title": "Can you log in to your computer?", 
    "description": null, 
    "id": "58763fb804460c787949f25e", 
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

Reply with one of the payload options and eventually reach a **solution** (class). 

### Request

```json
POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id-here/query/"

Headers:
"Content-Type: application/json; charset=utf-8"

POST body:
{
	"session_id": "1234567890",
	"postback": {
		"payload": "feature-58763fb804460c787949f25e-n"
	}
}
```
```bash
curl -X POST "https://admin.fredknows.it/session/api/expert-id/5899ee0f04460c5bfb00c1ad/query/" -H "Content-Type: application/json" -d '{
    "session_id": "1234567890",
    "postback": {
		"payload": "feature-589c930004460c3ad64702d2-y"
	}
}'
```
### Response

```json
{
  "result": {
    "type": "class"   
    "title": "There could be a typo in your password.", 
    "description": null, 
    "id": "58763fb804460c787949f25e", 
    "media_url": null, 
    "options": [
      {
        "label": "Yes, that helped", 
        "payload": "class-58763fb804460c787949f25e-y"
      }, 
      {
        "label": "No, it didn't work", 
        "payload": "class-58763fb804460c787949f25e-n"
      }
    ], 

  }, 
  "session_id": "1234567890",
  "progress": 0.90
}
```
If the user confirms the solution by sending the reply as in previous steps, the session is complete a new one can be started. 

If the proposed solution doesn't fix the user's problem the user should be asked to give **feedback** with the correct solution. This would be the final request of the session, after which the session is closed:

### Request
```json
POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id/query/"

Headers:
"Content-Type: application/json; charset=utf-8"

POST body:
{
	"session_id": "1234567890",
	"message": {
		"text": "The user name was actually wrong, not the password."
	}
}
```
```bash
curl -X POST "https://admin.fredknows.it/session/api/expert-id/your-expert-id/query/" -H "Content-Type: application/json" -d '{
	"session_id": "1234567890",
	"message": {
		"text": "The user name was actually wrong, not the password."
	}
}'
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

