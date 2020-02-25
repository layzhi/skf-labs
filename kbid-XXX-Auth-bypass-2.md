# KBID xxx - Auth-bypass-2

## Running the app

```text
$ sudo docker pull blabla1337/owasp-skf-lab:auth-bypass-2
```

```text
$ sudo docker run -ti -p 127.0.0.1:5000:5000 blabla1337/owasp-skf-lab:auth-bypass-2
```

{% hint style="success" %}
Now that the app is running let's go hacking!
{% endhint %}

![Docker image and write-up thanks to Contrahack.io !](.gitbook/assets/screen-shot-2019-03-04-at-21.33.32.png)

## Reconnaissance

While most applications require authentication to gain access to private information or to execute tasks, not every authentication method is able to provide adequate security. Negligence, ignorance, or simple understatement of security threats often result in authentication schemes that can be bypassed by simply skipping the log in page and directly calling an internal page that is supposed to be accessed only after authentication has been performed.

In addition, it is often possible to bypass authentication measures by tampering with requests and tricking the application into thinking that the user is already authenticated. This can be accomplished either by modifying the given URL parameter, by manipulating the form, or by counterfeiting sessions.

Obviously, an attacker can tamper with the URL, the form or the session cookie in order to get logged in as a user without knowing the actual credentials.

The goal of this lab is to get logged in as an administrator without knowing his/her credentials

Lets start the application and register a new user

![](.gitbook/assets/auth-2-register1.png)

![](.gitbook/assets/auth-2-register2.png)

Please note that (for convenience) your password will be reset if the user already exists.
Also note that the username and password are case sensitive.

![](.gitbook/assets/auth-2-register3.png)


Now that we have valid credentials, we can login:

![](.gitbook/assets/auth-2-login.png)

After providing the correct credentials we're logged in:

![](.gitbook/assets/auth-2-loggedin.png)


## Exploitation
We can capture the login in the burpsuite proxy and send it to the repeater. We notice that with every login, the session cookie stays the same. It is high likely that this sessionid is related to our user name:

![](.gitbook/assets/auth-2-repeater.png)

If we quickly google for this sessionid, we find nothing:

![](.gitbook/assets/auth-2-google.png)

We can check whether it is a hash:

![](.gitbook/assets/auth-2-sha1.png)

it seems to be a sha1...

It is possible that the developer added a salt to the username and hashed the concatenated string
admin+some_salt 
-> maybe this is also the reason why we can't find with Google what the hash represents.

The about page seem to contain a lot of text, maybe the salt is a typical word for this company that is also mentioned on that page…

Using cewel we can grab all the words from a page like this:
cewl -m 4 -w wordlist.txt -d 0 -v http://127.0.0.1:5000/about</br>
<I>-m 4: minimum word length is 4 characters</br>
-w wordlist: write output to file ‘wordlist’</br>
-d 0: follow links x times deep (0=stay on the same page)</br>
-v: verbose (show what you are doing)</br></I>

Using a terminal window:

![](.gitbook/assets/auth-2-cewl.png)

![](.gitbook/assets/auth-2-wordlist.png)

Let’s use burp intruder to calculate a sha-1 for every admin+word combination:

![](.gitbook/assets/auth-2-intruder1.png)

Payload position:

![](.gitbook/assets/auth-2-intruder2.png)

Paste the content of the word list in the payload options and add the payload processing rules as indicated in the following screenshot.

![](.gitbook/assets/auth-2-intruder3.png)

This will prefix the word 'admin' to each word from the list and calculate a sha1 of the concatenated string.
for example sha1(adminBank)

Start the attack

![](.gitbook/assets/auth-2-intruder4.png)

The result:

![](.gitbook/assets/auth-2-intruder5.png)

Now we can replace our cookie/sessionID with the value we found.

![](.gitbook/assets/auth-2-cookie1.png)

After refreshing the screen we're logged in as admin !

![](.gitbook/assets/auth-2-admin.png)


## Additional sources

{% embed url="https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html" caption="" %}
