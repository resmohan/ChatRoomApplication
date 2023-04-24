# Chatroom Application

<!-- TOC -->
* [Chatroom Application](#chatroom-application)
    * [Function](#function)
    * [How to Launch the App](#how-to-launch-the-app)
    * [Description of Key Classes and Methods](#description-of-key-classes-and-methods)
        * [Protocol part](#protocol-part)
        * [Server side](#server-side)
        * [Client side](#client-side)
    * [Description of the Project](#description-of-the-project)
        * [UI Design / Operating Steps](#ui-design--operating-steps)
        * [Assumptions and the corresponding logic to ensure correctness](#assumptions-and-the-corresponding-logic-to-ensure-correctness)
<!-- TOC -->

## Function

The project aims to build a simple chat room application to finish the following tasks:
* Server side: Responsible for managing the client connections (up to 10 clients can be connected and in a chat room at one time), accepting messages from one client and sending the messages to all attached clients;
* Client side: Send a message that is public in the chat room, or that goes directly to a single, specified client;
* Protocol part: Deal with the input message; transfer and process the message as the MessageFormat type.

Details are available in the [RequirementDoc](ChatRoomApplication/RequirementDoc.pdf).

## How to Launch the App

This project can be run from Intellij or in terminal:
1. In Intellij configuration,
    * Server setting:
        * set ```module``` as ```java 17``` and ```ChatRoomApplication.main```;
        * set ```Main class``` as ```chatRoom.chatRoomApp.ChatServerApp```.
        * Then, run the configuration directly to launch the server.
    * Client setting:
        * set ```module``` as ```java 17``` and ```ChatRoomApplication.main```;
        * set ```Main class``` as ```chatRoom.chatRoomApp.ChatClientApp```;
        * set ```Program argumets``` as ```the host and port of server```.
        * Then, run the configuration directly to launch a client.
        * To run the program, the host and port of server should be ```localhost``` and ```3000```.
    * Set the server configuration then run it; Set the separate configuration for each client then run the configuration of each client separately.
      

2. In terminal:
    1. Run task ```doAll``` in the build.gradle file, and the build folder will be generated automatically. The Javadoc and test report can be found in build folder.
    2. Change the current directory to the root directory of this project.
       ```
       cd <project_path>
       ```
    3. Launch this project via gradle command.
    * Server:
      ```
      ./gradlew runServer -q --console=plain
      ```
    * Client: To run this project locally, the server host is 'localhost' and the server port is '3000'
      ```
      ./gradlew runClient --args '<server host> <server port>' -q --console=plain
      ```

       
3. Add the Sentence Generator JAR: ```a4-1.0.jar```
   1. First, run ```dependencies``` in ```build.gradle``` first to add the external JAR to the dependency
   2. Then, in ```Project explore``` window, find the ```externalLibrary``` folder, right click and choose ```Add as Library...```.
   3. Now, we can run ```doAll``` task or run configurations with the ```a4-1.0.jar```.

## Description of Key Classes and Methods
### Protocol part
1. [MessageFormat.java](ChatRoomApplication/src%2Fmain%2Fjava%2Fassignment6%2FchatRoomApp%2FMessageFormat.java)

   Represents the frame data in the server-client communication.

   The fields of MessageFormat class:
    * success: boolean which represents the status of connect or disconnect response
    * message: message passed during the server-client communication
    * clients: set of active clients in chat room
    * sender: client which sends a request
    * recipient: client which receives a response


2. [ChatRoomProtocol.java](ChatRoomApplication/src%2Fmain%2Fjava%2Fassignment6%2FchatRoomApp%2FChatRoomProtocol.java)

   This interface is designed to represent the chat room protocol providing the read and write functions for client and server to process the message frame data.

   For example of sending connect request, the client side calls the ```writeConnectRequest()``` method to send connection request to the server side; The server side receives the frame data, then reads the message by calling the ```readConnectRequest()``` method and writes the response by calling the ```writeConnectResponse()``` method to send back a connection response to the client side; Finally, the client side receives the response message by calling the ```readConnectResponse()``` method and prints the response message to the console.
    1. ```writeConnectRequest``` method: send the connection request message to server (client side)
        * parameter:   
          DataOutputStream dataOutputStream -> the sender's data output stream;  
          MessageFormat messageFormat -> the frame data of connection request
    2. ```readConnectRequest``` method: receive the connection request message from client and store the information in the MessageFormat object (server side)
        * parameter: DataInputStream dataInputStream -> the server's data input stream in which the message is sent from specific client
        * return: the request information from client that are stored as MessageFormat type
    3. ```writeConnectResponse``` method: send the processed connection response message to sender client (server side)
        * parameter:  
          DataOutputStream dataOutputStream -> the data output stream corresponding to the sender client;  
          MessageFormat messageFormat -> the processed connection response message
    4. ```readConnectResponse``` method: receive the connection response message from server and store the information in the MessageFormat object (client side)
        * parameter: DataInputStream dataInputStream -> the client's data input stream in which the message is sent from server
        * return: the response information from server that are stored as MessageFormat type

### Server side
3. [ChatServer.java](ChatRoomApplication/src%2Fmain%2Fjava%2FchatRoom%2FchatRoomApp%2FChatServer.java)

   Represents the server in chat room application
    1. ```start``` method: set the executor pool for holding the ClientHandler and when a client want to connect, a new ClientHandler will be added to the executor pool. This makes sure that the process for each client is running in parallel


4. [ClientHandler.java](ChatRoomApplication/src%2Fmain%2Fjava%2FchatRoom%2FchatRoomApp%2FClientHandler.java)

   The runnable class to maintain the server message handler


5. [ServerMessageHandler.java](ChatRoomApplication/src%2Fmain%2Fjava%2FchatRoom%2FchatRoomApp%2FServerMessageHandler.java)

   Represents the message handler in server side containing the protocol handler and the map of current active clients. All the process in server side will be implemented by this class.
    1. ```process``` method: process the message frame data from the client side based on the received message type. After processing, return the result message frame data to specified clients


### Client side
6. [ChatClient.java](ChatRoomApplication/src%2Fmain%2Fjava%2FchatRoom%2FchatRoomApp%2FChatClient.java)

   Represents the client in chat room application
    1. ```start``` method: request to connect to the server, and initialize the input and server message handler
    2. ```requestConnection``` method: send a connection request to the server
    * return: boolean flag to indicate whether the client is connected to the server.
    3. ```invokeUiServerHandlers``` method: set up and start the threads for input and server message handler


7. [UiHandler.java](ChatRoomApplication/src%2Fmain%2Fjava%2FchatRoom%2FchatRoomApp%2FUiHandler.java)

   The runnable class to maintain the UI input message handler


8. [UiInputHandler.java](ChatRoomApplication/src%2Fmain%2Fjava%2FchatRoom%2FchatRoomApp%2FUiInputHandler.java)

   Represents the input message handler in client side. All the input message in client side will be implemented by this class.
    1. ```process``` method: process the console input from the client based on the message header. After processing, send the result message frame data to the server.


9. [ServerHandler.java](ChatRoomApplication/src%2Fmain%2Fjava%2FchatRoom%2FchatRoomApp%2FServerHandler.java)

   The runnable class to maintain the client message handler


10. [ClientMessageHandler.java](ChatRoomApplication/src%2Fmain%2Fjava%2FchatRoom%2FchatRoomApp%2FClientMessageHandler.java)

    Represents the client message handler class which helps to read the message from server side and print it to console.
    1. ```process``` method: process the message frame data from the server based on the received message type. After processing, print the processed message to the console of client

## Description of the Project
### UI Design / Operating Steps
1. Launch the server, and the server console will display a message if the startup is successful.
   

2. Launch the clients. Clients will join the chat room by running their configurations separately. The client console will display a welcome message and then ask the client for a username.
   

3. After inputting a valid username, the client console will tell the client how many other clients there are in current chat room, and will also display the help message including the message headers.
   

4. There are six command available for clients:

   4.1 input ```?``` to get the help message
   
   4.2 input ```who``` to get the information of other active clients in current chat room
   
   4.3 input ```@all <message>``` to send the broadcast message to all other clients
   
   4.4 input ```@user <message>``` to send the direct message to the specified user
   
   4.5 input ```!user``` to send a random insult message, and all the other clients will receive a message showing the sender, recipient and the message
   
   4.6 input ```logoff``` to disconnect from current chat room
   
  

### Assumptions and the corresponding logic to ensure correctness
1. The server of chat room should launch first, then the clients are allowed to join the chat room by providing correct server host and port. If a client try to connect to a server that does not exist or connect with wrong host and port, a failed message will show up.
      

2. To launch a client, two arguments, the host and port server, must be provided. If not provide, a fail message will be thrown as followed.

   ```
   There should be one argument of server IP and the second one as server port.
   ```
   In addition, if the format of server IP and port is not correct, a fail message will be thrown as followed.
   ```
   The input server IP is not valid.
   The input server port is not valid.
   ```

3. The seat number of chat room is 10, so the eleventh client will be not allowed to join the chat room. If the eleventh client tries to enter the chat room, a fail message will show up and the client app will close.
   
   
4. In order to distinguish different clients, we stipulate that the username of new client should not be the same as the existing clients; otherwise, a fail message will show up and the client app will close.
   
  
5. If the client tries to send message to who is not in current chat room, a fail message will show up.
   
   
6. The client also cannot send a message to itself. If the client tries to send a direct message or insult message to itself, a fail message will show up.
   
   
7. If there is no other active client in current chat room, the client will fail to send message and a fail message will show up.
   
   
8. Only the following commands are valid: ```?```, ```logoff```, ```who```, ```@all <message>```, ```@user <message>``` and ```!user```.

   All the other messages are illegal, like ```?test```, ```? test```, ```logoff test```, ```who test```, ```!user test```
   
   In particular, if the commands ```@all``` and ```@user``` are not followed by a message, a fail message will show up to ask for the message.

9. Only one server can listen on the same port on the same machine at the same time. If two servers tries to listen on the same port of same device, the second one will fail to launch and a fail message will show up.
      
10. If the client app collapses, the current client will be removed from current chat room.

