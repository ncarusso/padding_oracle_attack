<h1> From zero crypto to padding oracle attack </h1>

<h3>Intro </h3>
This write up has been written for the #OCSPGiveaway contest <link>https://twitter.com/mubix/status/1196821680137678849</link> 
and it is a humble attempt to explain the Padding oracle attack to an audience with no previous exposure to crypto.
In order to explain a crypto attack to someone who doesn’t know crypto, I have come up with the following idea:
I will define some key concepts in a very clear and simplified way, by using the universal concept of function, that we know from high school.

This work has been divided in 3 main parts:
<ol>
<li>Crypto 101. The foundations to understand crypto</li>
<li>More Crypto, Cipher Block Chaining (CBC) and Padding</li>
<li>Padding Oracle Attack</li>
</ol>

<h3>1. Crypto 101 </h3>

Suppose that you want to send a message to a friend. Moreover, you want to send the message securely: you want to make sure an eavesdropper cannot read it. Here is where <b>Cryptography</b> plays its role, because it is the process of transforming a readable message, that we will call from now on <b>Plaintext</b>, into a secret message, that we will call <b>Ciphertext</b>.

A <b>function</b> is a process that given a certain input, it outputs a value. For instance, given the function that adds 1 to a number, we have 

![](/images/1.png)

In the same way, in cryptography we have <b>crypto functions</b>.

As it can be seen in the diagram below, the sender feeds a message into the crypto function and he receives the secure version of the message. This Process is known as <b>Encryption</b>.

![](/images/2.png)

Then, the receiver takes the secret message and recovers the readable message by feeding it into the crypto function. This process is known as <b>Decryption</b>.

![](/images/3.png)

This <b>CRYPTO function</b> that we have defined to refer to both the encryption and decryption process is known as the <b>cipher</b>.

The complete process can be summarized as follows

![](/images/enc1.png)

So far, so good, but have you realized that if I know how the CRYPTO function works, I will be able to uncover the message that you have just sent to your friend? You don’t want me to do that but, how do you prevent it? By using a key, which is a secret value that both encryption and decryption operations use. 

Now, if we add the <b>key</b> to the scheme above we have

![](/images/enc2.png)


<h3>2. More Crypto, Cipher Block Chaining (CBC) and Padding </h3>

Now that you have the foundations of cryptography we can move to some other topics that will help you understand the padding oracle attack.

The CRYPTO functions (or ciphers) that we have seen in crypto 101 are usually grouped into two categories: stream and block ciphers. Stream ciphers encrypt data on a bit-by-bit basis. Plaintext and ciphertext are always the same length. Unlike stream ciphers, which can encrypt data of any size, block ciphers can only encrypt data in "blocks" of a fixed size. So, what happens when the message that you want to send is shorter than the given block size (usually 8 or 16 bytes)? Well, that’s when the concept of <b>Padding</b> is needed. Follow me in this example
You want to send the message <i>PWK Voucher</i> and the block size is 8 bytes. As the message is 10 bytes long (1 byte per character) you need two blocks. Let’s put the message in the following table

![](/images/table_1.png)

We have 10 boxes filled and the last 6 that are empty. What we do now is to complete every empty box with the number of empty boxes that we have<sup>[1](#myfootnote1)</sup>. In our case, 6. The table will end up like this

![](/images/table_2.png)

So that’s why we use Padding: to ensure that the message exactly fits one or multiple blocks.

Now that we have our blocks completed, we can proceed and encrypt them. The trivial mode to accomplish this is to take every block of the message and encrypt it with a <b>key</b>. But this mode, called ECB, is a bad way to encrypt most kinds of data. Why? Because if two blocks have the same value they will be encrypted to the same value.
Initially you would think, what is the problem with that? Well if the data that you are protecting has redundant portions, let’s say an image which has many pixels of the same color, you end up with an encrypted image from which you can uncover the original one. 

Look at this example. The image at the left is the message (plaintext) and the image at the right is the result of encrypting it using a trivial mode 

![](https://static.lwn.net/images/2018/kr-donenfeld-tux.jpg)


<a name="myfootnote1">1</a>: This is an over simplistic view. The Padding method described here is PKCS5, and it is one of the many available. Besides, I have padded with “06” instead of the expected Hex 0x06, because I consider it easier for the reader
