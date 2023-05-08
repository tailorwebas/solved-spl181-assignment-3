Download Link: https://assignmentchef.com/product/solved-spl181-assignment-3
<br>
In this assignment you will implement an online movie rental service (R.I.P. <a href="https://en.wikipedia.org/wiki/Blockbuster_LLC">Blockbuster</a><a href="https://en.wikipedia.org/wiki/Blockbuster_LLC">)</a> server and client. The communication between the server and the client(s) will be performed using a text based communication protocol, which will support renting, listing and returning of movies. Please read the entire document before starting.

The implementation of the server will be based on the <strong>Thread-Per-Client (TPC) </strong>and <strong>Reactor</strong> servers taught in class. The servers, as seen in class, do not support bi-directional message passing. Any time the server receives a message from a client it can reply back to that specific client itself, but what if we want to send messages between clients, or broadcast an announcment to all clients? The first part of the assignment will be to replace some of the current interfaces with new interfaces that will allow such a case. Note that this part changes the servers pattern and <strong>must not know</strong> the specific protocol it is running. The current server pattern also works that way (Generics and interfaces).

Once the server implementation has been extended you will have to implement an example protocol. We will implement the movie rental service over the User service textbased protocol. The User Service Text-based protocol is the base protocol which will define the message structure and base command. Given an implementation of the protocol we can implement many user service applications. The service you will build is a movie rental service. Since the service requires data to be saved about each user and available movies for rental, we will implement a simple JSON text database which will be read when the server starts and updated each time a change is made.

Note that these kinds of services that use passwords and/or money exchange require additional encryption protocols to pass sensitive data. In our assignment we will ignore security and focus on network programming.

You will also implement a simple terminal-like client in C++. To simplify matters, commands will be written by keyboard and sent “as is” to the server.




<h1>2  User Service Text based protocol</h1>

<h2>2.1  Establishing a client/server connection</h2>

Upon connecting, a client must identify themselves to the system. In order to identify, a user must be registered in the system. The <strong>LOGIN</strong> command is used to identiy. Any command (except for <strong>REGISTER</strong>) used before the login is complete will be rejected by the system.

<h2>2.2  Message encoding</h2>

A message is defined by a list of characters in UTF-8 encoding following the special character ‘
’. This is a very simple message encoding pattern that was seen in class. <strong> </strong>

<h2>2.3  Supported Commands</h2>

In the following section we will entail a list of commands supported by the User Service Text-based protocol. Each of these commands will be sent independently within the encoding defined in the previous section. (User examples appear at the end of the assignment)

Annotations:

<ul>

 <li>&lt;x&gt; – defines mandatory data to be sent with the command</li>

 <li>[x] – defines optional data to be sent with the command</li>

 <li>“x” – strings that allow a space or comma in complex commands will be wrapped with quotation mark (more than a single argument)</li>

 <li>x,… – defines a variable list of arguments</li>

</ul>

<u>Server commands:</u>

All <strong>ACK</strong> and <strong>ERROR</strong> message may be extended over the specifications, but the message prefix <u>must match</u> the instructions (reminder: testing and grading are automatic).

<h3>1) ACK [message]</h3>

The acknowledge command is sent by the server to reply to a successful request by a client. Specific cases are noted in the Client commands section.

<h3>2) ERROR &lt;error message&gt;</h3>

The error command is sent by the server to reply to a failed request. Specific cases are noted in the Client commands section.

<strong> </strong>

<strong> </strong>

<h3>3) BROADCAST &lt;message&gt;</h3>

The broadcast command is sent by the server to all <strong>logged in</strong> clients. Specific cases are noted in the Client commands section.




<u>Client commands:</u>

<h3>1) REGISTER &lt;username&gt; &lt;password&gt; [Data block,…]</h3>

Used to register a new user to the system.

<ul>

 <li>Username – The user name.</li>

 <li>Password – the password.</li>

 <li>Data Block – An optional block of additional information that may be used by the service.</li>

</ul>

In case of failure, an ERROR command will be sent by the server: ERROR registration failed Reasons for failure:

<ol>

 <li>The client performing the register call is already logged in.</li>

 <li>The username requested is already registered in the system.</li>

 <li>Missing info (username/password).</li>

 <li>Data block does not fit service requirements (defined in rental service section).</li>

</ol>

In case of successful registration an ACK command will be sent: ACK registration succeeded

<strong> </strong>

<h3>2) LOGIN &lt;username&gt; &lt;password&gt;</h3>

Used to login into the system.

<ul>

 <li>Username – The username.</li>

 <li>Password – The password.</li>

</ul>

In case of failure, an ERROR command will be sent by the server: ERROR login failed Reasons for failure:

<ol>

 <li>Client performing LOGIN command already performed successful LOGIN command.</li>

 <li>Username already logged in.</li>

 <li>Username and Password combination does not fit any user in the system.</li>

</ol>

In case of a successful login an ACK command will be sent: ACK login succeeded




<h3>3) SIGNOUT</h3>

Sign out from the server.

In case of failure, an ERROR command will be sent by the server: ERROR signout failed Reasons for failure:

<ol>

 <li>Client not logged in.</li>

</ol>

In case of successful sign out an ACK command will be sent: ACK signout succeeded <u>After a successful ACK for sign out the client should terminate!</u>

<strong>4)</strong><strong> REQUEST &lt;name&gt; [parameters,…] </strong>

A general call to be used by clients. For example, our movie rental service will use it for its applications. The next section will list all the supported requests.

<ul>

 <li>Name – The name of the service request.</li>

 <li>Parameters,.. – specific parameters for the request.</li>

</ul>

In         case     of         a          failure,             an        ERROR             command        will       be sent     by        the       server:               ERROR request &lt;name&gt; failed Reasons for failure:

<ol>

 <li>Client not logged in.</li>

 <li>Error forced by service requirements (defined in rental service section).</li>

</ol>

In case of successful request an ACK command will be sent. Specific ACK messages are listed on the service specifications.




<h1>3  Movie Rental Service</h1>

<h2>3.1  Overview</h2>

Our server will maintain two datasets with JSON text files. One file will contain the user information and the other the movie information. More about the files and the JSON format in the next section. A new user must register in the system before being able to login. Once registered, a user can use the login command to identify themselves and start interacting with the system using the <strong>REQUEST </strong>commands.




<strong>Note:</strong> the movie rental service is a “protocol in a protocol”. Your implementation should consider this fact such that it will be as easy as reconfiguring the REGISTER command additional information and implementing the different sub requests. Adding a different service should be easy given implementation of part 2. Your design should consider this fact.




<h2>3.2  Service REGISTER data block command</h2>

When a REGISTER command is processed the user created will be a normal user with credit balance 0 by default.

The service requires additional information about the user and the data block is where the user inserts that information. In this case, the only information we save on a specific user that is recieved from the REGISTER command is the users origin country.

<strong>REGISTER &lt;username&gt; &lt;password&gt; country=”&lt;country name&gt;” </strong>

<h2>3.3  Normal Service REQUEST commands</h2>

The REQUEST command is used for most of the user operations. This is the list of service specific request and their response messages. These commands are available to all logged in users.

<h3>1) REQUEST balance info</h3>

Server      returns     the     user’s     current     balance     within     an     ACK     message:

ACK balance &lt;balance&gt;

<h3>2) REQUEST balance add &lt;amount&gt;</h3>

Server adds the amount given to the user’s balance. The server will return an ACK message: ACK balance &lt;new balance&gt; added &lt;amount&gt;

Note: the new balance should be calculated after the added amount. You may assume amount is always a number greater than zero.

<h3>3) REQUEST info “[movie name]”</h3>

Server returns information about the movies in the system. If no movie name was given a list of all movies’ names is returned (even if some of them are not available for rental).

If the request fails an ERROR message is sent.

Reasons of failure:

<ol>

 <li>The movie does not exist</li>

</ol>

If the request is successful, the user performing the request will receive an ACK command: ACK info &lt;”movie name”,…&gt;.

If a movie name was given: ACK info &lt;”movie name”&gt; &lt;No. copies left&gt; &lt;price&gt; &lt;”banned country”,…&gt;

<h3>4) REQUEST rent &lt;”movie name”&gt;</h3>

Server tries to add the movie to the user rented movie list, remove the cost from the user’s balance and reduce the amount available for rent by 1. If the request fails an ERROR message is sent.

Reasons for failure:

<ol>

 <li>The user does not have enough money in their balance</li>

 <li>The movie does not exist in the system</li>

 <li>There are no more copies of the movie that are available for rental</li>

 <li>The movie is banned in the user’s country</li>

 <li>The user is already renting the movie</li>

</ol>

If the request is successful, the user performing the request will receive an ACK command:

ACK rent &lt;”movie name”&gt; success. The server will also send a broadcast to all logged-in clients: BROADCAST movie &lt;”movie name”&gt; &lt; No. copies left &gt; &lt;price&gt;

<h3>5) REQUEST return &lt;”movie name”&gt;</h3>

Server tries to remove the movie from the user rented movie list and increase the amount of available copies of the movies by 1. If the request fails an ERROR message is sent.

Reasons of failure:

<ol start="2">

 <li>The user is currently not renting the movie</li>

 <li>The movie does not exist</li>

</ol>

If the request is successful, the user performing the request will receive an ACK command: ACK return &lt;”movie name”&gt; success. The server will also send a broadcast to all loggedin clients: BROADCAST movie &lt;”movie name”&gt; &lt;No. copies left&gt; &lt;price&gt;

<h2>3.4  Admin Service REQUEST commands</h2>

These commands are only eligible to a user marked as admin. They are meant to help a remote super user to manage the list of movies. Any time a normal user attempts to run one of the following commands it will result in an error message.

<strong> </strong>

<strong>1)</strong><strong> REQUEST addmovie &lt;”movie name”&gt; &lt;amount&gt; &lt;price&gt; [“banned country”,…] </strong>

The server adds a new movie to the system with the given information. The new movie ID will be the highest ID in the system + 1. If the request fails an ERROR message is sent.

Reason to failure:

<ol>

 <li>User is not an administrator</li>

 <li>Movie name already exists in the system</li>

 <li>Price or Amount are smaller than or equal to 0 (there are no free movies)</li>

</ol>

If the request is successful, the admin performing the request will receive an ACK command: ACK addmovie &lt;”movie name”&gt; success. The server will also send a broadcast to all logged-in clients: BROADCAST movie &lt;”movie name”&gt; &lt;No. copies left&gt; &lt;price&gt;

<strong> </strong>

<strong> </strong>

<h3>2) REQUEST remmovie &lt;”movie name”&gt;</h3>

Server removes a movie by the given name from the system. If the request fails an ERROR message is sent.

Reason to failure:

<ol>

 <li>User is not an administrator</li>

 <li>Movie does not exist in the system</li>

 <li>There is (at least one) a copy of the movie that is currently rented by a user</li>

</ol>

If the request is successful, the admin performing the request will receive an ACK command: ACK remmovie &lt;”movie name”&gt; success. The server will also send a broadcast to all logged-in clients: BROADCAST movie &lt;”movie name”&gt; removed

<strong> </strong>

<strong> </strong>

<strong> </strong>

<strong> </strong>

<strong> </strong>

<strong> </strong>

<strong> </strong>

<h3>3) REQUEST changeprice &lt;”movie name”&gt; &lt;price&gt;</h3>

Server changes the price of a movie by the given name. If the request fails an ERROR message is sent.

Reason to failure:

<ol>

 <li>User is not an administrator</li>

 <li>Movie does not exist in the system</li>

 <li>Price is smaller than or equal to 0</li>

</ol>

If the request is successful, the admin performing the request will receive an ACK command: ACK changeprice &lt;”movie name”&gt; success. The server will also send a broadcast to all logged-in clients: BROADCAST movie &lt;”movie name”&gt; &lt;No. copies left&gt; &lt;price&gt;




<h1>4. JSON</h1>

In this assignment, we will work with the JSON format.




<h2>4.1 JSON</h2>

The movie rental store will keep data about its customers and its warehouse using the JSON format. You can read about JSON at <a href="https://json.org/">http://json.org</a> and see general examples at <a href="https://json.org/example.html">http://json.org/example.html</a>

In JAVA, we recommend using the <a href="https://github.com/google/gson/blob/master/UserGuide.md"><strong>GSON</strong></a> package to read, parse and write in JSON format. It is recommended to generate a java class to represent each JSON file as seen in this <a href="https://www.mkyong.com/java/how-do-convert-java-object-to-from-json-format-gson-api/">example</a><a href="https://www.mkyong.com/java/how-do-convert-java-object-to-from-json-format-gson-api/">.</a>

In our modern networking world, servers and clients send each other data using JSON format all the time. Therefore, it is a good idea to have an understanding and training with JSON for the important experience it gives.




<h2>4.2  Our JSON data</h2>

In this assignment, we will have two JSON files that are in the server-side. One is “Users.json”, which stores information about the customers registered to the online store. The other is “Movies.json”, which stores information about the warehouse, i.e. movies that the online store offers and information about them.

<strong>Every change in the state of the store must be updated into the files (movie rented, movie returned, movie removed, user registered etc.) </strong>




<h2>4.3  Users.json example</h2>

Please see the supplied file <strong>example_Users.json </strong>

The file implies that the store currently contains 3 users:

<ol>

 <li>User “john”, an admin, with password “potato”, from the United States, no movies rented and has a $0 balance.</li>

 <li>User “lisa”, a normal user (customer), with password “chips123”, from Spain, currently has (by rent) the movies “The Pursuit of Happyness” (movie id 2) and “The Notebook” (movie id 3), and has a balance of $37.</li>

 <li>User “shlomi”, a normal user (customer), with password “cocacola”, from Israel, currently has (by rent) the movies “The Godfather” (movie id 1) and “The Pursuit of Happyness” (movie id 2), and has a balance of $112.</li>

</ol>

<h2>4.4  Movies.json example</h2>

Please see the supplied file <strong>example_Movies.json </strong>

The file implies that the store currently contains 4 movies:

<ol>

 <li>The movie “The Godfather”, of price 25, which is banned in both the United Kingdom and Italy. The immediate amount available for rental is 1, and the total number of copies the store owns is 2 (but one of them is currently rented by the user shlomi as seen in the previous Users.json file)</li>

 <li>The movie “The Pursuit of Happyness”, of price 14, which is not banned in any country. The immediate amount available for rental is 3, and the total number of copies the store owns is 5 (but two of them are currently rented by users shlomi and lisa)</li>

 <li>The movie “The Notebook”, of price 5, which is not banned in any country. The immediate amount available for rental is zero (none), and the total number of copies the store owns is 1 (it is rented by lisa)</li>

 <li>The movie “Justice League”, of price 17, which is banned in Jordan, Iran and Lebanon. The immediate amount available for rental is 4, and the total number of copies the store owns is 4 (no one is renting the movie currently)</li>

</ol>

<strong>Note</strong>: you may assume movie prices and user balance is an integer.

<strong> </strong>

<strong> </strong>

<h1>5  Implementation Details</h1>

<h2>5.1  General Guidelines</h2>

<ul>

 <li>The server should be written in Java. The client should be written in C++ with BOOST. Both should be tested on Linux installed at CS computer labs.</li>

 <li>You must use maven as your build tool for the server and Makefile for the C++ client.</li>

 <li>The same coding standards expected in the course and previous assignments are expected here.</li>

</ul>

<h2>5.2  Server</h2>

When the <u>server</u> starts listening for new clients (ServerSocket bounded) it <strong><u>must</u></strong> print Server started to the screen. This will tell the automatic testing systems that we can start the simulation. Without it the assignment will receive a failing grade!




You will have to implement a single protocol, supporting both the <strong><em>Thread-Per-Client</em></strong> and <strong><em>Reactor</em></strong> server patterns presented in class. Code seen in class for both servers is included in the assignment wiki page. You are also provided with 3 new or changed interfaces:

<ul>

 <li><strong>Connections</strong> – This interface should map a unique ID for each active client connected to the server. The implementation of Connections is part of the server pattern and not part of the protocol. It has 3 functions that you must implement (You may add more if needed):

  <ul>

   <li><strong>boolean send(int connId, T msg)</strong> – sends a message T to client represented by the given connId</li>

   <li><strong>void broadcast(T msg)</strong> – sends a message T to <strong><u>all</u></strong> active clients. This includes clients that has not yet completed log-in by the User service text based protocol. Remember, Connections&lt;T&gt; belongs to the server pattern implemenration, not the protocol!. o <strong>void disconnect(int connId)</strong> – removes active client connId from map.</li>

  </ul></li>

</ul>

<strong> </strong>

<ul>

 <li><strong>ConnectionHandler&lt;T&gt;</strong> – A function was added to the existing interface.

  <ul>

   <li><strong>Void send(T msg)</strong> – sends msg T to the client. Should be used by <strong>send</strong> and <strong>broadcast</strong> in the <strong>Connections</strong></li>

  </ul></li>

</ul>

<strong> </strong>

<ul>

 <li><strong>BidiMessagingProtocol </strong>– This interface replaces the MessagingProtocol interface. It exists to support peer to peer messaging via the Connections interface. It contains 2 functions: o <strong>void start(int connectionId, Connections&lt;T&gt; connections)</strong> – initiate the protocol with the active connections structure of the server and saves the</li>

</ul>

owner client’s connection id.

<ul>

 <li><strong>void process(T message)</strong> – As in MessagingProtocol, processes a given message. Unlike MessagingProtocol, responses are sent via the <em>connections</em> object <strong>send</strong></li>

</ul>

Left to you, are the following tasks:

<ol>

 <li>Implement <strong><em>Connections&lt;T&gt;</em></strong> to hold a list of the new ConnectionHandler interface for each active client. Use it to implement the interface functions. Notice that given a connections implementation, any protocol should run. This means that you keep your implementation of <em>Connections</em> on T. <em>public class ConnectionsImpl&lt;T&gt; implements Connections&lt;T&gt; {…}.</em></li>

 <li>Refactor the <strong>Thread-Per-Client</strong> server to support the new interfaces. The <em>ConnectionHandler</em> should implement the new interface. Add calls for the new <em>Connections&lt;T&gt;</em> Notice that the ConnectionHandler&lt;T&gt; should now work with the BidiMessagingProtocol&lt;T&gt; interface instead of</li>

</ol>

MessagingProtocol&lt;T&gt;.

<ol start="3">

 <li>Refactor the <strong>Reactor</strong> server to support the new interfaces. The <em>ConnectionHandler</em> should implement the new interface. Add calls for the new <em>Connections&lt;T&gt;</em> Notice that the ConnectionHandler&lt;T&gt; should now work with the BidiMessagingProtocol&lt;T&gt; interface instead of MessagingProtocol&lt;T&gt;.</li>

 <li><strong>Tasks 1 to 3 MUST not be specific for the protocol implementation</strong>. Implement the new <em>BidiMessagingProtocol </em>and <em>MessageEncoderDecoder</em> to support the User service text based protocol as described in section 1.2. You will also need to define messages(&lt;T&gt; in the interfaces). You may add more classes as neccesery to implement the protocol (shared protocol data ect…).</li>

</ol>

<strong><u>Leading questions:</u> </strong>

<ul>

 <li>Which classes and interfaces are part of the Server pattern and which are part of the Protocol implementation?</li>

 <li>When and how do I register a new connection handler to the <strong>Connections</strong> interface implementation?</li>

 <li>When do I call <strong>start</strong> to initiate the connections list? <strong>Start</strong> must end before any call to <strong>Process</strong> What are the implications on the reactor?</li>

 <li>How do you collect a message? Are all packet types collected the same way?</li>

 <li>How do I implement <strong>BROADCAST</strong>? as it should send a message to all <strong>logged-in</strong> clients (unlike the <strong><em>broadcast</em></strong> method in Connections that sends a message to all active users).</li>

</ul>

<strong> </strong>

<strong> </strong>

<strong><u>Testing run commands:</u> </strong>

<ul>

 <li>Reactor server<strong>: </strong><strong>mvn exec:java -Dexec.mainClass=”bgu.spl181.net.impl.BBreactor.ReactorMain” Dexec.args=”&lt;port&gt;” </strong></li>

 <li>Thread per client server<strong>: </strong></li>

</ul>

<strong>mvn exec:java -Dexec.mainClass=”bgu.spl181.net.impl.BBtpc.TPCMain” Dexec.args=”&lt;port&gt;”</strong>

The <strong>server</strong> directory should contain a <strong>pom.xml</strong> file, a <strong>Database </strong>directory (for Movies.json and Users.json) and the <strong>src</strong> directory. Compilation will be done from the server folder using:

<strong>mvn compile</strong>

<h2>5.3  Client</h2>

An echo client is provided, but its a single threaded client. While it is blocking on stdin (read from keyboard) it does not read messages from the socket. You should improve the client so that it will run 2 threads. One should read from keyboard while the other should read from socket. Both threads may write to the socket. The client should receive the server’s IP and PORT as arguments. You may assume a network disconnection does not happen (like disconnecting the network cable).

The client should recive commands using the standard input. Commands are defined in previous sections. The client should print to screen any message coming from the server (<strong>ACK</strong>’s, <strong>ERROR</strong>’s and <strong>BROADCAST</strong>’s). Notice that the client should not close until he recives an <strong>ACK</strong> packet for the <strong>SIGNOUT</strong> call.

The <strong>Client</strong> directory should contain a src, include and bin subdirectories and a Makefile as shown in class. The output executable for the client is named <strong>BBclient</strong> and should reside in the bin folder after calling <strong>make</strong>.

<strong><u>Testing run commands:</u> bin/BBclient &lt;ip&gt; &lt;port&gt; </strong>