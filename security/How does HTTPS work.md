# How does HTTPS work
In this newsletter, we will talk about the following:

*   How does HTTPS work?
    
*   How to store passwords safely in the database and how to validate a password?
    
*   How to learn design patterns?
    

Hypertext Transfer Protocol Secure (HTTPS) is an extension of the Hypertext Transfer Protocol (HTTP.) HTTPS transmits encrypted data using Transport Layer Security (TLS.) If the data is hijacked online, all the hijacker gets is binary code. 

[

![](assets/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F0e.jpg)


](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F0e18db0d-f511-4f85-bb58-388fce70d42e_2631x2103.png)

How is the data encrypted and decrypted?

Step 1 - The client (browser) and the server establish a TCP connection.

Step 2 - The client sends a “client hello” to the server. The message contains a set of necessary encryption algorithms (cipher suites) and the latest TLS version it can support. The server responds with a “server hello” so the browser knows whether it can support the algorithms and TLS version.

The server then sends the SSL certificate to the client. The certificate contains the public key, hostname, expiry dates, etc. The client validates the certificate. 

Step 3 - After validating the SSL certificate, the client generates a session key and encrypts it using the public key. The server receives the encrypted session key and decrypts it with the private key. 

Step 4 - Now that both the client and the server hold the same session key (symmetric encryption), the encrypted data is transmitted in a secure bi-directional channel.

Why does HTTPS switch to symmetric encryption during data transmission? There are two main reasons:

1\. Security: The asymmetric encryption goes only one way. This means that if the server tries to send the encrypted data back to the client, anyone can decrypt the data using the public key.

2\. Server resources: The asymmetric encryption adds quite a lot of mathematical overhead. It is not suitable for data transmissions in long sessions.

Over to you: how much performance overhead does HTTPS add, compared to HTTP?

Let’s take a look.

[

![](assets/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F42.jpg)


](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F42fa9ae2-5e7d-4787-b293-8e229c70d8a9_2343x2436.png)

**Things Not to do**

🔹 Storing passwords in plain text is not a good idea because anyone with internal access can see them.

🔹 Storing password hashes directly is not sufficient because it is pruned to precomputation attacks, such as rainbow tables. 

🔹 To mitigate precomputation attacks, we salt the passwords. 

**What is salt?**

According to OWASP guidelines, “a salt is a unique, randomly generated string that is added to each password as part of the hashing process”.

**How to store a password and salt?**

1️⃣ A salt is not meant to be secret and it can be stored in plain text in the database. It is used to ensure the hash result is unique to each password.

2️⃣  The password can be stored in the database using the following format: _hash(password + salt)_

**How to validate a password?**

To validate a password, it can go through the following process:

1️⃣ A client enters the password.

2️⃣ The system fetches the corresponding salt from the database.

3️⃣ The system appends the salt to the password and hashes it. Let’s call the hashed value H1.

4️⃣ The system compares H1 and H2, where H2 is the hash stored in the database. If they are the same, the password is valid. 

Over to you: what other mechanisms can we use to ensure password safety?

Besides reading a lot of well-written code, a good book guides us like a good teacher.

[

![](assets/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fc7.png)


](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fc7efd24d-0a68-4947-b987-7cf3908d7598_1999x1500.png)

**𝐇𝐞𝐚𝐝 𝐅𝐢𝐫𝐬𝐭 𝐃𝐞𝐬𝐢𝐠𝐧 𝐏𝐚𝐭𝐭𝐞𝐫𝐧𝐬**, second edition, is the one I would recommend.

When I began my journey in software engineering, I found it hard to understand the classic textbook, **𝐃𝐞𝐬𝐢𝐠𝐧 𝐏𝐚𝐭𝐭𝐞𝐫𝐧𝐬,** by the GOF. Luckily, I discovered Head First Design Patterns in the school library. This book solved a lot of puzzles for me. When I went back to the Design Patterns book, everything looked familiar and more understandable.

Last year, I bought the second edition of Head First Design Patterns and read through it. Here are a few things I like about the book:

🔹 This book solves the challenge of software’s abstract, “invisible” nature. Software is difficult to build because we cannot see its architecture; its details are embedded in the code and binary files. It is even harder to understand software design patterns because these are higher-level abstractions of the software. The book fixes this by using visualization. There are lots of diagrams, arrows, and comments on almost every page. If I do not understand the text, it’s no problem. The diagrams explain things very well.

🔹 We all have questions we are afraid to ask when we first learn a new skill. Maybe we think it’s an easy one. This book is good at tackling design patterns from the student’s point of view. It guides us by asking our questions and clearly answering them. There is a Guru in the book and there’s also a Student.

Over to you: which book helped you understand a challenging topic? Why do you like it?

Our bestselling book “System Design Interview - An Insider’s Guide” is available in both paperback and digital format.

Paperback edition: [https://geni.us/XxCd](https://geni.us/XxCd)

Digital edition: [https://bit.ly/3lg41jK](https://bit.ly/3lg41jK)

**New System Design YouTube channel**: [https://bit.ly/ByteByteGoVideos](https://bit.ly/ByteByteGoVideos)