---
layout: post
title: "Sending secrets in the open"
date: 2015-02-18 11:07:36 -0700
comments: true
categories:
---
Steganography is a tool which can allow a person to send a secret message, without the adversary detecting that a message has been sent at all. Steganography has a long, rich history that you can read about on [the Information Hiding Homepage](http://www.petitcolas.net/steganography/history.html). My personal favorite, and one that lends itself to practical application, is the first example on that page, the Hypnerotomachia Poliphili of 1499 by Anonymous:

> The Hypnerotomachia Poliphili (translated as ‘the strife of love in a dream’) is a very puzzling and enigmatic book. Published by Aldus Manutius in 1499, it contains a vast knowledge of architecture and landscape and garden design, but also engineering, painting and sculpture. It also contains one of the most famous authorship steganogram: the first letter of the 38 chapters spelled out ‘Poliam frater Franciscus Columna peramavit’ (‘Brother Francesco Colonna passionately loves Polia’). Colonna was a monk, still alive when the book was published...

Apart from calling out a colleagues' trespasses, while a noble endeavour, a user of steganography could also send sensitive messages, where the fact that a message is sent (even if the message is not decoded) provides intelligence value. Furthermore, in the age of Edward Snowden and NSA espionage on American citizens, it is an exercise of your right to privacy to send secret messages to one another even about mundane things like asking your wife what is for dinner. Of course, there are practical limitations to this in that the worth of the message should exceed the cost of the effort to send and receive it.

This tutorial will discuss how to go about hiding secret messages in JPEG/JFIF images, but the principles used to do this can be extrapolated to any number of methods of implementing steganography. It is hoped that by reading and following along, this tutorial will serve as the flint to spark the fire of imagination in this respect.

------------
Requirements
------------
To follow this tutorial, the following items are required:

- A JPEG image in JFIF format (we'll discuss how to tell your JPEG fits this format). You can use the following image which I will be using. ![Tutorial Image](../images/bird_32_gray.png)
- A hex editor. I'm using [Hex Fiend](http://ridiculousfish.com/hexfiend/) and the screenshots in this tutorial will reflect that, but any hex editor that works on your operating system should suffice. [Wikipedia](http://en.wikipedia.org/wiki/Comparison_of_hex_editors) provides an excellent table of hex editors and their respective features. All that you require is the ability to alter the values of the Hexadecimal, and save the file when you are done. A nice feature to look for is the ability to use cut/paste, and display the ascii representation of the hexadecimal values.
- An encryption scheme. I'm using symmetric key encryption via [triplesec](https://keybase.io/triplesec/), but you may use a more or less secure encryption scheme to suit your purposes.

Terms
-----
The following terms are used liberally throughout this tutorial:
- adversary - the entity you do not wish to receive your message, or possibly even know that you have sent a message
- encoded image - the result of applying steganography to a base image, which contains your secret message

------------
Basic Steganography
-------------------
To begin, we will perform the most basic of steganography.

-------------------
Detection Avoidance
-------------------
###Amateur Photography

For each of these examples, I am using a JFIF image I found on the internet. Suppose my adversary were able to intercept my encoded image, and was also able to find the original on the internet, presumably where I found it. In this case, the adversary would be able to diff the two files and determine that there was a change to the data portion of the file, even if they might not be able to decrypt the message or read it. This is why it is often better to utilize one's own pictures, of which there is only one secure copy. The secure copy is never sent to the recipient, and the encoded image is the one which is sent out. Then, the adversary has no frame of reference to tell if the file has been modified from the original.

###Discretion is the Better Part of Valour

This is a reminder that subtlety can only work in your favor. Don't lump your whole message in one central location. Take a lesson from the anonymous monk who called out his fellow Colonna, and spread it out over a few areas. Obviously, you will still need some pre-defined scheme that your recipient understands, so that they can correctly extract the message, but nothing is more obvious than opening a JPEG as hex with ascii and seeing "Hello, this is dog" right before the final ˇŸ(FFD9).

###The Writing is on the Wall

Establish a pattern and stick to it. If you routinely post photos to facebook, continue to do so and use that to send your encoded images to the right recipients. Things only look suspicious if you deviate from the routine you've established. The best way to establish a routine for sending encoded messages is to get a hobby that involves pictures. DIY hobbies, birdwatching, whatever it is will give you the excuse to post images, and most people will not suspect at all that those images have been tampered with.

----------
Disclaimer
----------
I am in no way advocating any illegal activity. Any use of the techniques described in this tutorial that contravene the laws that you are subject to is entirely your own responsibility, and at your own risk.
