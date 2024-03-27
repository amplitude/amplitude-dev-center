---
title: User Privacy API
description: The User Privacy API lets you programmatically submit requests to delete all data for a set of known Amplitude IDs or User IDs.

---

The User Privacy API helps you comply with end-user data deletion requests mandated by global privacy laws such as GDPR and CCPA.
 The API lets you programmatically submit requests to delete all data for a set of known Amplitude IDs or User IDs.

--8<-- "includes/postman.md"

--8<-- "includes/auth-basic.md"

## Endpoints

| Region              | Endpoint                                                                                                             |
| ------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Standard Server     | [https://amplitude.com/api/2/deletions/users](https://amplitude.com/api/2/deletions/users)                           |
| EU Residency Server | [https://analytics.eu.amplitude.com/api/2/deletions/users](https://analytics.eu.amplitude.com/api/2/deletions/users) |

## Considerations

Keep these considerations in mind when using the User Privacy API.

- When you make a deletion request, Amplitude [emails all account admins](https://help.amplitude.com/hc/en-us/articles/360031965572-Manage-user-privacy-notifications-in-Amplitude) with the deletion details.
- Amplitude deletes all events and user properties added up until the time that job runs for each Amplitude ID in a deletion job.
- Running a deletion job for a user doesn't block new events for that user. Amplitude accepts new events from a deleted user.
- If Amplitude receives events for a deleted user, then it counts the deleted user as a new user.
 Because deletion removes all user data from Amplitude servers, Amplitude doesn't recognize the new user as the deleted user.
- To reduce resource impact and ensure high availability, Amplitude batches requests for deletions. Batch jobs are scheduled for 10-13 days after the date of the batch's first request.
 In line with GDPR article 12.3 and 17, Amplitude processes deletion requests without undue delay, within 30 days after receiving the request.
 The actual timeline for deletion depends on the complexity and number of requests Amplitude receives.
 If your data volume is large (>1BB/month), then Amplitude may need to reduce your frequency of deletion scheduling.
- You can revoke requests in the batch until three (3) days before the day the job scheduled run date. During the three (3) day period, you can't edit the batch.
 Amplitude adds deletion requests made during this time to a new batch.
- After the three (3) day period, the request's status changes to `submitted`. You can't stop the job at this point.
 The deletion process removes all data associated with the user from all Amplitude's systems, including associated recovery and back-up systems.
 After the job completes, its status changes to `done`.
- By default, deletion requests create jobs for a particular project within your organization. API calls identify this project with the api key you provide in the request. To delete a user across your entire organization, set both the `delete_from_org` and `ignore_invalid_ids` parameters to `true` in your request. The API creates one deletion job per project that contains information on the requested users. 

!!! warning "User tracking"

    Using this API doesn't prevent future user tracking for the deleted users. To learn about how to stop tracking users in your application, see the `setOptOut()` method in documentation for the Amplitude SDK you're using.

## Limits

The endpoint `/api/2/deletions/users` has a rate limit of 1 HTTP request per second. Each HTTP request can contain up to 100 `amplitude_ids` or `user_ids`.
 You can make up to 100 deletion requests per second if you batch 100 users in each request.

## Delete users

`POST /deletions/users`

Add a user for deletion using a JSON body. Specify up to 100 users at a time. You can use mix of Amplitude IDs and User IDs.

### JSON body parameter

The body parameter is required. It's the deletion request object listing the `user_ids` and `amplitude_ids` for the users to delete.

| <div class="big-column">Name</div> | Description                                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `amplitude_ids`                    | Amplitude IDs for the users to delete.                                                                                                                                                                                                                                                                                                                 |
| `user_ids`                         | User IDs for the users to delete.                                                                                                                                                                                                                                                                                                                      |
| `requester`                        | The internal user who requested the deletion. This is useful for auditing.                                                                                                                                                                                                                                                                             |
| `ignore_invalid_id`                | Boolean. Defaults to `false`, and returns a 400 error if any of the provided users are not found in the project(s) requested. With `false`, no users in the request are  added to a deletion job. If `true`, the response is a 200 success that includes a list of `invalid_ids`, and any users found in the project(s) are added to the relevant job(s). 

Amplitude recommends setting this to `true` if setting the `delete_from_org` parameter to `true`                                                                                                                                                                                                                    |
| `delete_from_org`                  | Boolean. Defaults to `false`. When `false`, it deletes the requested users from the project identified by the given api key. When `true`, it deletes the users from the entire org (all projects) instead of a single project. It inserts the requested users into a job for each project that contains data for the requested users. When `true`, Amplitude recommends you set the `ignore_invalid_ids` parameter to `true`.                                                                                                                          |
| `include_mapped_user_ids`          | When `true`, this parameter returns the valid `user_id` values that correspond to a supplied `amplitude_id`. This only changes the response object. To delete mapped users set with the [User Mapping API](https://www.docs.developers.amplitude.com/analytics/apis/aliasing-api/), include each `user_id` of the mapped user in the `user_ids` array. |

### Example request: Default parameters. Will only delete user in one project and will error if users not found 

=== "cURL"

    ```bash
    curl --location --request POST 'https://amplitude.com/api/2/deletions/users' \
    -U 'API_KEY:SECRET_KEY' \ 
    --header 'Content-Type: application/json' \
    --data-raw '{
        "amplitude_ids": [
            356896327775,
            356896327755

        ],
        "user_ids": [
            1000,
            2999
        ],
        "requester": "employee@yourcompany.com"
    }'
    ```

=== "HTTP"

    ```bash
    POST /api/2/deletions/users HTTP/1.1
    Host: amplitude.com
    Authorization: Basic API_KEY:SECRET_KEY
    Content-Type: application/json

    {
        "amplitude_ids": [
            356896327775,
            356896327755

        ],
        "user_ids": [
            1000,
            2999
        ],
        "requester": "employee@yourcompany.com"
    }
    ```

=== "JavaScript"

    ```js
    var headers = {
      'Content-Type':'application/json',
      'Accept':'application/json'

    };

    $.ajax({
      url: 'https://amplitude.com/api/2/deletions/users',
      method: 'post',

      headers: headers,
      success: function(data) {
        console.log(JSON.stringify(data));
      }
    })
    ```

=== "NodeJs"

    ```js
    const request = require('node-fetch');
    const inputBody = '{
      "amplitude_ids": [
        "amp_id_1",
        "amp_id_2",
        "..."
      ],
      "user_ids": [
        "user_id_1",
        "user_id_2",
        "..."
      ],
      "requester": "employee@yourcompany.com"
    }';
    const headers = {
      'Content-Type':'application/json',
      'Accept':'application/json'

    };

    fetch('https://amplitude.com/api/2/deletions/users',
    {
      method: 'POST',
      body: inputBody,
      headers: headers
    })
    .then(function(res) {
        return res.json();
    }).then(function(body) {
        console.log(body);
    });
    ```

=== "Ruby"

    ```ruby
    require 'rest-client'
    require 'json'

    headers = {
      'Content-Type' => 'application/json',
      'Accept' => 'application/json'
    }

    result = RestClient.post 'https://amplitude.com/api/2/deletions/users',
      params: {
      }, headers: headers

    p JSON.parse(result)
    ```

=== "Python"

    ```python
    import requests
    import json

    url = "https://amplitude.com/api/2/deletions/users"

    payload = json.dumps({
      "amplitude_ids": [
        356896327775,
        356896327755
      ],
      "user_ids": [
        1000,
        2999
      ],
      "requester": "employee@yourcompany.com"
    })
    headers = {
      'Authorization': 'Basic API_KEY:API_SECRET',
      'Content-Type': 'application/json'
    }

    response = requests.request("POST", url, headers=headers, data=payload)

    print(response.text)
    ```

=== "Java"

    ```java
    URL obj = new URL("https://amplitude.com/api/2/deletions/users");
    HttpURLConnection con = (HttpURLConnection) obj.openConnection();
    con.setRequestMethod("POST");
    int responseCode = con.getResponseCode();
    BufferedReader in = new BufferedReader(
        new InputStreamReader(con.getInputStream()));
    String inputLine;
    StringBuffer response = new StringBuffer();
    while ((inputLine = in.readLine()) != null) {
        response.append(inputLine);
    }
    in.close();
    System.out.println(response.toString());
    ```

=== "Go"

    ```go
    package main

    import (
      "fmt"
      "strings"
      "net/http"
      "io/ioutil"
    )

    func main() {

      url := "https://amplitude.com/api/2/deletions/users"
      method := "POST"

      payload := strings.NewReader(`{
        "amplitude_ids": [
            356896327775,
            356896327755

        ],
        "user_ids": [
            1000,
            2999
        ],
        "requester": "employee@yourcompany.com"
    }`)

      client := &http.Client {
      }
      req, err := http.NewRequest(method, url, payload)

      if err != nil {
        fmt.Println(err)
        return
      }
      req.Header.Add("Authorization", "Basic API_KEY:API_SECRET")
      req.Header.Add("Content-Type", "application/json")

      res, err := client.Do(req)
      if err != nil {
        fmt.Println(err)
        return
      }
      defer res.Body.Close()

      body, err := ioutil.ReadAll(res.Body)
      if err != nil {
        fmt.Println(err)
        return
      }
      fmt.Println(string(body))
    }
    ```

### Example request: Delete users from all projects in your organization

=== "cURL"

    ```bash
    curl --location --request POST 'https://amplitude.com/api/2/deletions/users' \
    -U 'API_KEY:SECRET_KEY' \ 
    --header 'Content-Type: application/json' \
    --data-raw '{
        "amplitude_ids": [
            356896327775,
            356896327755

        ],
        "user_ids": [
            1000,
            2999
        ],
        "ignore_invalid_id": "true",
        "delete_from_org": "true",
        "requester": "employee@yourcompany.com"
    }'
    ```

=== "HTTP"

    ```bash
    POST /api/2/deletions/users HTTP/1.1
    Host: amplitude.com
    Authorization: Basic API_KEY:SECRET_KEY
    Content-Type: application/json

    {
        "amplitude_ids": [
            356896327775,
            356896327755

        ],
        "user_ids": [
            1000,
            2999
        ],
        "delete_from_org": "true",
        "ignore_invalid_ids": "true",
        "requester": "employee@yourcompany.com"
    }
    ```

=== "JavaScript"

    ```js
    var headers = {
      'Content-Type':'application/json',
      'Accept':'application/json'

    };

    $.ajax({
      url: 'https://amplitude.com/api/2/deletions/users',
      method: 'post',

      headers: headers,
      success: function(data) {
        console.log(JSON.stringify(data));
      }
    })
    ```

=== "NodeJs"

    ```js
    const request = require('node-fetch');
    const inputBody = '{
      "amplitude_ids": [
        "amp_id_1",
        "amp_id_2",
        "..."
      ],
      "user_ids": [
        "user_id_1",
        "user_id_2",
        "..."
      ],
      "delete_from_org": true,
      "ignore_invalid_ids": true,
      "requester": "employee@yourcompany.com"
    }';
    const headers = {
      'Content-Type':'application/json',
      'Accept':'application/json'

    };

    fetch('https://amplitude.com/api/2/deletions/users',
    {
      method: 'POST',
      body: inputBody,
      headers: headers
    })
    .then(function(res) {
        return res.json();
    }).then(function(body) {
        console.log(body);
    });
    ```

=== "Ruby"

    ```ruby
    require 'rest-client'
    require 'json'

    headers = {
      'Content-Type' => 'application/json',
      'Accept' => 'application/json'
    }

    result = RestClient.post 'https://amplitude.com/api/2/deletions/users',
      params: {
      }, headers: headers

    p JSON.parse(result)
    ```

=== "Python"

    ```python
    import requests
    import json

    url = "https://amplitude.com/api/2/deletions/users"

    payload = json.dumps({
      "amplitude_ids": [
        356896327775,
        356896327755
      ],
      "user_ids": [
        1000,
        2999
      ],
      "delete_from_org": True,
      "ignore_invalid_ids": True,
      "requester": "employee@yourcompany.com"
    })
    headers = {
      'Authorization': 'Basic API_KEY:API_SECRET',
      'Content-Type': 'application/json'
    }

    response = requests.request("POST", url, headers=headers, data=payload)

    print(response.text)
    ```

=== "Java"

    ```java
    URL obj = new URL("https://amplitude.com/api/2/deletions/users");
    HttpURLConnection con = (HttpURLConnection) obj.openConnection();
    con.setRequestMethod("POST");
    int responseCode = con.getResponseCode();
    BufferedReader in = new BufferedReader(
        new InputStreamReader(con.getInputStream()));
    String inputLine;
    StringBuffer response = new StringBuffer();
    while ((inputLine = in.readLine()) != null) {
        response.append(inputLine);
    }
    in.close();
    System.out.println(response.toString());
    ```

=== "Go"

    ```go
    package main

    import (
      "fmt"
      "strings"
      "net/http"
      "io/ioutil"
    )

    func main() {

      url := "https://amplitude.com/api/2/deletions/users"
      method := "POST"

      payload := strings.NewReader(`{
        "amplitude_ids": [
            356896327775,
            356896327755

        ],
        "user_ids": [
            1000,
            2999
        ],
        "delete_from_org": true,
        "ignore_invalid_ids": true,
        "requester": "employee@yourcompany.com"
    }`)

      client := &http.Client {
      }
      req, err := http.NewRequest(method, url, payload)

      if err != nil {
        fmt.Println(err)
        return
      }
      req.Header.Add("Authorization", "Basic API_KEY:API_SECRET")
      req.Header.Add("Content-Type", "application/json")

      res, err := client.Do(req)
      if err != nil {
        fmt.Println(err)
        return
      }
      defer res.Body.Close()

      body, err := ioutil.ReadAll(res.Body)
      if err != nil {
        fmt.Println(err)
        return
      }
      fmt.Println(string(body))
    }
    ```

### Response

The response for a POST request contains these fields:

| <div class="big-column">Name</div> | Description                                                                                                       |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------|
| `day`                              | The day the deletion job is scheduled to begin.                                                                   |
| `status`                           | The status of the deletion job.                                                                                   |
| `amplitude_ids` and `user_ids`     | List of the Amplitude IDs to delete.                                                                              |
| `app`                              | The project or app ID. Included when the deletion request is for multiple projects.                               |
| `invalid_ids`                      | When `ignore_invalid_ids` is `true`, contains a list of users that were requested but not found in the project(s) |

The `amplitude_ids` key contains these fields:

| <div class="big-column">Name</div> | Description                                                                                                                    |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `amplitude_id`                     | The Amplitude ID of the user to be deleted.                                                                                    |
| `requester`                        | The person who requested the Amplitude ID to be deleted.                                                                       |
| `requested_on_day`                 | The day this deletion was requested.                                                                                           |
| `user_id`                          | The corresponding User ID. Included when `include_mapped_user_ids` is `true` and the `amplitude_id` are matched to `user_ids`. |

## Get deletion jobs

`/api/2/deletions/users?start_day=YYYY-MM-DD&end_day=YYYY-MM-DD`

Retrieves a list of deletion jobs scheduled in a time range. The time range should include the date you made the request on plus 30 days. For example, you made a deletion request on August 1st, 2018.
 Your deletion request should have `start_day = 2018-08-01` and `end_day = 2018-08-31`.

If the request returns no values, then no jobs are scheduled for that time range. Note: The largest permitted time range is six months.

### Example request
<!--vale off-->

=== "cURL"

    ```bash
    # You can also use wget
    curl -X GET https://amplitude.com/api/2/deletions/users?start_day=string&end_day=string \
      -H 'Accept: application/json' \
      -U API_KEY:API_SECRET
    ```

=== "HTTP"

    ```bash
    GET https://amplitude.com/api/2/deletions/users?start_day=string&end_day=string HTTP/1.1
    Host: amplitude.com
    Authorization: Basic API_KEY:API_SECRET
    Accept: application/json
    ```
=== "JavaScript"

    ```js
    var headers = {
      'Accept':'application/json'

    };

    $.ajax({
      url: 'https://amplitude.com/api/2/deletions/users',
      method: 'get',
      data: '?start_day=string&end_day=string',
      headers: headers,
      success: function(data) {
        console.log(JSON.stringify(data));
      }
    })
    ```
=== "NodeJs"

    ```js
    const request = require('node-fetch');

    const headers = {
      'Accept':'application/json'

    };

    fetch('https://amplitude.com/api/2/deletions/users?start_day=string&end_day=string',
    {
      method: 'GET',

      headers: headers
    })
    .then(function(res) {
        return res.json();
    }).then(function(body) {
        console.log(body);
    });
    ```
=== "Ruby"

    ```ruby
    require 'rest-client'
    require 'json'

    headers = {
      'Accept' => 'application/json'
    }

    result = RestClient.get 'https://amplitude.com/api/2/deletions/users',
      params: {
      'start_day' => 'string',
    'end_day' => 'string'
    }, headers: headers

    p JSON.parse(result)
    ```
=== "Python"

    ```python
    import requests
    headers = {
      'Accept': 'application/json'
    }

    r = requests.get('https://amplitude.com/api/2/deletions/users', params={
      'start_day': 'string',  'end_day': 'string'
    }, headers = headers)

    print r.json()
    ```
=== "Java"

    ```java
    URL obj = new URL("https://amplitude.com/api/2/deletions/users?start_day=string&end_day=string");
    HttpURLConnection con = (HttpURLConnection) obj.openConnection();
    con.setRequestMethod("GET");
    int responseCode = con.getResponseCode();
    BufferedReader in = new BufferedReader(
        new InputStreamReader(con.getInputStream()));
    String inputLine;
    StringBuffer response = new StringBuffer();
    while ((inputLine = in.readLine()) != null) {
        response.append(inputLine);
    }
    in.close();
    System.out.println(response.toString());
    ```

=== "Go"

    ```go
    package main

    import (
          "bytes"
          "net/http"
    )

    func main() {

        headers := map[string][]string{
            "Accept": []string{"application/json"},

        }

        data := bytes.NewBuffer([]byte{jsonReq})
        req, err := http.NewRequest("GET", "https://amplitude.com/api/2/deletions/users", data)
        req.Header = headers

        client := &http.Client{}
        resp, err := client.Do(req)
        // ...
    }
    ```
<!--vale on-->
### Query parameters

| Name    | Description                                                                                                                |
| ------- | -------------------------------------------------------------------------------------------------------------------------- |
| `start` | <span class="required">Required</span>. First hour included in data series, formatted `YYYYMMDD`. For example, `20220201`. |
| `end`   | <span class="required">Required</span>. Last hour included in data series, formatted `YYYYMMDD` For example, `20220201`.   |

### Response

The success response for a `GET` request contains these fields:

| <div class="big-column">Property</div> | Description                                                                                                                                                                                                                                                                                                                                             |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `day`                                  | The day the deletion job is scheduled to begin.                                                                                                                                                                                                                                                                                                           |
| `status`                               | The deletion job's status.  <br>  <br>**Staging**: The job hasn't started, and you can modify it. More deletion requests may get scheduled into this job and you can remove requests from this job.  <br>  <br>**Submitted**: The job is submitted to run. You can't modify it.  <br>  <br>**Done**: The job has finished running. You can't modify it.  |
| `amplitude_ids`                        | List of the Amplitude Ids of users to delete.                                                                                                                                                                                                                                                                                                             |
| `app`                                  | Project or app ID. Appears if the deletion is applied to more than one project.                                                          |                          
| `active_scrub_done_date`               | The date that the scrub has completed, and the data is no longer accessible. After this point, the system waits 5 days for any backups to be automatically cleared. The status will change to 'done' only when the backups are removed.                                                                                                                 |

The `amplitude_ids` key contains these fields:

| <div class="big-column">Name</div> | Description                                              |
| ---------------------------------- | -------------------------------------------------------- |
| `amplitude_id`                     | The Amplitude ID of the user to be deleted.              |
| `requester`                        | The person who requested the Amplitude ID to be deleted. |
| `requested_on_day`                 | The day this deletion was requested.                     |

```json
[
  {
    "day": "string",
    "amplitude_ids": [
      {
        "amplitude_id": 0,
        "requested_on_day": "string",
        "requester": "string"
      }
    ],
    "status": "string"
  }
]
```

## Delete a User from a deletion job

Removes the specified Amplitude ID from a deletion job.

`/api/2/deletions/users/AMPLITUDE_ID/YYYY-MM-DD`

### Example request

```bash
curl -X DELETE \
  'https://amplitude.com/api/2/deletions/users/AMPLITUDE_ID/JOB_START_DAY' \
    -H 'Content-Type: application/json' \
  -U API_KEY:API_SECRET
}'
```

### Path variables

| <div class="big-column">Name</div> | Description                                                                  |
| ---------------------------------- | ---------------------------------------------------------------------------- |
| `AMPLITUDE_ID`                     | Required. The `amplitude_id` to be removed from a deletion job.              |
| `JOB_START_DAY`                    | Required. Day the deletion is schedule for. YYYY-MM-DD                       |

### Response

A successful request returns a response with this schema:

| <div class="big-column">Property</div> | Description                                                 |
| -------------------------------------- | ----------------------------------------------------------- |
| `amplitude_id`                         | The Amplitude ID of the user that was removed from the job  |
| `requester`                            | The person who requested the Amplitude ID to be deleted.    |
| `requested_on_day`                     | The day this deletion was requested.                        |

```json
 {
   "amplitude_id": 1234567,
   "requested_on_day": "string",
   "requester": "string"
 }
```

## Status codes

| Code | Message      |
| ---- | ------------ |
| 200  | Success      |
| 400  | Bad Request  |
| 401  | Unauthorized |

--8<-- "includes/abbreviations.md"
