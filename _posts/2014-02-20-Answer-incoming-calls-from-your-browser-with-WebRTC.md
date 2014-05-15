---
layout: post
title: Answer incoming calls from your browser with WebRTC
---

Can I ring my web browser whenever someone calls me on my number, and answer directly from my web browser ? Well, at APIdaze, we do know how to ring regular phones and SIP terminals, but we must admit that we don't know how to ring web browsers. Does it matter ? Not really, because web technologies like Ajax and WebSockets are here to help pushing events to your browser. And great APIs like Google Channel API and PubNub do it very well ! 

> In the VoIP world, ringing a phone is a <em>signaling</em> function, handled by a <em>protocol</em> like SIP or XMPP. Here, this signaling function is up to the developer.

Let's take a look by the example, with Google App Engine and using the Google Channel API.

## What we need

Two things mostly :

* an API to push information to the web browser that a call is coming. This is the job of the Google Channel API, even though PubNub would fit here too.
* a technology that allows my web browser to have a direct voice chat with APIdaze's PSTN gateway. That sounds close to the definition of WebRTC !

> The useful information passed through the Google Channel API is a Unix `uuid` that actually identifies a live call on APIdaze. From the WebRTC side, we just grab the corresponding call and answer it.

## The code

We made a sample Google App Engine application (in Java) available on [GitHub](https://github.com/apidaze/webrtc_call_monitor).

The `src/webrtccallmonitor/WebRTCCallMonitorServlet.java` file handles the web routing. Depending on the HTTP parameters sent to the application, various actions are taken.

If the request comes from a web browser, display a blank page that will act as our monitoring console. The JavaScript code behind the blank page opens a new Google Channel so we're ready to receive notifications from, and connects to APIdaze's platform as well. You can check the `war/clientchannel.jsp` to get the content of this page.

Any other HTTP request is considered coming from APIdaze, and our application acts like an ExternalScript by replying with XML instructions. Let's take a closer look at those call requests from APIdaze.

If the HTTP request contains a `uuid_to_intercept` parameter, this means the call originated from the web monitoring console by the JavaScript code, so the application replies with an `<intercept/>`command. Here is the content of the corresponding `war/clientintercept.jsp` file :

    <%@ page contentType="text/xml;charset=UTF-8" language="java" %>
    <document>
     <work>
      <answer/>
      <intercept><%= request.getAttribute("uuid_to_intercept") %></intercept>
      <hangup/>
     </work>
    </document>

If the HTTP request does not contain any `uuid_to_intercept` parameter, we're handling a regular call. We need to notify this to the web browser using our Google Channel and place the call on hold  and ready for answer.

Call notification is done from the Servlet code (`src/webrtccallmonitor/WebRTCCallMonitorServlet.java`) :
{% highlight java %}
  ...
  // Notify via Google Channel API that someone is calling
  String caller_id_number = request.getParameter("caller_id_number");
  String json = "{\"uuid\": \"" + uuid + "\", \"caller_id_number\": \"" + caller_id_number + "\"}";
  this.channelService.sendMessage(new ChannelMessage(this.token, json));

  // Return XML instructions to put the call on hold (kind of)
  rd = request.getRequestDispatcher("fromapidaze.jsp");
  ...
{% endhighlight %}

Below is the content of the `war/fromapidaze.jsp` file. We're actually looping over the instructions as no `<hangup/>` instruction is set, and therefore the caller is effectively put on hold.

    <%@ page contentType="text/xml;charset=UTF-8" language="java" %>
    <document>
     <work>
      <answer/>
      <speak>Thank your for calling us. We're processing your call</speak>
      <wait>3</wait>
     </work>
    </document>

If you have any question or issue, please share with us on GitHub : [https://github.com/apidaze/webrtc_call_monitor](https://github.com/apidaze/webrtc_call_monitor) or by email support@apidaze.io. Thanks !
