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

```
return 'This is a test.';
```

Now you will create another custom function that will call the URL of your new custom function. We will call our function simply with an API KEY. In your new function add the following:

```
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
```
execute_function = invokeurl [
  url: 'YOUR_SERVERLESS_FUNCTION_URL_WITH_OAUTH2'
  type: GET
  connection: 'YOUR_ZOHO_CRM_CONNECTION'
];

info execute_function;
```


When you execute this function, you should see a JSON output similar to to the following:

```
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

To use URL parameters with your serverless function, add a string parameter called `test_input` to the function arguements of your serverless function.
```
---------------------------------
PARAMETERS
  test_input: STRING
---------------------------------
```

Edit your serverless function so that just returns something like the following:
```
return "Look mom! I'm returning my parameter: " + test_input;
```

Now when we execute the function, we will include `test_input` in our URL:
```
execute_function = invokeurl [
  url: 'YOUR_SERVERLESS_FUNCTION_URL_WITH_API_KEY' + '&test_input=hootenanny'
  type: GET
];

info execute_function;
```

When you execute this function with the URL parameter, you should see a JSON output similar to to the following:

```
{
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

## JSON Body Parameters

