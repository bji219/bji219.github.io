---
title:  "Vigenere Cipher"
date:   2024-04-21 15:33:33 -0500
permalink: "/VigenereCipher/"
excerpt: "62ZL.R2?VP!UTA (key: Python)"
image: assets/Images/VigenereCipher/vigenere.png
---

# Background
I read a historical fiction novel recently about the Knights Templar, and there was some reference to the [Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher). It is an unbreakable Ceasar shift which utilizes a key word so that the amount of shift changes with each letter in the coded message. I was intrigued because it involves matrices, and it would be a simple and fun coding project!

A "Ceasar shift" involves substituting each letter of the alphabet with a different letter at some specified offset. For example, "A B C" could be encoded as "B C E" (a shift of 1). A simple way to visualize this is placing the shifted alphabet over top of the original alphabet: 
```
B C E
A B C 
```
In a Vigenere cipher, the amount of "shift" changes based on a specified keyword. To create the encoded message, you use a matrix of letters (similar to the image below) with the letter of the key word determining the row index, and the letter of the message determining the column index. In my case, I added numbers 0-9 and punctuation to my matrix to make things a little more tricky! To write this out visually, the keyword is written above the message and repeated until every letter of the message is accounted for. 

The best way to describe this would be with an example- using "Python" as our key word, let's encode the message "Vigenere Cipher".
```
P y t h o n P y t h o n P y # Key word
V i g e n e r e C i p h e r # Message 
6 2 Z L . R 2 ? V P ! U T A # Encoded Message
```
 
![codematrix](/assets/Images/VigenereCipher/codematrix.png)

# Python Code
Here is an excerpt of the python code which reads the message to encode and the key word in order to scramble the input:
```python
encoded = [] # Coded message initialize
count = 0 # initialize counter
for i in msg:
    r = b.index(cp[count].capitalize()) # row index from key word
    c = b.index(i.capitalize()) # column index from the message to be encoded

    if count == c_len-1: # reset counter if it exceeds the number of chars in the key word
        count = 0
    else:
        count+=1

    # The new encoded letter
    nl = b[(r+c)%len(b)] # the alphabet 26 chars plus 14 punctuation chars. Modulo to get remainder
    encoded.append(nl)
    cm = ''.join(encoded)
```

# Decode 
I also created the decoding side of the equation, which would be a fun way to pass messages between friends:
```python
# Coded message initialize
decoded_message = []
# Get row index of each
count = 0
for i in msg:
    r = b.index(cp[count].capitalize()) # row index from code phrase
    c = b.index(i.capitalize())# row index from the coded message
    if count == c_len-1:
        count = 0
    else:
        count+=1

    # The new coded letter
    nl = b[(c-r)%len(b)] # the alphabet 26 chars plus 14 punctuation chars. Modulo to get remainder
    decoded_message.append(nl)
    dcm = ''.join(decoded_message)
```

Check out my github for the full code and more details on the project!
