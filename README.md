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
