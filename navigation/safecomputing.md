---
layout: post
title: Safe Computing
description: Team Teach on Safe Computing
type: issues
permalink: /safecomputing
comments: true
---

# Big Idea 5: Safe Computing

## What is PII(Personally Identifiable Information)

- Information identifying a user on the Internet
- Safe computing revolves around this PII and how it is exploited or kept secure
- For example:- Social Security Number, email, full name, driver's license etc.

### Cookies

- A variety of different cookies are used in different sites
    - **Session Cookies**: Temporary cookies that are deleted when the browser is closed.  
    - **Persistent Cookies**: Stored on a user's device for a specified duration, even after the browser is closed.  
    - **First-Party Cookies**: Set by the website being visited, mainly for user experience and analytics.  
    - **Third-Party Cookies**: Placed by external domains (e.g., advertisers) to track user behavior across sites.
- These cookies track your PII, track your history on the site, and provide recommendations based on that history
- Can breach security of the user because it takes personal info and preferences and displays that on the site

## Popcorn Hack 1: Cookies

- Open Developer Tools(fn + F12 -> Application -> Cookies)
- Find cookie from one site and find:
    - Name, Value, Expiration Date
    - Where the cookie came from(See if you can figure out which category that is)
    
![Cookies Hack](cookieshack.png)

## Password Security

- As seen more and more on the web, passwords are required to be more and more sophisticated for the users safety. Modern expectations for a password include:
    - Minimum **10 characters** 
    - Use of both **uppercase** and **lowercase** lettering
    - Contains at least **one number**  
    - Contains at least **one special character**  


## Encryption
<div style="display: inline-block; margin-right: 10px;">
    <img src="images/encryption.png" alt="Encryption" width="450"/>
</div>
<div style="display: inline-block;">
    <img src="images/emoji.png" alt="yay" width="150"/>
</div>




- Encryption is the process of converting data into a coded format to prevent unauthorized access. It ensures only authorized user can read the information. 
- Types of Encryption
1. **Symmetric Encryption (Private Key Encryption)**
    - The same key is used for both encryption and decryption
    - Faster but requires securely sharing the key between parties
    - Example: AES (Advance Encryption Standard) used for securing files
2. **Asymmetric Encryption (Public Key Encryption)**
    - Uses a pair of keys:
        - Public Key (for encryption)
        - Private key (for decryption)
    - More secure for internet communications since users don't need to share a single key. 
3. **Hashing (One-way encryption)**
    - Converts data into a fixed length string that cannot be reversed
    - Used for storing passwordss securely. 
    - Example: SHA-256 (Secure Hash Algorithm) used in blockchain and password security. 

### Where Encryption is Used:
- **Web Security**: HTTPS (SSL/TLS encryption) protects websitess from eavesdropping
- **Messaging Apps**: End-to-end encryption ensures that messages are private
For a more detailed explanation, here is a short video:-

<div style="display: inline-block; margin-right: 10px;">
    <a href="https://www.youtube.com/watch?v=9chKCUQ8_VQ">
        <img src="https://img.youtube.com/vi/9chKCUQ8_VQ/0.jpg" alt="Video Title" />
    </a>
</div>
<div style="display: inline-block;">
    <img src="images/question.png" alt="?" width="150"/>
</div>

## Phishing
- Phishing is a cyber-attack where scammers trick you into giving personal information (such as bank details or passwords) by pretending to be a legal entity, usually through fake emails, messages, or websites. 
<br>
People can be tricked into giving information through many different kinds of phishing attacks and it is essential for us to understand how to prevent them. 

### Email Phishing 
Attackers send fake emails pretending to be trusted companies (for example, Amazon, Banks etc.)
To prevent this an individual must
- Prevent clicking on links or downloading attachments from unexpected emails. 
- check the sender's email carefully because scammers often use weird looking domains like the one shown below
<p></p>
<div style="text-align: center;">
    <div style="display: inline-block;">
        <img src="navigation/yesyes.png" alt="?" width="450"/>
    </div>
</div>

### Website Spoofing
Attackers can also create fake versions of popular websites to steal login credentials. Some popularly "spoofed" websites include Google, Amazon and various banking sites
- To Prevent this, we must ensure that
    - We type out the webite URL manually instead if clicking links in emails or messages because we prevent a misdirection into a malicious link when we type it out. 
    - Look for HTTPS in front of the URL and a padlock icon in the browser bar, most spoofed sites are insecure and lack security. (However some scamming sites may be secure)
    - If unsure of the site, then contact customer services for said organization. 

Here is an example
<div style="text-align: center;">
    <div style="display: inline-block;">
        <img src="navigation/safe.png" alt="?" width="700"/>
    </div>
</div>

### Smishing (SMS Phishing)
Hackers also send fake text images pretending to be from banks, delivery services, or government agencies, urging you to click a malicious link.
- To prevent this, we must 
    - never click on links in texts from unknown numbers
    - never contact our bank or service provider directly if we recieve a suspicious text
    - and enable spam filters on our phones to block spam messages
<p></p>
<div style="text-align: center;">
    <div style="display: inline-block;">
        <img src="navigation/phone.png" alt="?" width="300"/>
    </div>
</div>



## Verification

- Verification is a crucial aspect of safe computing, ensuring that users, systems, and software are legitimate and secure.
- Uses and types of verification
1. Multi-Factor Authentification (MFA)
    - Added security process that requires users to verify their identity using multiple independent authentication factors.
    - Strengthens security by preventing unauthorized access even if passwords are stolen
    
2. Digital Structures
    - A digital signature is a cryptographic technique used to verify the authenticity and integrity of digital documents, messages, and software
    - Sender would generate a hash, the recipient would decrpt the hash and confirm it matches the original

3. CAPTCHA (Completely Automated Public Turing test to tell Computers and Humans Apart)
    - A common popup that ensure the user is not a bot
    - Most CAPTCHA are simple puzzles or questions that would be easy for humans but an AI should have a hard time figuring it out

## Popcorn Hack 2: CAPTCHA

<img src="{{site.baseurl}}/images/captcha.png" alt="CAPTCHA" width="350">

## Homework Hack 1

<div style="margin-top: 20px;">
    <a href="https://forms.gle/XyRK8JhrutuHfEQRA" target="_blank" style="
        display: inline-block;
        background-color: #4CAF50;
        color: white;
        text-align: center;
        text-decoration: none;
        padding: 10px 20px;
        font-size: 16px;
        border-radius: 5px;
        cursor: pointer;
        transition: background-color 0.3s ease;
    " onmouseover="this.style.backgroundColor='#45a049'" onmouseout="this.style.backgroundColor='#4CAF50'">
        MCQ
    </a>
</div>

## Homework Hack 2
Here is a simple code for encrypting characters or words (run in notebook to show output)
```python
def caesar_cipher(text, shift, mode):
    result = ""
    for char in text:
        if char.isalpha():  # This part of the code only ecnrypts letters
            shift_amount = shift if mode == "encrypt" else -shift
            new_char = chr(((ord(char.lower()) - 97 + shift_amount) % 26) + 97)
            result += new_char.upper() if char.isupper() else new_char
        else:
            result += char  # this keeps the spaces and position unchanged
    return result

# Here is the code for getting the user input
mode = input("Do you want to encrypt or decrypt? ").strip().lower()
message = input("Enter your message: ")
shift = int(input("Enter shift value (number of places to shift): "))

# And finally the code performs the encryption/decryption
output = caesar_cipher(message, shift, mode)
print(f"Result: {output}")
```
- Edit this code in such a way that the code is able to take "random" as a shift value. Which means that it picks a random integer between 1 and 25. 
- Make full use of Jupyter notebooks and play around witht the outputs until you get your desired output. 
### Answer

<button onclick="toggleCode()" style="
  background-color: #4CAF50; 
  color: white; 
  border: none; 
  padding: 12px 24px; 
  text-align: center; 
  text-decoration: none; 
  display: inline-block; 
  font-size: 16px; 
  border-radius: 8px; 
  cursor: pointer; 
  transition: background-color 0.3s, transform 0.2s;
">
  Click to Show/Hide Code
</button>

<div id="codeBlock" style="display:none; white-space: pre-wrap; font-family: 'Courier New', monospace; background-color:rgb(0, 0, 0); padding: 10px; border: 1px solid #ddd; border-radius: 5px; margin-top: 15px;">
  <pre>
<code>
import random

def caesar_cipher(text, shift, mode):
    result = ""
    for char in text:
        if char.isalpha():  # Encrypts only letters
            shift_amount = shift if mode == "encrypt" else -shift
            new_char = chr(((ord(char.lower()) - 97 + shift_amount) % 26) + 97)
            result += new_char.upper() if char.isupper() else new_char
        else:
            result += char  # Keeps spaces and punctuation unchanged
    return result

# Get user input
mode = input("Do you want to encrypt or decrypt? ").strip().lower()
message = input("Enter your message: ")
shift_input = input("Enter shift value (or type 'random'): ").strip().lower()

# Generate a random shift if "random" is entered
shift = random.randint(1, 25) if shift_input == "random" else int(shift_input)

print(f"Using shift value: {shift}")  # Display the shift used
output = caesar_cipher(message, shift, mode)
print(f"Result: {output}")
</code>
  </pre>
</div>

<script>
  function toggleCode() {
    var codeBlock = document.getElementById("codeBlock");
    if (codeBlock.style.display === "none") {
      codeBlock.style.display = "block";
    } else {
      codeBlock.style.display = "none";
    }
  }
</script>