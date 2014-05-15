---
layout: post
title: Log the HTTP calls made to your ExternalScript
---

Every call made to your application, be it from the PSTN, SIP or WebRTC/Flash UDP triggers a call to your ExternalScript, where APIdaze gets the list of instructions to be run. This is why a publicly accessible URL must be attached to your ExternalScript object, using our HTTP/REST API or from the [developer portal](http://developers.apidaze.io).

There are two main reasons why a developer would log the HTTP requests made from APIdaze to his ExternalScript : get useful status codes from, and know from which source the call has been placed. Let us take a look.

## Get status codes from APIdaze

APIdaze fetches your ExternalScript at least twice : when a call starts, to gather the list of XML instructions to run, and when the last instruction of the list has been run. Some useful variables are returned by APIdaze in the latter, for instance :

* `XML_error` : any error in your ExternalScript will be reported in this variable
* `exiting` : set to true when the last XML instruction has been run
* `hangup_cause` and `bridge_status` : sent as a result of a <dial> command, and can take exmplicit values like `NORMAL_CLEARING`, `USER_BUSY`, `NO_USER_RESPONSE` to let you decide how to handle the call afterwards

## Defining the context of call

When a call is placed, how to know that a call comes from the PSTN, from a SIP account or from a web page using our WebRTC JavaScript API ? The answer is simple : use HTTP.

Every HTTP request APIdaze makes to your ExternalScript contains parameters that let your script know the context of a call.

Variables set when calling from the PSTN :

    caller_id_number
    destination_number

Variables set when calling from a SIP account :

    sip_from_uri
    caller_id_number
    destination_number

Calling from WebRTC is a little different. Let us assume that you are running a webpage that allows people to directly place a call as shown on our [developer portal](https://developers.apidaze.io/webrtc). Actually, all the variables set in the first argument of the `call` function are passed to your ExternalScript.
{% highlight js %}
...
var channel = client.call(
  {
    my_parameter_that_contains_a_number: "0033123456789",
    my_web_session_id: "AF8E74E5-3C8B-4D80-A797-3FF85F910B1D"
  },
  {
    onAnswered: function(){ console.log("Remote party answered"); },
    onRinging: function(){ console.log("Remote party is ringing"); },
    onHangup: function(){ console.log("Remote party hung up"); }
  }
);
{% endhighlight %}

In the above example, the variables `my_parameter_that_contains_a_number` and `my_web_session_id` are passed to your ExternalScript, therefore allowing you to grant or refuse the call based on who is calling.

## PHP

The code snippet below shows how to log the HTTP parameters of the requests made by APIdaze to a local file `/tmp/log-urltest.txt`. The XML instructions to run on APIdaze indicate that any incoming call from number `33987654321` are routed to `33987654321`, whereas calls from any other source are refused.
{% highlight php %}
<?php
$date = date('d-m-Y G:i:s');
$logs = "[". $date . "] START OF PARAMETERS\n";
foreach($_GET as $key => $value) {
  $logs .= "[". $date . "] " .  $key . " : " . $value ."\n";
}
$logs .= "[". $date . "] END OF PARAMETERS\n";
$fplogs = fopen('/tmp/log-urltest.txt', 'a+');
fwrite($fplogs, $logs);
fclose($fplogs);
...

header('Content-Type: text/xml; charset=utf-8');

if ($_GET['caller_id_number'] != '33987654321') {
  echo "
<document>
 <work>
  <speak>Sorry, we cannot process your call, bye.</speak>
  <hangup/>
 </work>
</document>";
} else {
  echo "
<document>
 <work>
  <dial>  
   <number>0033123456789</number>
  </dial>
  <hangup/>
 </work>
</document>";
}
?>
{% endhighlight %}

Sample content of the `/tmp/urltest.txt` log file :

    [2014-01-12 10:48:38] START OF PARAMETERS
    [2014-01-12 10:48:38] session_id : b57cd556-dc0d-11e3-89d5-07add783a6d5
    [2014-01-12 10:48:38] uuid : b57cd556-dc0d-11e3-89d5-07add783a6d5
    [2014-01-12 10:48:38] destination_number : 0033123456789
    [2014-01-12 10:48:38] caller_id_number : 33987654321
    [2014-01-12 10:48:38] caller_username : anonymous
    [2014-01-12 10:48:38] url : http://mysite.mydomain.com/myexternalscript/answer.php
    [2014-01-12 10:48:38] END OF PARAMETERS
    [2014-01-12 10:49:07] START OF PARAMETERS
    [2014-01-12 10:49:07] session_id : b57cd556-dc0d-11e3-89d5-07add783a6d5
    [2014-01-12 10:49:07] hangup_cause : NORMAL_CLEARING
    [2014-01-12 10:49:07] bridge_status : SUCCESS
    [2014-01-12 10:49:07] uuid : b57cd556-dc0d-11e3-89d5-07add783a6d5
    [2014-01-12 10:49:07] destination_number : 0033123456789
    [2014-01-12 10:49:07] caller_id_number : 33987654321
    [2014-01-12 10:49:07] caller_username : anonymous
    [2014-01-12 10:49:07] url : http://mysite.mydomain.com/myexternalscript/answer.php
    [2014-01-12 10:49:07] work_tag : <hangup/>
    [2014-01-12 10:49:07] exiting : true
    [2014-01-12 10:49:07] END OF PARAMETERS

