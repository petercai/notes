# 8 Data Structures That Power Your Databases
This week’s system design refresher:

*   Importance of system design interview (Youtube video)
    
*   Data Structures That Power Your Databases
    
*   Git Merge vs. Git Rebase
    
*   Buy Now, Pay Later (BNPL)
    
*   Code complexity vs. Experience
    

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd7df9dfc-9938-4110-9ef3-c7615602f05f_1600x840.png)


](http://bit.ly/3wCvSR4)

Onboarding new engineers can take months - especially if they struggle to wrap their heads around your complex microservice architecture. We help teams at Cisco, IBM, BMW, and McKinsey communicate their system architectures using C4 model diagramming.

If you work with microservice architectures, we’d love to offer your team an exclusive free training session covering:

*   Common diagramming mistakes
    
*   Modeling vs diagramming
    
*   Introduction to the C4 model
    
*   C1 - defining your software systems
    
*   C2 - diagramming your microservices
    
*   Visualizing your event-driven architecture
    

[Learn more](http://bit.ly/3wCvSR4)

The answer will vary depending on your use case. Data can be indexed in memory or on disk. Similarly, data formats vary, such as numbers, strings, geographic coordinates, etc. The system might be write-heavy or read-heavy. All of these factors affect your choice of database index format.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F38f892f0-81f6-41b9-9227-4d6bfa66f9eb_1474x1536.jpeg)


](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F38f892f0-81f6-41b9-9227-4d6bfa66f9eb_1474x1536.jpeg)

The following are some of the most popular data structures used for indexing data:

*   Skiplist: a common in-memory index type. Used in Redis
    
*   Hash index: a very common implementation of the “Map” data structure (or “Collection”)
    
*   SSTable: immutable on-disk “Map” implementation
    
*   LSM tree: Skiplist + SSTable. High write throughput
    
*   B-tree: disk-based solution. Consistent read/write performance
    
*   Inverted index: used for document indexing. Used in Lucene
    
*   Suffix tree: for string pattern search
    
*   R-tree: multi-dimension search, such as finding the nearest neighbor
    

This is not an exhaustive list of all database index types. Over to you:

1.  Which one have you used and for what purpose?
    
2.  There is another one called “reverse index”. Do you know the difference between “reverse index” and “inverted index”?
    

When we **merge changes** from one Git branch to another, we can use ‘git merge’ or ‘git rebase’. The diagram below shows how the two commands work.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F149b8083-361b-4784-9c41-73d8e73820f8_2421x3003.jpeg)


](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F149b8083-361b-4784-9c41-73d8e73820f8_2421x3003.jpeg)

**Git Merge**  
This creates a new commit G’ in the main branch. G’ ties the histories of both main and feature branches.

Git merge is **non-destructive**. Neither the main nor the feature branch is changed.

**Git Rebase**  
Git rebase moves the feature branch histories to the head of the main branch. It creates new commits E’, F’, and G’ for each commit in the feature branch.

The benefit of rebase is that it has **linear commit history.**

Rebase can be dangerous if “the golden rule of git rebase” is not followed.

**The Golden Rule of Git Rebase:** Never use it on public branches!

👉 Over to you: When do you usually use git rebase?

The growth of BNPL has been dramatic in recent years. The BNPL provider represents the primary interface between the merchants and the customers for both eCommerce and POS (Point of Sale).

The diagram below shows how the process works:

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff4050bfd-acd4-4c8c-80f0-67af15125f56_901x984.jpeg)


](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff4050bfd-acd4-4c8c-80f0-67af15125f56_901x984.jpeg)

**Step 0**. Bob registers with AfterPay. An approved credit/debit card is linked to this account.

**Step 1**. The "Buy Now, Pay Later" payment option is chosen by Bob when he wants to purchase a $100 product.

**Steps 2-3**. Bob's credit score is checked by the BNPL provider, and the transaction is approved.

**Steps 4-5**. A BNPL provider grants Bob a $100 consumer loan, which is usually financed by a bank. A total of $96 out of $100 is paid to the merchant immediately (yes, the merchant receives less with BNPL than with credit cards!) Bob must now pay the BNPL provider according to the payment schedule.

**Step 6-8**. Bob now pays the $25 down payment to BNPL. Stripe processes the payment transaction. It is then forwarded to the card network by Stripe. The card network must be paid an interchange fee since this goes through them as well.

**Step 9.** Bob can now receive the product since it has been released.

**Steps 10-11**. The BNPL provider receives installment payments from Bob every two weeks. Payment gateways process installments by deducting them from credit/debit cards.

Over to you: What is your experience with BNPL? Who were the providers you used?

By [flaviocopes](https://twitter.com/flaviocopes) on Twitter

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F330a7f50-7697-45e3-bdc0-dc1aba78e969_1375x1072.jpeg)


](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F330a7f50-7697-45e3-bdc0-dc1aba78e969_1375x1072.jpeg)