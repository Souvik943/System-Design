Functional Requirements :

1. User to User Chat
2. User to Group Chat
3. Able to send multi-media (images / texts)
4. Last seen
5. Read/Delivered receipts

Non-Functional Requirements :

1. No latency (should feel like a real time conversation)
2. High Availability
3. Scalable
4. No lag

Capacity Estimations :

Traffic :

DAU : 1 billion

Average messages sent by each person  : 20

Messages sent everyday : 20 * 1 billion = 20 billion

Total Messages sent per year : 20 billion * 365 ~ 7300 billion = 7.3 trillion

Storage :

Average size of each message = 300 KB

Storage needed for a year = 300 KB * 7.3 trillion = 2200 trillion KB = 2200 PB

1. WebSocket Handler : Lightweight servers which connect the current active users
2. WebSocket Manager connected to Redis :
    1. Stores which active user is connected to which websocket handler
    2. Stores information about websocket handlers
    3. Connected to a REDIS (which stores all the data)
    4. When a client changes its connected from websocket handler to any other , that is also updated in the Websocket Manager
3. Message Service connected to Cassandra :
    1. After knowing where to send the message , the message is sent to message service and the message is stored in the CASSANDRA DB
    2. Performs validations if required
    3. Then it sends to its respective Websocket handler and eventually to the recipient
4. Group Messaging Handler :
    1. It performs some validations / authorizations .
    2. This queries to Group messaging service , to find out all the users with the id (group_id_1 …..) .
    3. Sends the required messages to its users .
5. Group Messaging Service :
    1. It finds out all the users to whom , the message is being sent to , if they are present and all  validations .
6. Asset Service with Amazon S3 and CDN :
    1. This helps in storing the multimedia messages to be sent .

## User A to User B Flow (One on One messaging) :

### Happy Flow :

1. User A , sends request to WebSocket Handler 1 to send the message to User B
2. WebSocket Handler 1 , requests the WebSocket-Manager to find out the WebSocket Handler to which User B is connected
3. Meanwhile parallely , the message is also sent to message-service , which stores the cassandra DB (Reason : What if User B disconnects , and WebSocket Handler 2 is not able to locate User B , to store the messages temp store to DB)
4. After finding out the User B through WebSocket Handler 2 , the meesage is sent to User B

### Read / Delivered Reciepts :

1. Once User B gets the message , sends back a delivered reciept ack to User A with the help of WebSocket-Manager .
2. Once User B reads the message , it sends back a read reciept ack to User A with the help of WebSocket-Manager .

### If any user goes offline :

1. User A sends a message to User B
2. User B is not connected to server
3. User A message gets stored in the message DB .
4. In that case , when User B comes online , it connects to a WebSocket Handler 3 and then talk to the Message Server asking , if there are any messages in sent state present in the DB .
5. So basically User B , WebSocket Hndler 3 keeps on sending a request to Message Service .


    Improvement : Maybe we can keep a messaging queue , in here , whenever the offline user comes online , the Kafka automatically sends all the messages to the User .


### When the sender itself is offline :

→ When the sender itself is offline

→ We can store the messages somewhere on the device itself

→ When the user goes online , it sends the messages stored .

## Group Messaging :

Happy Flow :

1. User A sends a message to a group G1
2. It flows thru websocket-handler → Message-Service .
3. Message-Service puts all the message and everything to a Kafka queue
4. Kafka-Queue forwards it to Group-Messaging-Handler
5. Group-Messaging-Handler performs some validations
6. Sends to Group Service , it finds out the users present in that group
7. Then , the Group-Messaging-Handler , sends all the messages to its users separately and it flows as is .

## Multi-Media Messages :

User A tries to send an image to User B

1. Its a 2 way process :
    1. First the image is sent to the Asset Service and it stores the image/video in the S3 DB and generates image_id and stored .
    2. image_id , is then sent as request rather than the whole image .
    3. And it then flows like a normal messaging user to user

Improvement : When an event is occuring and people are sending same images multiple times , do we need to upload same image so many times .

Rather , what we can do is , while uploading the image , we generate a hash for that image and send it to our service and store that data along with metadata .