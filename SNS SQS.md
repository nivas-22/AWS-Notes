
## What is Communication ?
 - sending msg from src to des. - But it would be meaning ful info.

### Queues:  **Buffer Location or Temporary location to stay the message.**
### Types of Queue:  
 - DLQ - Undelivered messages (Outbox)
 - Local - Inbox
 - Remote - send

## Types of Communication:

- **Synchronized** - Both sender and receiver should be active
- **Asynchronized** - Sender should be active and won't be bother about receiver

### Types of Asynchronized : 

-  1 to 1 : **Simple Queue service**
- 1 to Many : **Simple Notification service**

![[Pasted image 20240217101642.png]]


![[Pasted image 20240217102449.png]]



SQS SNS SES


message - Meaningful info 
parts - sub, body , meta data
types - req, res, info, file, raw data..
queue - buffer location / temorary location

types of queues: 
    
DLQ - Undeleivered messages - outbox
local - inbox - 
remote - sent
alias - duplucate queue


communication : sending msg from src to dest

types of commu:

  - Synchronized mode of com   -- acceptence & acknowledge  - both S/R should be active
  - Asynchronized mode of com  -- sending coninously --> sender should be active and dont bother about the receiver

types of Asynch mode of com:

1 to 1 : simple queue service
1 to M : simple Notification service

ARN : amazon resource name : use to connect our internal services

