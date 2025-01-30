---
layout: post
date: 2025-01-30
categories: tips, c#, security 
title: Password security checking
---


Hackers are using lists of known passwords to automate login attempts against systems. We can help users by checking if the password they are using is known to be in previous data breaches from other websites. HaveIBeenPwned has an api to do this [available here](https://haveibeenpwned.com/API/v3#PwnedPasswords).

To call the api you hash the users password and send the first part of the the hash to the api and it will respond with a list of second parts, if the second part is in the response the password has been exposed elsewhere. This makes the check secure and anonymous, the password does not leave our system.

Steps to check:

1. Intercept password during login to get the plain text password, do this after you have validated the password
2. Hash the password with the SHA1 algorithm
3. Split the hash into 2 parts, the first with 5 chars for the api call and the remainder for the check
4. Call the api with the first part of the hash
5. Check the response to see if the second part is included
6. If it is included notify the user that the password is insecure

When notifying the user make sure you state that the password was not exposed from our system and that it may not have been them that was exposed in a breach. Someone else could be using the same password.

### Example service ###

``` c#
public class PasswordPwnedService
    {
        /// <summary>
        /// Check if password is insecure
        /// </summary>
        public async Task<bool> CheckPassword(string password)
        {
            using (var httpClient = new HttpClient {BaseAddress = new Uri("https://api.pwnedpasswords.com/")})
            {
                // convert password to hash
                var hashString = HashString(password);

                // split up hash, first 5 for query, remainder for result checking
                var passStart = hashString.Substring(0, 5);
                var passEnd = hashString.Substring(5);

                // make request to HaveIBeenPwned
                var response = await httpClient.GetAsync($"range/{passStart}");

                var count = await Contains(response.Content, passEnd);
                return count > 0;
            }

        }

        /// <summary>
        /// Convert Password into SHA1 Hash
        /// </summary>
        private static string HashString(string password)
        {
            using (var sha1 = SHA1.Create())
            {
                var passBytes = Encoding.UTF8.GetBytes(password);
                // create sha1 hash
                var hashBytes = sha1.ComputeHash(passBytes);
                // convert hash to UTF8
                var sb = new StringBuilder();
                foreach (var b in hashBytes)
                {
                    var hex = b.ToString("X2");
                    sb.Append(hex);
                }

                var hashString = sb.ToString();
                return hashString;
            }
        }

        /// <summary>
        /// Finds the number of times the password is listed
        /// </summary>
        internal static async Task<long> Contains(HttpContent content, string sha1Suffix)
        {
            using (var streamReader = new StreamReader(await content.ReadAsStreamAsync()))
            {
                while (!streamReader.EndOfStream)
                {
                    var line = await streamReader.ReadLineAsync();
                    var segments = line.Split(':');
                    if (segments.Length == 2
                        && string.Equals(segments[0], sha1Suffix, StringComparison.OrdinalIgnoreCase)
                        && long.TryParse(segments[1], out var count))
                    {
                        return count;
                    }
                }
            }

            return 0;

        }
    }
```