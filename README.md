# Recaptcha automation (normal and invisible)

This page is intended to demonstrate not only how our [API](https://github.com/imagetyperz-api/API-docs) works but also how to use it in a *real life situation* to bypass recaptcha by completing a web form with reCAPTCHA, both in browser ([selenium](http://www.seleniumhq.org/)) and pure requests. Examples are written in C# and Python to help you bypass recaptcha, for both normal and invisible recaptcha.

## Introduction

### <ins>How does recaptcha work ?</ins>

- As most of you already know, the google captcha is different from other  captchas. They ask you to select multiple images... sometimes the images you select refresh after clicking, sometimes they don't, making it  trickier then it used to be in the back old days. Another way they do it is instead of showing you the captcha when the page loaded, it shows it only when you click the submit button, which is called invisible  recaptcha.                            
- After the captcha is completed, g-response-code (*a key/string*) is being sent with the request as a parameter when the form is submitted.                            
- Whichever recaptcha it is, our recaptcha solver service can help you  bypass it. There are two things we need from you in order to solve it  and send you the g-response-code to bypass google recaptcha.                            

### <ins>API usage</ins>

- In order to for us to give you the g-response-code, we need two things from your end:
  - **page_url** - the URL where captcha was encountered
  - **sitekey** - a key/string that can be gathered from the source of the page, where captcha was found
  - **type** - required only when reCAPTCHA is invisible. Has to be set to 2

- You submit the page_url and site using our API (*easiest to use our [libraries](https://github.com/imagetyperz-api/)*) to our servers. Once submitted, we return a **captcha ID**. That                            captcha ID is used to get the **g-response-code**. It takes few seconds until the g-response-code is available after submission, for our workers to complete it.
- Gathering the g-response-code from us, now you can bypass the page on  which you encountered the captcha, either using a browser or                             by submitting the form directly with  requests.
- Check our captcha service [recaptcha solver API page](https://github.com/imagetyperz-api/API-docs) to find out more about our recaptcha solver API.



## Recaptcha solver Automation - browser and requests (C# & Python)

### <ins>Recaptcha v2 - normal - browser</ins>

- We have a page set up for testing the normal recaptcha which can be found [here](https://imagetyperz.xyz/automation/recaptcha-v2.html). It's a simple page though, that shows recaptcha too. It requires a  username, password and captcha to be completed. Once the form is  submitted the username, password and g-response-code are sent with a  POST request to the same page, that checks if the g-response-code is OK  and returns accordingly.

```html
<html lang="en">
<head>
<title>reCAPTCHA v2 automation</title>
<style>
    input{
        margin: 4px;
    }
</style>
</head>
</body>
<h2>Register</h2>
</hr>
<form action="recaptcha-verify.php" method="POST">
<label for="username">Username:</label>
<input type="text" name="username" id="username" />
</br>
<label for="password">Password:</label>
<input type="text" name="password" id="password" />
</br>
<div style="margin-left: 12px;" class='g-recaptcha' data-sitekey='6Lcz5HQlAAAAAFoGDwrd5tYs97p2mxt4jdgxKQob'></div>
</br>
<input type="submit" value="Register">
</form>
<script src='https://www.google.com/recaptcha/api.js'></script>
</body>
</html>
```

- The important part here is this:

```html
<div style="margin-left: 12px;" class='g-recaptcha' data-sitekey='6LdXeIYUAAAAAFmFKJ6Cl3zo4epRZ0LDdOrYsvRY'></div>
```

- In fact, this is the only important thing: 6LdXeIYUAAAAAFmFKJ6Cl3zo4epRZ0LDdOrYsvRY, which is the sitekey. This is  the second thing that's needed for us, to complete it. The first one  would be the page URL on which it captcha was found, which would be: https://imagetyperz.xyz/automation/recaptcha-v2.html
- To understand things better, let's take a look on how we could bypass [Recaptcha](https://imagetyperz.xyz/automation/recaptcha-v2.html) on this page in [C# using selenium browser by using our captcha solver](https://imagetyperz.xyz/automation/recaptcha-v2.html)

```csharp

d.Navigate().GoToUrl(TEST_PAGE_NORMAL);               // go to normal test page
// complete regular data
d.FindElementByName("username").SendKeys("my-username");
d.FindElementByName("password").SendKeys("password-here");
Console.WriteLine("[+] Completed regular info");
// ---------------------

// get sitekey
string site_key = d.FindElementByClassName("g-recaptcha").GetAttribute("data-sitekey");    
Console.WriteLine(string.Format("[+] Site key: {0}", site_key));

// complete captcha

Console.WriteLine("[+] Waiting for recaptcha to be solved ...");
ImageTypersAPI i = new ImageTypersAPI(IMAGETYPERS_TOKEN);
Dictionary p = new Dictionary();
p.Add("page_url", TEST_PAGE_NORMAL);
p.Add("sitekey", site_key);
string recaptcha_id = i.submit_recaptcha(p);       // submit recaptcha info
// while in progress, sleep for 10 seconds
while (i.in_progress(recaptcha_id)) { Thread.Sleep(10000); }
string g_response_code = i.retrieve_captcha(recaptcha_id);

Console.WriteLine(string.Format("[+] Got g-response-code: {0}", g_response_code));

// set g-response-code in page source (with javascript)
IJavaScriptExecutor e = (IJavaScriptExecutor)d;
string javascript_code = string.Format("document.getElementById('g-recaptcha-response').innerHTML = '{0}';", g_response_code);
e.ExecuteScript(javascript_code);
Console.WriteLine("[+] Code set in page");

// submit form
d.FindElementByTagName("form").Submit();
Console.WriteLine("[+] Form submitted");
```

1. First we open a browser and go to our test page
2. The regular fields are completed first
3. We get the sitekey from the page source
4. Send **sitekey and page_url** to our [captcha](https://imagetyperz.xyz/automation/recaptcha-v2.html) service and wait for completion
5. Once completed, look for the element with the HTML ID **g-recaptcha-response** and set it's **innerHTML** value to be the **g-response-code** you just received from us.

### <ins>Recaptcha v2 - normal - requests</ins>

- For bypassing the same page with requests, code is very similar

```csharp
Console.WriteLine("[+] Getting sitekey from test page...");
string resp = get(TEST_PAGE_NORMAL);    // download page first (to get sitekey)
HtmlDocument d = new HtmlDocument();
d.LoadHtml(resp);

// get sitekey
string site_key = d.DocumentNode.SelectSingleNode("//div[@class='g-recaptcha']").GetAttributeValue("data-sitekey", "");
Console.WriteLine(string.Format("[+] Site key: {0}", site_key));

// complete captcha
Console.WriteLine("[+] Waiting for recaptcha to be solved ...");
ImageTypersAPI i = new ImageTypersAPI(IMAGETYPERS_TOKEN);
Dictionary p = new Dictionary();
p.Add("page_url", TEST_PAGE_NORMAL);
p.Add("sitekey", site_key);
string recaptcha_id = i.submit_recaptcha(p);       // submit recaptcha info
// while in progress, sleep for 10 seconds
while (i.in_progress(recaptcha_id)) { Thread.Sleep(10000); }
string g_response_code = i.retrieve_captcha(recaptcha_id);
Console.WriteLine(string.Format("[+] Got g-response-code: {0}", g_response_code));

// create post request data
string data = string.Format(
    "username=my-username&" +
    "password=password-here&" +
    "g-recaptcha-response={0}",
    g_response_code);

// submit
string response = post(TEST_PAGE_REQUESTS_VERIFY, data);
Console.WriteLine(string.Format("[+] Response: {0}", response));
```

- For submitting the form with requests,  we have to get the `action` parameter value, of the form. That is the  URL to which we have to submit the data. The `method` parameter tells  with what method the HTTP request should sent: GET, POST, PUT, etc                                

  ### [Download example](https://github.com/imagetyperz-api/automation-csharp)



### <ins>Recaptcha v2 - invisible - browser</ins>

- We've created an automation test page for invisible reCATPCHA as well, found here: [ https://imagetyperz.xyz/automation/recaptcha-invisible.html ](https://imagetyperz.xyz/automation/recaptcha-invisible.html)

- [When it comes to the invisible captcha ](https://imagetyperz.xyz/automation/recaptcha-invisible.html)[bypassing](https://imagetyperz.xyz/automation/recaptcha-invisible.html), it's not very different compared to the normal one. The difference is that instead of getting the **sitekey** from a *<div>*, it is gathered from a *<button>*.
- To make things more clear, let's look at how the **HTML form** looks now, with the invisible captcha:

```html
<html lang="en">
<head>
    <title>reCAPTCHA v2 (invisible) automation    </title>
    <style>
    input {
        margin: 4px;
    }
    </style>
</head>
<body>
<h2>Register    </h2>
<hr>
<form action="recaptcha-verify.php" method="POST">
    <label for="username">Username:    </label>
    <input type="text" name="username" id="username"/>
    <br>
    <label for="password">Password:    </label>
    <input type="text" name="password" id="password"/>
    <br>
    <button
        class="g-recaptcha btn btn-primary btn-link btn-wd btn-lg"
        data-sitekey="6LeX4nQlAAAAAPUU70t2D9324_iFbg0VsoLuLLCy"
        data-callback="check">
    Submit
    </button>
</form>
<script src='https://www.google.com/recaptcha/api.js'>    </script>
<script
    src="https://code.jquery.com/jquery-3.3.1.min.js"
    integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8="
    crossorigin="anonymous">    </script>
<script>
function check(token) {
    if (!window.jQuery) return alert('wait for page to load');
    var username = $('#username').val();
    if (!username) return alert('Username is missing');
    var password = $('#password').val();
    if (!password) return alert('Password is missing');

    $.ajax({
        url: '/automation/recaptcha-verify.php',
        type: 'POST',
        dataType: 'json',
        data: {
            username: username,
            password: password,
            'g-recaptcha-response': token,
            invisible: true
        }
    }).done(function (data) {
        if(data.status === 'error') return alert('Error: ' + data.details);
        $('body').empty();
        $('body').append($('    <h2 style="color: green;">Registered !    </h2>'))
    }).fail(function (err) {
        // something went wrong
        var se = '';
        try {
            se = (err.responseJSON.error || err.statusText || err);
        } catch (err) {
        }
        alert('Error: ' + se);
    });
}
</script>
</body>
</html>
```

- We don't have the *<div>* anymore, but we have the submit button, integrated with the captcha. It contains the **sitekey** that we need. When the button is pressed, the captcha appears on page. After completion, the form is submited through a JavaScript method. The request goes again with the **g-response-code** as it did before with the normal captcha.                            

- Here's how we'll do it using a browser:

```csharp
d.Navigate().GoToUrl(TEST_PAGE_INVISIBLE);               // go to normal test page
// complete regular data
d.FindElementByName("username").SendKeys("my-username");
d.FindElementByName("password").SendKeys("password-here");
Console.WriteLine("[+] Completed regular info");
// ---------------------

// get sitekey
string site_key = d.FindElementByClassName("g-recaptcha").GetAttribute("data-sitekey");
string callback_method = d.FindElementByClassName("g-recaptcha").GetAttribute("data-callback");
Console.WriteLine(string.Format("[+] Site key: {0}", site_key));
Console.WriteLine(string.Format("[+] Callback method: {0}", callback_method));

// complete captcha
Console.WriteLine("[+] Waiting for recaptcha to be solved ...");
ImageTypersAPI i = new ImageTypersAPI(IMAGETYPERS_TOKEN);
Dictionary p = new Dictionary();
p.Add("page_url", TEST_PAGE_NORMAL);
p.Add("sitekey", site_key);
p.Add("type", "2");
string recaptcha_id = i.submit_recaptcha(p);       // submit recaptcha info
// while in progress, sleep for 10 seconds
while (i.in_progress(recaptcha_id)) { Thread.Sleep(10000); }
string g_response_code = i.retrieve_captcha(recaptcha_id);
Console.WriteLine(string.Format("[+] Got g-response-code: {0}", g_response_code));

// set g-response-code in page source (with javascript)
IJavaScriptExecutor e = (IJavaScriptExecutor)d;
string javascript_code = string.Format("document.getElementById('g-recaptcha-response').innerHTML = '{0}';", g_response_code);
e.ExecuteScript(javascript_code);
Console.WriteLine("[+] Code set in page");
// submit form
e.ExecuteScript(string.Format("{0}(\"{1}\");", callback_method, g_response_code));
Console.WriteLine("[+] Callback function executed");
```

1. Here, we first do a request on the page itself, to get the **sitekey** from the *<button>*
2. Another thing that's needed is here the callback method. This can be also gathered, similar to how we got the sitekey 
3. Once we have that, we make use the [**API**](https://github.com/imagetyperz-api/API-docs) to send the **page_url** and **sitekey** and we set the **type** to 2, which tells our captcha service it's an invisible reCAPTCHA
4. As soon as we have the **gresponse**, we can call the callback method, with one argument, that being the gresponse

### <ins>Recaptcha v2 - invisible - requests</ins>

- For bypassing the same page with requests using our captcha solver, code is very similar

```csharp
Console.WriteLine("[+] Getting sitekey from test page...");
string resp = get(TEST_PAGE_INVISIBLE);    // download page first (to get sitekey)
HtmlDocument d = new HtmlDocument();
d.LoadHtml(resp);

// get sitekey
string site_key = d.DocumentNode.SelectSingleNode("//button").GetAttributeValue("data-sitekey", "");
Console.WriteLine(string.Format("[+] Site key: {0}", site_key));

// complete captcha
Console.WriteLine("[+] Waiting for recaptcha to be solved ...");
ImageTypersAPI i = new ImageTypersAPI(IMAGETYPERS_TOKEN);
Dictionary p = new Dictionary();
p.Add("page_url", TEST_PAGE_NORMAL);
p.Add("sitekey", site_key);
p.Add("type", "2");
string recaptcha_id = i.submit_recaptcha(p);       // submit recaptcha info
// while in progress, sleep for 10 seconds
while (i.in_progress(recaptcha_id)) { Thread.Sleep(10000); }
string g_response_code = i.retrieve_captcha(recaptcha_id);
//Console.Write("CODE:"); Console.ReadLine(); string g_response_code = File.ReadAllText("g-response.txt");        // get manually
Console.WriteLine(string.Format("[+] Got g-response-code: {0}", g_response_code));

// create post request data
string data = string.Format(
    "username=my-username&" +
    "password=password-here&" +
    "g-recaptcha-response={0}",
    g_response_code);

// submit
string response = post(TEST_PAGE_REQUESTS_VERIFY, data);
Console.WriteLine(string.Format("[+] Response: {0}", response));
```

1. Very similar to the normal  reCAPTCHA submission with requests. Main difference is that the  submisison to our captcha service has to be with **type** set to 2
2. The other difference is that the sitekey is gathered from a **button** and not **div**

### [Download example](https://github.com/imagetyperz-api/automation-csharp)

### <ins>Summary</ins>

We have examples to bypass normal recaptcha and invisible recaptcha in  both C# and Python. Each language has 4 ways of completing the captcha,  and that is:

1. Browser (selenium) normal reCAPTCHA v2                                 
2. Requests normal reCAPTCHA v2                                 
3. Browser invisible reCAPTCHA                                 
4. Requests invisible reCAPTCHA                                



## Download

### [Download C#](https://github.com/imagetyperz-api/automation-csharp)

### [Download Python](https://github.com/imagetyperz-api/automation-python)
