---
layout: post
title: "Sending secrets in the open"
date: 2015-02-18 11:07:36 -0700
comments: true
categories:
published: true
---
Steganography is a tool which can allow a person to send a secret message, without the adversary detecting that a message has been sent at all. Steganography has a long, rich history that you can read about on [the Information Hiding Homepage](http://www.petitcolas.net/steganography/history.html). My personal favorite, and one that lends itself to practical application, is the first example on that page, the Hypnerotomachia Poliphili of 1499 by Anonymous:

> The Hypnerotomachia Poliphili (translated as ‘the strife of love in a dream’) is a very puzzling and enigmatic book. Published by Aldus Manutius in 1499, it contains a vast knowledge of architecture and landscape and garden design, but also engineering, painting and sculpture. It also contains one of the most famous authorship steganogram: the first letter of the 38 chapters spelled out ‘Poliam frater Franciscus Columna peramavit’ (‘Brother Francesco Colonna passionately loves Polia’). Colonna was a monk, still alive when the book was published...

Apart from calling out a colleagues' trespasses, while a noble endeavour, a user of steganography could also send sensitive messages, where the fact that a message is sent (even if the message is not decoded) provides intelligence value. Furthermore, in the age of Edward Snowden and NSA espionage on American citizens, it is an exercise of your right to privacy to send secret messages to one another even about mundane things like asking your wife what is for dinner. Of course, there are practical limitations to this in that the worth of the message should exceed the cost of the effort to send and receive it.

This tutorial will discuss how to go about hiding secret messages in JPEG/JFIF images, but the principles used to do this can be extrapolated to any number of methods of implementing steganography. It is hoped that by reading and following along, this tutorial will serve as the flint to spark the fire of imagination in this respect.

------------
Requirements
------------
To follow this tutorial, the following items are required:

- A JPEG image in JFIF format (we'll discuss how to tell your JPEG fits this format). You can use the following image which I will be using, which comes from the [Wikipedia Commons](http://commons.wikimedia.org/wiki/Main_Page). ![Tutorial Image](../images/India_Landscape.jpg)
- A hex editor. I'm using [Hex Fiend](http://ridiculousfish.com/hexfiend/) and the screenshots in this tutorial will reflect that, but any hex editor that works on your operating system should suffice. [Wikipedia](http://en.wikipedia.org/wiki/Comparison_of_hex_editors) provides an excellent table of hex editors and their respective features. All that you require is the ability to alter the values of the Hexadecimal, and save the file when you are done. A nice feature to look for is the ability to use cut/paste, and display the ascii representation of the hexadecimal values.
- An encryption scheme. I'm using symmetric key encryption via [triplesec](https://keybase.io/triplesec/), but you may use a more or less secure encryption scheme to suit your purposes.

Terms
-----
The following terms are used liberally throughout this tutorial:
- adversary - the entity you do not wish to receive your message, or possibly even know that you have sent a message
- encoded image - the result of applying steganography to a base image, which contains your secret message

-------------------
Basic Steganography
-------------------
To begin, we will perform the most basic of steganography. We're just going to stuff the desired message in plaintext in the image stream and see what happens. The first thing that needs to be done is to open the image in your hex editor. If you use the beach scene I am using, you will notice some EXIF data near the beginning of the file (shown at the right of Figure 1). This can be useful if you are trying to gather information on identifying the photographer, but that can be covered in its own separate post. You'll also want to bring up your editors find functionality and point it to FFD9, which signifies the end of the stream. If the image has more than one FFD9, someone else has already tampered with the image.

->![Figure 1](../images/hexfiend_1.png)<-

->Figure 1: Opening the image in hex fiend<-

If all is well with your image, you should be at the bottom of the file, with the last two bytes being FFD9, and this should be the only occurrence of FFD9 in the file. The most basic way to hide a message in the file is to simply type it in the text side, making sure to leave the ˇŸ(FFD9) intact, as shown in figure 2. In this case, the message is "Hello. This is dog."

->![Figure 2](../images/hexfiend_2.png)<-

->Figure 2: Adding our message

Below, in Figure 3, are the initial image and the resulting image side by side. You should not see any major difference between the two images, or our objective has failed. Indeed, there is no immediately obvious difference, because our message is hiding outside the size of the image (defined in the image header). Unfortunately, it is fairly easy to calculate the expected size of the image based on the information in the header. Additionally, if the adversary has the original image, they can see that the size is changed (we added 19 bytes, at 1 byte per character and 19 characters). Finally, the adversary could also run the binary of the file through a word scanner and would get a hit with Hello, This, is, and dog.

->![Figure 3](../images/Basic_Compare.png)<-

->Figure 3: Original and encoded images<-

Getting Warmer
--------------
So we found in the previous section that we could in fact hide a message in an image without altering the image, but there were several drawbacks. One of these drawbacks, the file size difference, can be overcome by simply replacing the number of bytes we wish to include with the message, instead of appending the bytes to the end. Additionally, the adversary may be only scanning near the end to find words, or manually looking at the end of the file. Therefore, we can move the message higher up in the image (care must be taken to not include it too high, or the exif data will be corrupted). The following figure shows the side by side comparison of the result of removing our original message and replacing 19 bytes a little higher up in the file with the message. Notice that there is now an obvious artifact in the image which can clue the adversary in to the fact that something is up. Of course, they may just chalk it up to cruddy compression, if we're lucky.

->![Figure 4](../images/Warm_Compare.png)<-

->Figure 4: Original and encoded images<-

While following along with this exercise, you may have noticed that you end up corrupting much more data than the image I have shown, probably by creating a solid white line at the bottom of the image. It took several attempts to find a suitable location for the message that only made the corruption shown above (and no one wants to manually search through 88,000+ bytes/characters for the plaintext message). Some images are better than others, and you will notice that this image has a border around it, that was likely added after the fact, making this process much trickier than it necessarily has to be. Additionally, the words are still in plaintext for anyone to see. Fortunately, there are additional improvements we can make to the process to avoid corrupting data noticeably.

Bits and Bytes
--------------
So far we've been encoding our message in plaintext as one byte per character, ensuring that the filesize does not change, and corrupting the data significantly enough for it to be obvious to the human eye. How do we encode our message without the plaintext being obvious, without using more space, and without making obvious corruptions to the image? The answer lies within the fact that 8 bits (one byte) is all that is required to represent each character. These 8 bits do not have to be contiguous. Since JPEGs are combinations of 8-bit red green blue values per pixel, 3 bits per pixel can be altered. Therefore, 3 pixels provides 9 bits of useable space that won't yield a discernable difference if altered, which is enough to hide a character in. For example, suppose you have the bytes FF FF A1 B2 C3 4F D2 AB CD, which is 11111111 11111111 10100001 10110010 11000011 01001111 11010010 10101011 11001101 in binary. Suppose also you wish to encode the single character 'H', which in ascii is hex 48, or binary 01001000. We can then modify the binary sequence for our 9 bytes to be 11111110 11111111 10100000 10110010 11000011 01001110 11010010 10101010 11001101. Notice that we only changed 4 of the 9 bytes to encode our message. This will alter four of the 9 red green and blue values by only 1/255 of their range, which should be indiscernible to the human eye. It does require much more work, though, which is why several programs have been written to do this work automatically, but a thorough discussion of these programs is outside the scope of this blog post.

-------------------
Detection Avoidance
-------------------
###Amateur Photography

For each of these examples, I am using a JFIF image I found on the internet. Suppose my adversary were able to intercept my encoded image, and was also able to find the original on the internet, presumably where I found it. In this case, the adversary would be able to diff the two files and determine that there was a change to the data portion of the file, even if they might not be able to decrypt the message or read it. This is why it is often better to utilize one's own pictures, of which there is only one secure copy. The secure copy is never sent to the recipient, and the encoded image is the one which is sent out. Then, the adversary has no frame of reference to tell if the file has been modified from the original.

###Discretion is the Better Part of Valour

This is a reminder that subtlety can only work in your favor. Don't lump your whole message in one central location. Take a lesson from the anonymous monk who called out his fellow Colonna, and spread it out over a few areas. Obviously, you will still need some pre-defined scheme that your recipient understands (say, one bit encoded at the end of the first of every tenth byte), so that they can correctly extract the message, but nothing is more obvious than opening a JPEG as hex with ascii and seeing "Hello, this is dog" right before the final ˇŸ(FFD9).

###The Writing is on the Wall

Establish a pattern and stick to it. If you routinely post photos to facebook, continue to do so and use that to send your encoded images to the right recipients. Things only look suspicious if you deviate from the routine you've established. The best way to establish a routine for sending encoded messages is to get a hobby that involves pictures. DIY hobbies, birdwatching, whatever it is will give you the excuse to post images, and most people will not suspect at all that those images have been tampered with.

----------
Disclaimer
----------
I am in no way advocating any illegal activity. Any use of the techniques described in this tutorial that contravene the laws that you are subject to is entirely your own responsibility, and at your own risk.
