---
layout: post
title: Route your Voxbone numbers to your ExternalScript
---

Any SIP trunk can connect to APIdaze to have your incoming calls processed by your ExternalScript. This post shows how you can connect Voxbone numbers to APIdaze.

> Voxbone is an international DID numbers provider that interfaces with SIP.

## From your Voxbone management interface

Select `Configure` -> `URIs` -> `All URIs`. Then fill in the following fields in the web form :

* URI : {E164}@80.248.213.82 (mandatory)
* Description : BYOT APIdaze (optional)

Then save and proceed with attaching the newly created URI to the numbers your want to route to APIdaze.

Select `Configure` -> `Numbers`, pick the numbers and choose the `BYOT APIdaze` URI under the `Voice URI` tab. Click `apply to selected` to confirm. 

You have now attached your Voxbone numbers to APIdaze, let's now a register the numbers at APIdaze.

## APIdaze

From the [developer's portal](https://developers.apidaze.io) in the `API Reference` section, select `External Number Create (BYOT)` and just enter each number you routed to APIdaze from Voxbone.

That's it ! Your numbers are now routed and managed by your ExternalScript.

