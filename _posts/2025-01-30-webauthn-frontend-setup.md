---
layout: post
date: 2025-01-30
categories: tips, js, security 
title: WebAuthn frontend setup
---

## How to register a passkey

1. Get the settings for creating new credentials from server
2. Clean up the settings. json converts byte[] to strings so you need to manually convert them to array buffers. Function ```coerceToArrayBuffer``` is in [webauth-helpers.js](/static/examplecode/webauth-helpers.js)
    ```javascript
    options.challenge = coerceToArrayBuffer(options.challenge);
    options.user.id = coerceToArrayBuffer(options.user.id);
    options.excludeCredentials = options.excludeCredentials.map((c) =>{
        c.id = coerceToArrayBuffer(c.id);
        return c;
    });
    ```
3. Create the credentials
    ```javascript
    let credentials = await navigator.credentials.create({ publicKey: options })
    ```
4. Pull out arrays into own arrays, may be needed if data is too long
    ```javascript
    let attestationObject = new Uint8Array(newCredential.response.attestationObject);
    let clientDataJSON = new Uint8Array(newCredential.response.clientDataJSON);
    let rawId = new Uint8Array(newCredential.rawId);
    ```
5. Create object to be sent to server. Function ```coerceToBase64Url``` is in [webauth-helpers.js](/static/examplecode/webauth-helpers.js)
    ```javascript
    const data = {
        id: newCredential.id,
        rawId: coerceToBase64Url(rawId),
        type: newCredential.type,
        extensions: newCredential.getClientExtensionResults(),
        response: {
           AttestationObject: coerceToBase64Url(attestationObject),
           clientDataJSON: coerceToBase64Url(clientDataJSON),
           transports: newCredential.response.getTransports()
        }
    };
    ```
6. Send to server to be saved

## How to verify a passkey

1. Get the assertion settings from server
2. Clean up the byte[]'s in the settings
    ```javascript
    options.challenge = coerceToArrayBuffer(options.challenge);
    options.allowCredentials = options.allowCredentials .map((c) =>{
        c.id = coerceToArrayBuffer(c.id);
        return c;
    });
    ```
3. Get credentials from passkey
    ```javascript
    var credentials = await navigator.credentials.get({publicKey: options});
    ```
4. Pull out arrays into own arrays, may be needed if data is too long
    ```javascript
    let authData = new Uint8Array(credential.response.authenticatorData);
    let clientDataJSON = new Uint8Array(credential.response.clientDataJSON);
    let rawId = new Uint8Array(credential.rawId);
    let sig = new Uint8Array(credential.response.signature);
    ```
5. Create object to be sent to server
    ```javascript
    const data = {
        id: credential.id,
        rawId: coerceToBase64Url(rawId),
        type: credential.type,
        extensions: credential.getClientExtensionResults(),
        response: {
            authenticatorData: coerceToBase64Url(authData),
            clientDataJSON: coerceToBase64Url(clientDataJSON),
            signature: coerceToBase64Url(sig)
        }
    };
    ```
6. Send to server to be verified
