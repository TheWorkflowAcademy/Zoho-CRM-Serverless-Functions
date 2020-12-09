# **Zoho CRM Serverless Functions**
Serverless functions in Zoho CRM allow you to execute a function via REST API. There are many practical use cases for serverless functions within Zoho CRM, including webhook consumption, removing duplicate code, and system data migrations.

This tutorial will cover both using and implementing Zoho CRM serverless functions. There are four main methods for using and implementing serverless functions: 
1. [No Parameters](#no-parameters)
2. [URL Parameters](#url-parameters)
3. [Formatted Parameters](#formatted-parameters)
4. [JSON Body Parameters](#json-body-parameters)

## Setting Up Serverless Functions
To setup a serverless function:
1. Go to **Setup** -> **Functions** -> **New Function**
2. Name the function and under Category select **Standalone**. Click **Next**.
3. You'll be taken to the Deluge code editor. Click **Save**.
4. Find your new function in the list of functions.
5. Select the **(...)** on your function and select **REST API**.
6. Toggle the API Key to enabled, copy the URL, and click Save.

Your custom function can now act as a serverless function. You can use this URL you copied in a http request or `invokeurl` function. 

## No Parameters
We will first show how to execute a serverless function without any input parameters. This is the easiest way to execute these functions. We will cover how to execute these functions with two methods of authentication: API Keys and OAuth 2. 

Edit your serverless function so that it is just:

```javascript
/* SERVERLESS FUNCTION */
return 'This is a test.';
```

Now you will create another custom function that will call the URL of your new custom function. We will call our function simply with an API KEY. In your new function add the following:

```javascript
/* REQUESTING FUNCTION */
execute_function = invokeurl [
  url: 'YOUR_SERVERLESS_FUNCTION_URL_WITH_API_KEY'
  type: GET
];

info execute_function;
```
Alternatively you may want to execute your function with OAuth 2.0. In this case you will need to get the OAuth 2.0 URL of your function by going into the settings of your serverless function. To do this, click the (...) on your funciton, then REST API, toggle OAuth2 to enabled, copy the URL, and click save. To execute this function within Zoho CRM, you will need to create a custom API connection to Zoho CRM with the following scopes:
- `ZohoCRM.functions.execute.READ`
- `ZohoCRM.functions.execute.CREATE`

You can now execute the serverless function with the following:
```javascript
/* REQUESTING FUNCTION */
execute_function = invokeurl [
  url: 'YOUR_SERVERLESS_FUNCTION_URL_WITH_OAUTH2'
  type: GET
  connection: 'YOUR_ZOHO_CRM_CONNECTION'
];

info execute_function;
```


When you execute this function, you should see a JSON output similar to to the following:

```javascript
/* OUTPUT */
{
  "code": "success",
  "details": {
    "output": "This is a test",
    "output_type": "string",
    "id": "99708000000544001"
  },
  "message": "function executed successfully"
}
```

Notice in the `output` value is the same as the return string of the serverless function. You will be able to access this data simply with `execute_function.get('details').get('output');`.

## URL Parameters
For the rest of the tutorial, we will use API Keys to execute our functions for simplicity.

Adding URL parameters is the simplest way to pass data into serverless functions. It is best when you have a few parameters and security of the data is not a big concern. If you need better security and more parameters, using JSON Body Parameters is your best option.

To use URL parameters with your serverless function, add a string parameter called `test_input` to the function arguments of your serverless function.
```
---------------------------------
PARAMETERS
  test_input: STRING
---------------------------------
```

Edit your serverless function so that just returns something like the following:
```javascript
/* SERVERLESS FUNCTION */
return "Look mom! I'm returning my parameter: " + test_input;
```

Now when we execute the function, we will include `test_input` in our URL:
```javascript
/* REQUESTING FUNCTION */
execute_function = invokeurl [
  url: 'YOUR_SERVERLESS_FUNCTION_URL_WITH_API_KEY' + '&test_input=hootenanny'
  type: GET
];

info execute_function;
```

When you execute the function calling the serverless function with the URL parameter, you should see a JSON output similar to to the following:

```javascript
{
/* OUTPUT */
  "code": "success",
  "details": {
    "output": "Look mom! I'm returning my parameter: hootenanny",
    "output_type": "string",
    "id": "99708000000544001"
  },
  "message": "function executed successfully"
}
```

## Formatted Parameters
This method for calling a serverless function is done by formatting your variables on the request side, then passing them into your serverless function. 

First, let's update our serverless function. We will update our parameters and our return string.
```
---------------------------------
PARAMETERS
  FIRST_NAME: String
  LAST_NAME: String
  EMAIL: String
---------------------------------
```
```javascript
/* SERVERLESS FUNCTION */
return "First Name: " + FIRST_NAME + " | Last Name: " + LAST_NAME + " | Email: " + EMAIL;
```

Now, let's update the function that is making the request to the serverlss function.

```javascript
/* REQUESTING FUNCTION */
// Construct Parameters. Note: These must match the name and type of the parameters in the Serverless Function.
parameters = Map();
parameters.put("FIRST_NAME","Jane");
parameters.put("LAST_NAME","Doe");
parameters.put("EMAIL", "jane.doe@example.com");

// Format the Parameters map to meet API's required parameters. Note: We arbitrarily use the words "entry" and "entries" to describe the formatted parameters.
entries = list();
for each key in parameters.keys()
{
	entry = Map();
	entry.put("paramName",key);
	entry.put("content", parameters.get(key));
	entry.put("stringPart", "true");
	entries.add(entry);
}
```

Put the formatted parameters into the `files` argument in the invokeurl function as seen below:
```javascript
/* REQUESTING FUNCTION */
// Call the serverless function. Note: The "files" field allows Lists to be submitted, where "parameters" only accepts Map objects.
execute_function = invokeurl
[
	url: 'YOUR_SERVERLESS_FUNCTION_URL_WITH_API_KEY'
	type: POST
	files: entries
];
info execute_function;
```

When you execute the serverless function via API, you should see results similar to the following: 
```javascript
/* OUTPUT */
{
  "code": "success",
  "details": {
    "output": "First Name: Jane | Last Name: Doe | Email: jane.doe@example.com",
    "output_type": "string",
    "id": "99708000000544001"
  },
  "message": "function executed successfully"
}
```

## JSON Body Parameters
JSON Body parameters are a powerful and flexible option for passing parameters into a serverless function. This is the method you will use to consume webhooks from external services. Using JSON body parameters lets you send any JSON data to the serverless function for consumption. You will not setup parameters for specific data in the serverless function, but a single, specific parameter `crmAPIRequest` (must be exact) which holds the data for the entire request.

This request contains information about the entire HTTP request sent to the function, not just the data.

Here is how you set this up in a serverless function:
```
---------------------------------
PARAMETERS
  crmAPIRequest: String
---------------------------------
```
```javascript
/* SERVERLESS FUNCTION */
request = crmAPIRequest.toMap();
return request;
```

Now, let's call our new serverless function. We will construct a map object and simply pass it through the `parameters` argument in the `invokeurl` function. Be sure to cast the parameters to a String with `toString()`.
```javascript
/* REQUESTING FUNCTION */
params = Map();
params.put('FIRST_NAME','Jane');
params.put('LAST_NAME','Doe');
params.put('EMAIL','janedoe@gmail.com');
// Call the serverless function. Note: The "files" field allows Lists to be submitted, where "parameters" only accepts Map objects.
execute_function = invokeurl
[
	url: "https://www.zohoapis.com/crm/v2/functions/a/actions/execute?auth_type=apikey&zapikey=1003.00715e94a0845f712efe9d4661c73d8b.1431ee69891b8d90aa91f1fefea7e4cc"
	type: POST
	parameters: params.toString()
];
info execute_function;
```

The crmAPIRequest parameter that you can use within your serverless function will contain the below data. You may access any of this information in your serverless function. The most important is the `body` which contains the data you sent through the `parameters` argument.
```javascript
/* INPUT */
{
  "file_content": null,
  "headers":{ ... },
  "auth_type": "apikey",
  "method": "POST", 
  "user_info":{ ... },
  "record":{},
  "params":{},
  "body": "{\"FIRST_NAME\":\"Jane\",\"LAST_NAME\":\"Doe\",\"EMAIL\":\"janedoe@gmail.com\"}"
}
```
