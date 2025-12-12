# CH12 -- concept of payload and webhook event payload
## objectives
You will understand the concept of

  + event
  + event handler
  + request handling
  + payload
  + webhook
  + webhook event payload

## CH12-1 -- event
When an event is triggered, it will do something.

## CH12-2 -- event handler
Event handler stores events on something and listens to something whether events are triggered or not. 

## CH12-3 -- request handling
<img width="366" height="169" alt="image" src="https://github.com/user-attachments/assets/a630031d-1837-46c7-8b55-bde2a1a2f362" />

To communicate with client-side and server-side,

it needs to 

1. prepare the request on client-side.
2. send request from client-side to server-side.
3. handles the request on the server-side and make a response (if needed)
4. send back the response from server-side to client-side.
5. client-side will recieve response sent from server-side.
6. do something with response.
7. etc.

## CH12-4 -- payloads
Payloads of request stores the context of a request. Such as

  + event info (such as event name, event id)
  + the request context itself (such as serialized JSON data)

## CH12-5 -- webhook
Webhook uses a pushing (推送) techinque. 

It is event-driven

When an server satisfies the specific condition, it will trigger an event, sending requests from server-side to client-side.   

One can say that it uses webhook on the server.

## CH12-6 -- webhook event payloads
webhook event payloads is a context of a request that is automatically sent when specific event is trigger (using webhook techinque)
