# Design a URL Shortener

## Functional Requirements :

1. A long URL will be provided to us
2. We need to convert the long URL to a short URL
3. We can store it in a DB and should have an expiry

## Non-Functional Requirements :

1. It should be highly available
2. There should be NO-Latency (while redirection)

## Capacity Estimations :

Traffic :

1. URL / second = 1000
2. URL / year generated = 1000 * 60 * 60 * 24 * 365 ~ 30 billion each year
3. Read : Write = 10 : 1
4. URL reads / year = 300 billion per year
5. Characters to be included = a-z + A-Z + 0-9
    1. So thats a total of 26 + 26 + 10 = 62
    2. So , the first place of URL can be a choice of 62 chars , the 2nd character again will be from a choice of 62 chars and so on ……
    3. So it goes with the power of 62
    4. After calculation 62^7 ~ 300 billion (whats calculated in point 4)
    5. Sp we get the length of URL = 7

Storage :

From point (2) , there will be URL Generated / year = 30 billion per year

- Each URL size = 20 bytes
- If we want to store it for 10 years = 30 * 10 * 20 = 6000 billion = 6 TB

## **NAIVE APPROACH 1 :**

We can introduce all these along with a count cache

- Whenever a request comes , the count cache gives an incremented number , which is then converted to base 62 , using it the service generates a url .

Problem :

There is a single point of failure .

## NAIVE APPROACH 2 :

Problem :

1. It can happen that the multiple URL Shortener services deployed may generate same characters (Duplicacy issue) because the multiple caches can easily generate a duplicate number .

## OPTIMAL APPROACH :

While CREATING Short URLs (SET-API) :

- All remains the same , expect ,
- there is a Token Generator Service , which generates a token , within a range , lets say : 0 - 1 million to a server A , 1 - 2 million to server B …..
- And with that range the URL generator service takes a number serially and with that number base 62 , a set of characters / alpha-numeric characters are generated and hence numerous unique URLs are generated .

While GETTING the URL (GET-API) :

- When the server is hit with a GET-Request , it checks in the cache first
    - (We keep the most requested / visited short URL in the cache)
- If not found , then it goes to the DB and searches for it .