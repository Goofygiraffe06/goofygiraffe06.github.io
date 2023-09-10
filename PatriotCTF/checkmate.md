---
layout: post
title:  "Checkmate [Medium]"
date:   2023-09-10
categories: CTF Writeups
---


---

# Checkmate [Medium]

----
## Challenge Description 

Get your adrenaline pumping as you navigate the thrilling world of Crypto Web in Capture the Flag.

Minor brute force is allowed, but you shouldn't need over 4000 tries.

Flag format: PCTF{}

Author: @sau_12
http://chal.pctf.competitivecyber.club:9096/

----

## Finding the Correct Name

The first thing we see is a regular login page with two input fields: name and password, accompanied by a button labeled "Hi :)". When you enter typical credentials, such as "admin" for both fields, it fails. The "wrong username" error might provide a scope to enumerate users. Looking at the source code using `ctrl+u`, it becomes evident that there is some logic we need to reverse engineer to get the correct name and password. Let's focus on the name for now:

```
function checkName(name){

    var check  = name.split("").reverse().join("");
    return check === "uyjnimda" ? true : false;
}
```

This code snippet evaluates our input. To understand it:

The `checkName(name)` function takes the name as its parameter. It splits the string, effectively converting it into an array, then reverses the array and joins it. This essentially reverses the entered name and checks if it equals `uyjnimda`. Reversing this ourselves, the output should be `adminjyu`, which is the correct username. YAY!

## Reverse Engineering the Password

Let's repeat the procedure to find the correct username. First, let's identify the code snippet responsible for authentication, understand it, and then crack it.

```
function checkLength(pwd){
     return (password.length % 6 === 0)? true : false;
}
function checkValidity(password){
    const arr = Array.from(password).map(ok);
    function ok(e){
        if (e.charCodeAt(0) <= 122 && e.charCodeAt(0) >= 97){
        return e.charCodeAt(0);
    }}

    let sum = 0;
    for (let i = 0; i < arr.length; i += 6){
        var add = arr[i] & arr[i + 2];
        var or = arr[i + 1] | arr[i + 4];
        var xor = arr[i + 3] ^ arr[i + 5];
        if (add === 0x60 && or === 0x61 && xor === 0x6) sum += add + or - xor;
    }
   return sum === 0xbb ? true : false;
}
```

At first glance, this might seem overwhelming, but it becomes clearer once we break it down. Two functions play a part here. The `checkLength(pwd)` function takes the password as its parameter and checks if the length of the entered string, when divided, has a remainder equal to zero. So, we can infer that the password should be 6 characters long.

The `checkValidity(password)` function is similar. It takes the user's entry from the login page and processes it using a series of binary operations on each character of the string. The line `const arr = Array.from(Password).map(ok)` transforms the password into an array and maps it using the `ok` function.

The `ok(e)` function checks the Unicode or ASCII code for each character. If it's lowercase (i.e., if the ASCII Value is between 97 and 122, inclusively), it returns the value; otherwise, it returns undefined.

The core logic involves a loop that starts at index 0 of the array. As long as `i` is less than the array's length, it continues iterating. With each iteration, the value of `i` increases by 6. 

Three variables are initialized in the loop: `add`, `or`, and `xor`, which perform bitwise operations on the characters at specific positions. 

The line `if (add === 0x60 && or === 0x61 && xor === 0x6) sum += add + or - xor;` checks if the variables match certain criteria, and then updates the value of `sum`. Eventually, the sum is checked against 0xbb (187 in decimal).

Given all these steps, multiple strings could satisfy these criteria, prompting the need to build a brute-forcer in Python.

## Building a Bruteforcer 

```python
def generate_passwords():
    passwords = []   # A list to store potential passwords 
    
    # Iterating through lowercase characters (Unicode code between 97 and 123)
    for a in range(97, 123):     
        for b in range(97, 123):
            for c in range(97, 123):
                for d in range(97, 123):
                    for e in range(97, 123):
                        for f in range(97, 123):
                            # Check conditions
                            if a & c == 0x60 and b | e == 0x61 and d ^ f == 0x6:
                                # Construct the potential password
                                password = chr(a) + chr(b) + chr(c) + chr(d) + chr(e) + chr(f)
                                passwords.append(password)

    return passwords

valid_passwords = generate_passwords()

for pwd in valid_passwords:
    print(pwd)
```

Save the results into a file using the `tee` command, like so: `python3 brute.py | tee pass.txt`. Since outputting the password after finding it might slow down the brute-forcer, the script simply adds them to the list and prints them after processing all possible combinations. The entire process takes around 2-3 minutes, although this might vary based on your CPU.

Upon navigating to `/check.php`, we encounter password verification again. Time to brute-force using the passwords from our previous script.

## Building Bruteforcer 2

The correct password needs to be posted to /check.php. If an incorrect password is entered, the website outputs "incorrect password", which can be used to find the correct flag. Recall saving your passwords in a file? It's time to put them to use.

```python
import requests
import threading

# Function to check the password
def check_password(password):
    data = {'password': password}
    response = requests.post(url, data=data)

    # Check if the response contains "incorrect password"
    if "incorrect password" not in response.text.lower():
        print(f"Correct password: {password}")

# Read passwords from the file
with open('pass.txt', 'r') as f:
    passwords = [line.strip() for line in f.readlines()]

url = "http://chal.pctf.competitivecyber.club:9096/check.php"
threads = []

# Create and start a new thread for each password
for password in passwords:
    thread = threading.Thread(target=check_password, args=(password,))
    threads.append(thread)
    thread.start()

    if len(threads) >= 10:  # Multithreading as the list is long and the CTF is time-bound.
        for thread in threads:
            thread.join()
        threads = []

# Wait for all threads to finish
for thread in threads:
    thread.join()
```

After some time, the script will identify the correct

 password. Entering it on the website will finally unveil the flag!

WHOA! THIS WAS A REALLY GOOD CHALLENGE!
