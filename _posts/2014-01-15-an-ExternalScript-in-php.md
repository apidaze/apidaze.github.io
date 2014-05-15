---
layout: post
title: An ExternalScript in PHP
---

Here is a simple example of an ExternalScript that will place incoming calls into an audio conference.

## Set up your ExternalScript URL

We''ll assume that your ExternalScript will be accessible from Heroku at http://hidden-ridge-9239.herokuapp.com. Hosting your PHP code at Heroku is easy even if you're not familiar with it : [https://devcenter.heroku.com/articles/getting-started-with-php](https://devcenter.heroku.com/articles/getting-started-with-php).

In order to have your calls handled there, you simply need to instruct APIdaze of that URL either by logging in to your APIdaze [developer account](https://developers.apidaze.io) and creating an ExternalScript in the API Reference section, or by sending a REST command.

Note that any web PaaS service (e.g. Google App Engine) or private web server will be convenient to host your ExternalScript. APIdaze just needs to access its URL.

Creating an ExternalScript with `cURL` :

    $ curl -v -X POST "https://api2.apidaze.io/d3adb33f/externalscripts" -d 'api_secret=223dc42e78cfb84bf618b68069e11986&name=myexternalscript&url=http://hidden-ridge-9239.herokuapp.com'

## PHP 
{% highlight php %}
<?php
header('Content-Type: text/xml; charset=utf-8');

echo "
<document>
 <work>
  <answer/>
  <conference>myconference</conference>
  <hangup/>
 </work>
</document>";
?>
{% endhighlight %}

And that's it ! Now every incoming calls will be routed to an audio conference named `myconference`. If you access this conference from WebRTC, you'll also be able to get more information to your JavaScript codec like who's entering/leaving the conference and who is talking.

