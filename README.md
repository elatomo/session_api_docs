# Fredknows.it expert API

[Fredknows.it](https://www.fredknows.it) lets you build your own AI expert.

This repository contains the documentation for the expert API.

A reference jquery implementation is also available on our public [jquery expert repository](https://github.com/fredknowsit/jquery-expert).

## Quickstart

The base API expert URL is:
`https://admin.fredknows.it/session/api/expert-id/<your-expert-id>/`

Where `<your-expert-id>` is the identifier of your expert.

For the purpose of this tutorial, we will use the sample [zooki animal expert](https://expert.fredknows.it/5887544647657bc7145ea94c).

Let's **start a new session** by providing a unique session identifier (`session_id`):

```shell
curl -X POST https://admin.fredknows.it/session/api/expert-id/5887544647657bc7145ea94c/query/ \
  -H 'Content-Type: application/json' \
  -d '{"session_id": "123456789"}'
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
        ]
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

Let's tell our _animal_ expert that the animal we are thinking has _0 legs_. For
that, we'll send the payload `feature-57c82a4d04460c3ca6350ef7-y`:

```shell
curl -X POST https://admin.fredknows.it/session/api/expert-id/5887544647657bc7145ea94c/query/ \
  -H 'Content-Type: application/json' \
  -d '{"session_id": "123456789", "postback": {"payload": "feature-57c82a4d04460c3ca6350ef7-y"}}'
```

Which returns:

```json
{
    "session_id": "123456789",
    "result": {
        "id": "57c82a4d04460c3ca6350efe",
        "type": "feature",
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
        ]
    },
    "progress": 0.04
}
```

As you can see, the expert replied with another **feature** (_question_), and three possible payloads.

Our expert will keep on asking features until it:

- Finds a _class_ (a _solution_)
- Runs out of questions.

A **class** response will look like this:

```json
{
    "session_id": "123456789",
    "result": {
        "id": "5811bca73a641c69dd85ac0d", 
        "type": "class",
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
        ]
    }, 
    "progress": 0.95
}
```

We could confirm the class proposed by our expert sending the payload `class-5811bca73a641c69dd85ac0d-y`:

```shell
curl -X POST https://admin.fredknows.it/session/api/expert-id/5887544647657bc7145ea94c/query/ \
  -H 'Content-Type: application/json' \
  -d '{"session_id": "123456789", "postback": {"payload": "class-5811bca73a641c69dd85ac0d-y"}}'
```

Which will return the following response:

```json
{
    "session_id": "lep1zlh78fd2kked66ep45",
    "result": {
        "id": "5811bca73a641c69dd85ac0d", 
        "type": "end of game",
        "title": "Class feedback was correctly submitted", 
        "description": "", 
        "media_url": "", 
        "options": []
    }, 
    "progress": 1.0
}
```

As you can see, the result _type_ of this response is `end of game`. We will
find out more about possible response types in the next section.

## Result types

| Result type               | Description                                        |
|---------------------------|----------------------------------------------------|
| `class`                   | A class (solution).                                |
| `feature`                 | A feature (question).                              |
| `no-good-question-to-ask` | The expert found no good questions to ask.         |
| `max-questions-reached`   | The expert reached the maximum limit of questions. |
| `no-question-no-solution` | The expert exhausted all available questions.      |
| `end of game`             | The game is finished.                              |
| `closed`                  | The session is closed.                             |


## Retrieving project custom texts

Project _custom texts_ defined in the [admin panel](https://admin.fredknows.it) could be retrieved sending a `GET` request to the base expert endpoint:

```
http https://admin.fredknows.it/session/api/expert-id/5887544647657bc7145ea94c/
```

## Need help? Found a bug?

Just [submit an issue](https://www.github.com/fredknowsit/session_api_docs/issues) to this repository if you need any help.
