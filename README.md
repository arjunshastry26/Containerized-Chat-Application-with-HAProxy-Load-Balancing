-> How do we scale web sockets?
when multiple clients are trying to connect to a server , they can do individual 1 to 1 connection as its a very resource intensive way of doing it , that's why we use reverse proxy to reduce the strain as they act as an  intermediary between a client and a web server
 ( A reverse proxy is a server that sits between clients (like web browsers or mobile apps) and backend servers, handling incoming requests and forwarding them to the appropriate backend server. )
When revrse proxy are established and a client connects to a server , it treats that as a single TCP connection to that perticular serverʼs websocket , so each connection via reverse proxy would be treated as a tcp connection to that perticular websocket .

![image](https://github.com/user-attachments/assets/75241119-2dea-457b-85ef-b39fd3dd7f4e)

 
We run into a problem here: frequent requests from the client canʼt be properly load balanced because WebSocket connections are stateful over TCP. Once a client connects to a particular WebSocket server, the connection is persistent, meaning the client cannot simply switch to another server without disconnecting and reconnecting
Here Reverse proxy would act as a Level 4 load balancer  There are 2 levels of load balancer , one at L4 that is transport layer which does load balancing at TCP/UDP socket level another is L7 load balancer that is at the application layer which load balances websockets and application )
Before the actual solution to our Problem statement we need to know redis➖
Redis is a NOSQL database ( you dont need to mention primary fields )
Redis uses tcp
Has a request response model like HTTP
Has own msg format
Supports PUB/SUB  let me explain ):
Just like Kafka we can Publish and subscribe to  a channel in redis too , i.e Publisher: The news anchor sends updates (messages).
Subscribers: People watching the channel receive those updates instantly.
Basically PUB/SUB helps it sync across multiple channels so each client with same channel can recive the same msg ( also called clustering) ]
Now our solution ➖

![image](https://github.com/user-attachments/assets/43d11724-0bfc-452a-9a3f-71fb1b727547)

 
Now just like the previous senario we are using a reverse proxy called Hapoxy for load balancing also we are using a redis server.
What happens is when redis is there all the servers with websockts would subscribe to a channel we create via redis , in this case live chat room  hence a PUBSUB is established now .
When a “HIˮ msg is sent from a client it goes to its dedictaed server , the server as its subscribed to the chatroom channel sends it to redis , redis code is trggered here and it pushes this msg to all the subscribed servers.
when the servers get these msg they push to on to their respective clients and all the clients now access to the msg.

-> To run this 

1) build this image as "docker build -t wsapp ."
2) than run docker-compose up
3)for same browser
ws = new WebSocket("ws://localhost:8080");

ws.onopen = () => {
  console.log("✅ Connected");
  ws.send("Hello! I'm client");
};

ws.onmessage = message => console.log(`Received: ${message.data}`);

4)for different persons browser
ws = new WebSocket("ws://10.14.142.226:8080");

ws.onopen = () => {
  console.log("✅ Connected");
  ws.send("Hello! I'm client from another device");
};

ws.onmessage = message => console.log(`Received: ${message.data}`);
