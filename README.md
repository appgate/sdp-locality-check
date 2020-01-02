## Disclaimer
© 2019, AppGate, Inc.  All rights reserved. 
Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met: (a) redistributions of source code must retain the above copyright notice, this list of conditions and the disclaimer below, and (b) redistributions in binary form must reproduce the above copyright notice, this list of conditions and the disclaimer below in the documentation and/or other materials provided with the distribution.
THE CODE AND SCRIPTS POSTED ON THIS WEBSITE ARE PROVIDED ON AN “AS IS” BASIS AND YOUR USE OF SUCH CODE AND/OR SCRIPTS IS AT YOUR OWN RISK.  APPGATE DISCLAIMS ALL EXPRESS AND IMPLIED WARRANTIES, EITHER IN FACT OR BY OPERATION OF LAW, STATUTORY OR OTHERWISE, INCLUDING, BUT NOT LIMITED TO, ALL WARRANTIES OF MERCHANTABILITY, TITLE, FITNESS FOR A PARTICULAR PURPOSE, NON-INFRINGEMENT, ACCURACY, COMPLETENESS, COMPATABILITY OF SOFTWARE OR EQUIPMENT OR ANY RESULTS TO BE ACHIEVED THEREFROM.  APPGATE DOES NOT WARRANT THAT SUCH CODE AND/OR SCRIPTS ARE OR WILL BE ERROR-FREE.  IN NO EVENT SHALL APPGATE BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, RELIANCE, EXEMPLARY, PUNITIVE OR CONSEQUENTIAL DAMAGES, OR ANY LOSS OF GOODWILL, LOSS OF ANTICIPATED SAVINGS, COST OF PURCHASING REPLACEMENT SERVICES, LOSS OF PROFITS, REVENUE, DATA OR DATA USE, ARISING IN ANY WAY OUT OF THE USE AND/OR REDISTRIBUTION OF SUCH CODE AND/OR SCRIPTS, REGARDLESS OF THE LEGAL THEORY UNDER WHICH SUCH LIABILITY IS ASSERTED AND REGARDLESS OF WHETHER APPGATE HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH LIABILITY.   

# Overview
When deciding whether to allow or block traffic (based on location), you need to detect whether an end user device is located in a specific network. There are different ways to do this. In this scenario, we show how to detect the network with the help of an https service that is located and reachable only on the local network.

We encourage you are familiar with the [AppGate Extension](https://appgate.github.io/appgate-extensions/) before continuing.


## Approach
1. Device script: detect local presence, e.g., creates a device claim for its result. 
2. Condition: uses the device claim, looking for the matching value.
3. The condition is attached to entitlements that is location controlled. 

 
## How It Works
With the help of a device script, the client detects whether the device is in a certain location or not. To make this adequately secure so the end user cannot easily tamper with it (i.e., forge network presence when he is not), some measures are required.

In this example, we use an application that runs in the AppGate client called a detector that attempts to retrieve a web server's certificate and create a fingerprint of it.

During that process, the certificate chain is verified and assures the trust of the server. The result is the fingerprint is shipped back to AppGate where it becomes the value of the claim we defined.

 
### The detector binary
You find the latest binary under the [latest release](https://github.com/appgate/appgate-locality/releases/latest) for Windows, OSX and Linux.  In short, these are the basic options you can supply at the command line:

    -ext
        Extends the output by skipVerify, valid, issuer, subject, fingerprint
    -help
        Display usage
    -port string
        The port for the URL to detect (default "443")
    -skipVerify
        Skip CA chain and hostname verification
    -url string
        The URL (don't type https://) to detect (TLS/https only)

Upload the binary to the device script section.

### Map a new on-demand device claim
On the identity provider, add a new device claim mapping. Do it once on each platform you want to roll out. Example for Windows:

* Device Script: `detector.exe`
* Arguments: `-url litchi.packnot.int`
* Claim Name: `detect_github`
* Platform: `All Windows Devices`
 

### Create a condition
In this step, we validate the fingerprint in a condition, and capture that from a device local to that server. For this purpose, run the binary from the command line. Example: 

    detector -url litchi.packnot.int
```json    
    {
        "finger_print": [       
            "SHA256::85:AB:73:C7:C1:7A..etc"
        ]
    }
```
 
We assume the server can only be reached from the local network.

A very basic JavaScript for a condition could look as the following:

```js
var fingerprint = "SHA256::85:AB:73:C7:C1:7A:2B:43:10:89:DA...etc";
var av_json = JSON.parse(claims.device.detect_litchi_packnot_int);
if( av_json.finger_print[0] == fingerprint) { return true; }
return false;
```

The condition now verifies that the returned fingerprint matches the fingerprint we trust that is coded into the condition. Attach the condition to an entitlement.