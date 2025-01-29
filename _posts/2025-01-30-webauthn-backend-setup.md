---
layout: post
date: 2025-01-30
categories: tips, c#, security
title: WebAuthn backend setup
---

1. Install nuget packages (use version 4):
    ```
    dotnet add package Fido2 --version 4.0.0-beta.13
    dotnet add package Fido2.AspNet --version 4.0.0-beta.13
    ```
2. Add table in database to store passkeys
    ```C#
    public class UserPasskey
    {
        public Guid UserId { get; set; }
        public byte[] Id { get; set; } 
        public byte[] UserHandle { get; set; } 
        public uint SignCount { get; set; } 
        public DateTime RegistrationDate { get; set; } 
        public Guid AaGuid { get; set; }  
        public byte[] PublicKey { get; set; }
        public string AttestationFormat { get; set; }
        public AuthenticatorTransport[] Transports { get; set; }
        public bool IsBackupEligible { get; set; }
        public bool IsBackedUp { get; set; }
        public byte[] AttestationObject { get; set; }
        public byte[] AttestationClientDataJson { get; set; }
        public List<byte[]> DevicePublicKeys { get; set; }
    }
    ```
3. Configure fido service, in ```Program.cs```
    ```c#
    builder.Services.AddFido2(options =>
    {
        options.ServerDomain = "localhost";
        options.ServerName = "WebAuth test";
        options.Origins = new HashSet<string>() { "https://localhost:44365/" };
        options.TimestampDriftTolerance = 300000;
        options.MDSCacheDirPath = "";
    }).AddCachedMetadataService(config =>
    {
        config.AddFidoMetadataRepository(httpClientBuilder =>
        {
        });
    });
    ```
4. Add session cookies to site, in ```Program.cs```
    ```c#
    builder.Services.AddSession(options =>
    {
        options.Cookie.HttpOnly = true;
        options.IdleTimeout = TimeSpan.FromMinutes(5);
        options.Cookie.SameSite = SameSiteMode.Strict;
    });
    ```
    ```c#
    app.UseSession();
    ```
   

## How to register a passkey

### Create settings for registering a passkey

1. Create a fido user from logged-in user
    ```c#
    var fidoUser = new Fido2User
    {
        Id = user.Id.ToByteArray(),
        DisplayName = user.UserName,
        Name = user.UserName
    };
    ```
2. Get existing keys for user
    ```c#
    var existingKeys = await _context.UserPasskeys
        .Where(x => x.UserId == user.Id)
        .Select(x =>
            new PublicKeyCredentialDescriptor(PublicKeyCredentialType.PublicKey, x.Id, x.Transports))
        .ToListAsync();
    ```
3. Create settings used for registrations
    ```c# 
    var authenticatorSelection = new AuthenticatorSelection
    {
        UserVerification = UserVerificationRequirement.Preferred,
        ResidentKey = ResidentKeyRequirement.Discouraged,
        AuthenticatorAttachment = AuthenticatorAttachment.CrossPlatform

    };
    var exts = new AuthenticationExtensionsClientInputs()
    {
        Extensions = false,
    };
    ```
4. Create the credential create options 
    ```c#
    var options = _fido.RequestNewCredential(fidoUser, existingKeys, authenticatorSelection, AttestationConveyancePreference.None, exts);
    ```
5. Save options in session
    ```c#
    HttpContext.Session.SetString("fido2.attestationOptions", options.ToJson());
    ```
6. Return options to client

### Register passkey

1. Get options from session
    ```c#
    var jsonOptions = HttpContext.Session.GetString("fido2.attestationOptions");
    var options = CredentialCreateOptions.FromJson(jsonOptions);
    ```
2. Create callback to check if a credential has been used before
    ```c#
    IsCredentialIdUniqueToUserAsyncDelegate callback = async (args, cancellationToken) =>
    {
        var passkey = await _context.UserPasskeys.FirstOrDefaultAsync(x => x.Id == args.CredentialId, cancellationToken);
        if (passkey is not null)
        {
            return false;
        }
        return true;
    };
    ```
3. Create credential
    ```c#
    var credential = await _fido.MakeNewCredentialAsync(attestationResponse, options, callback, cancellationToken: ct);
    ```
4. Save credential to database
    ```c#
    var newPasskey = new UserPasskey()
    {
        UserId = user.Id,
        Id = credential.Result.Id,
        PublicKey = credential.Result.PublicKey,
        UserHandle = credential.Result.User.Id,
        SignCount = credential.Result.SignCount,
        AttestationFormat = credential.Result.AttestationFormat,
        RegistrationDate = DateTime.UtcNow,
        AaGuid = credential.Result.AaGuid,
        Transports = credential.Result.Transports,
        IsBackupEligible = credential.Result.IsBackupEligible,
        IsBackedUp = credential.Result.IsBackedUp,
        AttestationObject = credential.Result.AttestationObject,
        AttestationClientDataJson = credential.Result.AttestationClientDataJson,
        DevicePublicKeys = new List<byte[]> {credential.Result.DevicePublicKey}
    };

    _context.Add(newPasskey);
    await _context.SaveChangesAsync(ct);
    ```
5. Return credential to client

## How to verify a passkey

### Create settings for verifying a passkey

1. Get existing credentials for user
    ```c#
    var existingCredentials = await _context.UserPasskeys
        .Where(x => x.UserId == user.Id)
        .Select(x =>
            new PublicKeyCredentialDescriptor(PublicKeyCredentialType.PublicKey, x.Id, x.Transports))
        .ToListAsync();
    ```
2. Set verification settings
    ```c#
    var exts = new AuthenticationExtensionsClientInputs()
    {
       Extensions = false,
    };
    var uv = UserVerificationRequirement.Preferred;
    ```
3. Create assertion options
    ```c#
    var options = _fido.GetAssertionOptions(existingCredentials, uv, exts);
    ```
4. Save options in session
    ```c#
    HttpContext.Session.SetString("fido2.assertionOptions", options.ToJson());
    ```
5. Return options to client


### Verify passkey

1. Get options from session
    ```c#
    var jsonOptions = HttpContext.Session.GetString("fido2.assertionOptions");
    var options = CredentialCreateOptions.FromJson(jsonOptions);
    ```
2. Get credential from database
    ```c#
    var credential = await _context.UserPasskeys.FirstOrDefaultAsync(x => x.Id == clientResponse.Id, ct);
    ```
3. Create callback to check that the credential belongs to user
    ```c#
    IsUserHandleOwnerOfCredentialIdAsync callback = async (args, cancellationToken) =>
    {
       var storedCreds = await _context.UserPasskeys.Where(x => x.UserHandle == args.UserHandle).ToListAsync(cancellationToken);
       return storedCreds.Exists(c => c.Id.SequenceEqual(args.CredentialId));
    };
    ```
4. Verify the public key
    ```c#
    var res = await _fido.MakeAssertionAsync(clientResponse, options, credential.PublicKey,
       credential.DevicePublicKeys, storedCounter, callback, cancellationToken: ct);
    ```
5. Update the sign count and device public keys on the database credential
    ```c#
    if (res.ErrorMessage is null)
    {
       credential.SignCount = res.SignCount;
       credential.DevicePublicKeys.Add(res.DevicePublicKey);
       _context.Update(credential);
       await _context.SaveChangesAsync(ct);
    }
    ```
6. Return the result to client