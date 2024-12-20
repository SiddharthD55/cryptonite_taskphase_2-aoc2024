In this task, I worked through a vulnerability within Wareville's bank, specifically a **race condition** in the fund transfer functionality, which allowed malicious actors to exploit it and conduct fraudulent transactions. The goal was to reproduce this behavior in a controlled environment and understand the underlying flaws that allowed it to happen.

#### Understanding Race Conditions

A **race condition** occurs when multiple processes attempt to modify a shared resource simultaneously, and the order in which these processes execute can affect the final outcome. In the context of web applications, this can lead to unintended behaviors, such as performing multiple actions when only one was intended, or in this case, transferring funds more than once.

For this task, the vulnerability in the Wareville Bank system stemmed from a failure to handle concurrent transactions. When multiple transfer requests were sent simultaneously, the system failed to correctly synchronize access to the account balances, resulting in negative balances and inflated amounts in the target account.

#### The Attack Flow

1. **Identifying the Vulnerability**: 
   The bank application allows users to transfer funds between accounts. Upon inspecting the application, I discovered that the fund transfer functionality was vulnerable to race conditions. The system processed each transaction sequentially without proper synchronization, which allowed simultaneous requests to bypass the integrity checks and perform multiple transfers.

2. **Setting Up Burp Suite**: 
   To exploit the race condition, I configured Burp Suite to intercept and manipulate HTTP requests between my browser and the bank application. I captured the requests during a fund transfer action and sent them to the **Repeater** tab for further manipulation.

3. **Creating Multiple Requests**:
   In the Repeater tab, I cloned the initial POST request (which transferred funds from my account) multiple times and adjusted the request parameters (e.g., the transfer amount and target account). By duplicating these requests and sending them in parallel, I initiated multiple fund transfers at once.

4. **Executing the Race Condition**:
   After creating 10 duplicate requests, I grouped them together in Burp Suite for simultaneous execution. By sending these requests all at once, I instructed the system to deduct the same amount from my account multiple times without locking the database or handling concurrent transactions properly. This resulted in a negative balance in my account and an inflated balance in the recipient account.

#### Technical Breakdown of the Exploit

The underlying issue was a **Time-of-Check to Time-of-Use** (TOCTOU) vulnerability. When a user initiated a transfer, the application first checked if the user had enough funds. However, the transfer was not completed atomically. Between the time the system checked the balance and when it updated the account balances, there was a window where multiple requests could be processed, leading to the transfer of more funds than actually existed in the account.

Here’s how the flawed code might look:

```python
if user['balance'] >= amount:
    conn.execute('UPDATE users SET balance = balance + ? WHERE account_number = ?', 
                 (amount, target_account_number))
    conn.commit()

    conn.execute('UPDATE users SET balance = balance - ? WHERE account_number = ?', 
                 (amount, session['user']))
    conn.commit()
```

Since the two database operations are committed separately and without any locking mechanisms, a race condition arises when multiple requests are processed concurrently, allowing each request to see the same balance and complete the transfer.

#### Fixing the Vulnerability

To prevent race conditions like this, several mitigation techniques should have been implemented:

1. **Atomic Transactions**:
   The bank should have used **atomic transactions** to ensure that both the debit and credit actions are part of a single, indivisible operation. This way, if one part of the transaction fails, the entire process is rolled back, ensuring data consistency.

2. **Mutex Locks**:
   The application could have implemented **mutex locks** to ensure that only one request can access the account balance at a time. This would prevent simultaneous requests from interfering with each other.

3. **Rate Limiting**:
   Applying **rate limiting** to critical functions like fund transfers could help mitigate the risk of such attacks by restricting the number of requests that can be sent in a short time frame.

#### Exploiting the Vulnerability

Here’s how I performed the exploit step by step:

1. **Logging In**: I logged into the Wareville Bank application using the provided credentials.
2. **Initiating a Fund Transfer**: I initiated a fund transfer, which captured the POST request in Burp Suite.
3. **Duplicating the Request**: I cloned the request in Burp Suite’s Repeater tab and changed parameters like the account number and transfer amount.
4. **Simulating Parallel Requests**: Using Burp Suite’s "Send group (parallel)" feature, I sent all requests simultaneously.
5. **Verifying the Outcome**: After executing the requests, I checked the account balances in the application. As expected, the sender’s balance became negative, and the recipient account had an inflated balance.

#### In the end...

This demonstrated the importance of properly handling concurrent operations in web applications. By exploiting the race condition, I was able to manipulate the system into transferring more funds than were available, resulting in a loss for the bank. Implementing proper locking mechanisms, atomic transactions, and rate limiting would have prevented this attack.

In the real world, such vulnerabilities can be devastating, and it's crucial for developers to consider concurrency and race conditions during the design and testing phases of their applications. 

Now that the vulnerability has been demonstrated and understood, the next step is to implement these fixes to ensure the system operates securely and efficiently.

**Answers**

*What is the flag value after transferring over $2000 from Glitch's account?* **THM{WON_THE_RACE_007}**
