<h1> From Zero Crypto to the Padding Oracle Attack </h1>

<h3>Intro </h3>
This write up has been written for the <a href=https://twitter.com/mubix/status/1196821680137678849>#OCSPGiveaway</a> contest
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

![](/images/encrypted_tux.png)

To avoid this pitfall, we use CBC mode, which basically chains blocks together by taking the output of one encryption and mixing it into the input for the next block. If you recall the Crypto function boxes from the Crypto 101 section, this is no more than playing with them as you would do it with LEGO’s block, chaining one with the following, as can be seen in the diagram below

![](/images/4.png)
<sup>[2](#myfootnote2)</sup>

<a name="myfootnote1">1</a>: This is an over simplistic view. The Padding method described here is PKCS5, and it is one of the many available. Besides, I have padded with “06” instead of the expected Hex 0x06, because I consider it easier for the reader
<a name="myfootnote2">2</a>: IV stands for Initialization vector, which is a random input added to the message used only once (usually not kept as secret). I have taken the license of using the “+” symbol to avoid mentioning that the operation is Exclusive OR, XOR.

<h3>3. Padding Oracle Attack</h3>

Finally, after understanding all the underlying concepts, we can address the attack. The first thing that our attacker needs is to capture an encrypted message (ciphertext). Then, the attacker can exploit the behavior of the intended receiver of the message in order to learn information about the ciphertext.
The encrypted message “c” captured by the attacker has been generated by a sender using the appropriate padding and an unknown key (for the attacker). So how does the attacker learn information of the ciphertext? By constructing modified ciphertexts from “c” and then send them as if they were coming from the sender. The receiver will try to decrypt the ciphertext in order to recover the data. And here comes the magic, <b>if the encoded data was not properly formatted (invalid padding), the receiver will most probably return an error. This is our Padding Oracle! </b>

<h4>How it works</h4>

Taking the model that we have been using of crypto functions, we need now to unchain the CBC described before. We want to recover Message from Block 2 (which in our example is the message of the last block)

![](/images/5.png)

![](/images/table_3.png)

We have two blocks of 8 bytes each (remember that the second block only has two bytes of actual data, the last six correspond to padding). If we encrypt only the message with AES CBC with some random key and IV we get:

![](/images/table_4.png)

As we have originated the message, we know that the last 6 bytes of the encrypted block #2 are padding. But let’s put ourselves in the attacker’s perspective.

<code>Block #1= b6a7979ebcd4d8<b>4f</b></code><br>
<code>Block #2= 3aba8c3d363bd8a8</code>

As the attacker does not know the length of the padding, he/she will send both block #1 and block #2 to the oracle, but manipulating the last byte of block #1. How? We know that after decrypting the last byte of Block #2, a8, we need to get a 06, which is the correct byte of padding.
But, there is another byte that can lead to a correct padding message: 01, because it will mean that there is only one byte of padding. 

Let’s assume that last byte of Block #2, a8, is “n” (n between 0..255). So we perform the following operation

<code>New last byte of Block #1 = <b>4f</b> + n + 01 (Last Byte of Block #1 + our_guess_of_last_byte_Block_#2 + padding) </code>

Now when the last byte of B#2 gets decrypted it will then be operated with this altered byte

</code>Last byte of Message from Block #2: Decrypt(a8) + New last byte of Block #1 </code><br>

<ul>
<li>If our assumption was not correct, we just replace “n” by one of the remaining 255 possible values.</li>
<li>If correct, the last byte of Block #2 is an “n” and the resulted last byte of Message from Block #2 will only be 01, which means that padding is correct. If that it the case, we need to move backwards and calculate the second to last byte of Block #1, and in this case, it will be done with the following expression:

<code>Block #1= b6a7979ebcd4<b>d8</b>4f</code><br>
<code>Block #2= 3aba8c3d363bd8a8</code>

<code>New second-to-last byte of Block #1 = <b>d8</b> + n + 02 ( remember that now we have 2 bytes of padding) </code> 
</li>
</ul>
The same procedure can be applied to all the bytes in the block in order to decrypt them with the help of the oracle.

<h4>How can it be detected?</h4>

What is really interesting, and unique about this attack, is that knowing if a given ciphertext produces a invalid padding is all that the attacker needs to break the security of that system. Translating this to a real world example, if a web application replies with a 200 HTTP code after receiving a valid padding and 500 if not, will do the trick.
Even if the receiver does not return an error message about improper encoding to the sender and attacker may still be able to detect it, by paying attention at the response time or by analyzing the Behavior of the receiver. If the receiver terminates the connection, that is indicative of an error.

<h4>How can it be prevented</h4>
<ul>
<li>By catching the exception or using generic error messages instead of invalid padding message</li>
  <li>Rate-limiting requests from the same IP address</li>
  <li>Monitoring for suspicious requests</li>
</ul>
