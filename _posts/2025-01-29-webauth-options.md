---
layout: post
date: 2025-01-29
categories: tips, security
title: WebAuthn options
---

## Credential creation options

These are the options you need to set is order to create new credentials from a passkey.

### Relying Party (rp)

The relying party identifies the server doing the authentication. These fields as set as you add the fido service to the app in `program.cs`.

- The ```id``` field should be a subset of the domain name of the server. e.g. ```nestalk.dev``` or ```localhost```.
    - this is set with the ```ServerDomain``` field 
- the ```name``` field should be the human friendly name for the site, this may be shown to the user when registering or verifying the passkey.
    - this is set with the ```ServerName``` field

### User (user)

This is the object to define the user being registered with the passkey

- The ```id``` field is used to identify the user in the passkey
  - if the ```user.id``` and ```rp.id``` of the registering user are already on the authenticator, the credential may be overwritten
- The ```name``` is the username of the user registering the passkey
- the ```displayName``` field is the 

### Authenticator selection (authenticatorSelection)

This is the object to tell the browser what type of passkey can be registered. These options are set in the `AuthenticatorSelection` object used in the call to create the options object.

- ```authenticatorAttachment``` determines if the browser should user platform or cross-platform passkeys
  - `platform` indicates that it should use the operating system authentication such Windows Hello or TouchId
  - `cross-platform` indicates that it should use external authentication such as a security key or authenticator app
  - If you don't specify the value it indicates the browser can use either type of attachment
- ```residentKey``` specifies if the authenticator is to create a credential that can be seen on the authenticator
  - `discouraged` specifies that only a server side credential should be created, but it will still accept a client side credential
  - `preferred` specifies that a client side credential should be created, but it will accept a server side credential
  - `required` specifies that a client side credential is required and to throw an error if it cant create one
- ```requiresResidentKey``` should be set to true only if `residentKey` is `required`
- ```userVerification``` specifies if the authenticator should perform user verification as part of creating the credential, e.g. pin or password entry or similar
  - `required` specifies that user verification must be performed
  - `preferred` specifies that user verification should be performed, but it will not fail the creation if it is not done
  - `discouraged` specifies that user verification shouldn't be performed, but the authenticator may still do it if it wants to

### Excluded credentials (excludeCredentials)

Lists the existing credentials used by the user on the server. This list is passed into the call to create the options object.
If there are any entries and if the user tries to create a new credential on the same authenticator it will throw an error. 
If there is already a credential on the authenticator, and it isn't specified in the list a new credential will be created on the authenticator overwriting the existing credential.

### Attestation (attestation)

Attestation is the process of verifying the origin of the authenticator. Performing the attestation may include calling external servers to confirm public keys are correct.
- `none` - the server is not interested in verifying the authenticator
- `indirect` - the server wants to be able to verify the authenticator but will accept an anonymized attestation
- `direct` - the server wants to verify the authenticator
- `enterprise` - the server wants to uniquely identify the authenticator so it can register it with 

### Extensions (extensions)

Extensions add additional requirements on the processing for the client and authenticator. These options are passed in an `AuthenticationExtensionsClientInputs` object to the call to create the credential options.

- `extensions` - set to true if there are any extensions set

see [WebAuthn extensions](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API/WebAuthn_extensions) for available extensions

## Assertion options

These are the options you need to set is order to authenticate a passkey

### Allowed credentials (allowCredentials)

If you specify a list of credentials then only the credentials listed can be verified. 

If you don't specify any credentials then only credentials that were created as discoverable can be verified. This means that only if the credential was created with `residentKey` set to `preferred` or `required`. This option allows a user to login without even specifying a username as the user can be matched up with the user id contained in the passkey.

### User verification (userVerification)

Specifies if the authenticator should perform user verification such as pin entry or fingerprint login. 

- `required` - verification must be performed
- `preferred` - verification should be performed but is not required
- `discouraged` - verification shouldn't be performed, but the authenticator may do it anyway.

### Authentication Extensions (extensions)

Extensions add additional requirements on the processing for the client and authenticator.

- `extensions` - set to true if there are any extensions set

