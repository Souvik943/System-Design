Functional Requirements :

1. There are multiple services that wants to send notifications
2. Notifications can be received on phones, laptops tablets
3. Notifications can be :
    1. Email notifications
    2. SMS notifications
    3. Device Notifications
4. Users need to provide their details :
    1. Ph number
    2. Email ID
    3. Name

Non-Functional Requirements :

1. There can be slight delay
2. Reliable and consistent (guaranteed delivery)


High-Level Design explanation :
1. Services will hit the notification service (Message body + User details)
2. There will be a DB + Cache (which will maintain details about the user and their registered device + A message template if its for email / SMS / Phone )
    - Example : When we install a new application , it asks to allow notifications , If we provide an Yes , then the device token id , is saved in the DB with its corresponding user details .
3. Authorization will also take place (in case of transactions/financial notifications)
4. A Rate limiter should also be present in case there is flood of notifications
5. After getting the desired user details and verifying it , Its sent to its corresponding Message Queue Section (we can make it async (if required))
6. Then it goes to these workers
7. Here there can be two possibilities :
    1. If its a success , then it goes forward to the 3rd party notification client and from there on , it goes to the user .
    2. If its a failure , its written in the log file and then its sent back to the message queue , again to retry sending it to the proper one . Then if it fails , again the same process is repeated .