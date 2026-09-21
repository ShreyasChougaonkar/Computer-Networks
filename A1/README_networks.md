# Chat Server README

## Overview

This project implements a multi-client chat server in C++ that supports private messaging, broadcasting, and group messaging. The server authenticates users from a file and manages connections using multi-threading, ensuring safe concurrent access to shared data structures.

---

## Features

### Implemented Features

- **User Authentication**
  - Users are authenticated using credentials loaded from a file (`users.txt`).
  - Prevents duplicate logins.

- **Private Messaging**
  - Clients can send private messages using the `/msg` command.
  - If the target user is offline, the sender is notified.

- **Broadcast Messaging**
  - Clients can send a broadcast message to all active users using the `/broadcast` command.

- **Group Management**
  - **Create Group:** Users can create groups using `/create_group`.
  - **Join Group:** Users can join existing groups using `/join_group`.
  - **Group Messaging:** Clients can send messages to all members of a group using `/group_msg`.
  - **Leave Group:** Clients can exit a group using `/leave_group`.

- **Graceful Exit**
  - Users can log out with `/exit`, and the server removes them from all groups and active user lists.

### Not Implemented Features

- **Persistent Group Data**
  - Groups are maintained in memory only. They are lost when the server restarts.
  
- **Advanced Message History or Logging**
  - Message history is not stored.
  
- **Encryption/SSL for Secure Communication**
  - All data is sent in plain text.

- **Scalability Beyond a Single Machine / Distributed Setup**
  - The server runs on a single host (localhost).

---

## Design Decisions

### Concurrency Model

- **Thread-per-Connection:**  
  Each new client connection spawns a new thread. This design was chosen because it:
  - Simplifies connection handling by isolating each client’s interactions.
  - Is sufficient for a moderate number of simultaneous clients.
  - Avoids the complexity of process-based or asynchronous I/O designs for this assignment.

### Synchronization

- **Mutex Locking:**  
  A global `std::mutex` (`client_mutex`) is used to protect shared data structures (e.g., `clients` map and `groups` map). Synchronization is necessary because:
  - Multiple threads might access or modify these structures concurrently (e.g., when sending messages or updating groups).
  - It prevents race conditions and ensures data consistency.

### Socket and Networking

- **TCP Sockets:**  
  The server uses TCP sockets for reliable message delivery.
  
- **Localhost Communication:**  
  The server binds to `127.0.0.1` on port `12345`, assuming local testing and development.

### Design Trade-offs

- **Simplicity vs. Scalability:**  
  The thread-per-connection model is easy to implement and understand. However, it may not scale well with a very high number of clients. For this assignment, the expected load is low to moderate, so the trade-off is acceptable.

- **In-Memory Data Structures:**  
  Using `std::unordered_map` and `std::unordered_set` provides fast lookup and insertion. This is ideal for the assignment’s scale but might require adaptation (e.g., persistent storage) for production systems.

---

## Implementation Details

### High-Level Overview of Functions

- **`main()` Function:**
  - Loads user credentials from a file.
  - Creates and binds a server socket.
  - Listens for incoming client connections.
  - Spawns a new thread for each client using `handle_client`.

- **`load_users()` Function:**
  - Reads `users.txt` line by line.
  - Parses username and password pairs and stores them in `user_credentials`.

- **`authenticate_client()` Function:**
  - Prompts the client for a username and password.
  - Validates credentials against the `user_credentials` map.
  - Ensures a user is not already logged in.

- **`handle_client()` Function:**
  - Manages the main loop for handling commands from an authenticated client.
  - Parses commands and delegates to functions handling private messages, broadcast, group management, etc.

- **Messaging Functions:**
  - **`send_message()`**: Sends a string message to a specified client socket.
  - **`send_private_message()`**: Finds the target client and sends a private message.
  - **`send_broadcast()`**: Broadcasts a message to all active clients.
  - **Group Messaging Functions:**  
    - `create_group()`, `join_group()`, `leave_group()`, and `message_group()` manage group-based communications.

## Code Flow Diagram and Description

The diagram below illustrates the detailed flow of the server's operation, from initialization through client connection, authentication, command processing, and eventual client exit.

```mermaid
flowchart TD
    %% Server Initialization
    A[Start Server] --> B[load_users(): Load user credentials from file]
    B --> C[Create TCP socket]
    C --> D[Bind socket to IP (127.0.0.1) and port (12345)]
    D --> E[Listen for incoming connections]
    
    %% Accepting New Clients
    E --> F[Accept new client connection]
    F --> G[Spawn a new thread with handle_client(client_socket)]
    
    %% Client Thread - Authentication Phase
    G --> H[handle_client() function begins]
    H --> I[authenticate_client() is called]
    I --> J[Prompt client: "Enter username:"]
    J --> K[Receive username]
    K --> L{Is username valid?}
    L -- No --> M[Send error: "[Error]: Invalid username" and loop back to J]
    L -- Yes --> N{Is user already logged in?}
    N -- Yes --> O[Send error: "[Error]: User already logged in" and loop back to J]
    N -- No --> P[Prompt client: "Enter password:"]
    P --> Q[Receive password]
    Q --> R{Is password correct?}
    R -- No --> S[Send error: "[Error]: Incorrect password" and loop back to J]
    R -- Yes --> T[Send welcome message: "Welcome to the server!"]
    T --> U[Add client_socket and username to active clients map]
    
    %% Client Thread - Command Processing Loop
    U --> V[Enter command processing loop]
    V --> W[Receive command from client]
    W --> X{Determine command type}
    
    %% Command Handling
    X -- "/msg" --> Y[Parse target username and message]
    Y --> Z[Call send_private_message()]
    
    X -- "/broadcast" --> AA[Parse broadcast message]
    AA --> AB[Call send_broadcast()]
    
    X -- "/create_group" --> AC[Parse group name]
    AC --> AD[Call create_group()]
    
    X -- "/join_group" --> AE[Parse group name]
    AE --> AF[Call join_group()]
    
    X -- "/group_msg" --> AG[Parse group name and message]
    AG --> AH[Call message_group()]
    
    X -- "/leave_group" --> AI[Parse group name]
    AI --> AJ[Call leave_group()]
    
    X -- "/exit" --> AK[Call remove_client() and exit thread]
    
    X -- Default --> AL[Send error: "[Error]: Oops...Invalid command"]

    %% Looping back for more commands
    AL --> V
    AH --> V
    AJ --> V
    Z --> V
    AB --> V

## Diagram Description

### Server Initialization:
- **Start Server:**  
  The server begins its execution.
- **Load Users:**  
  The function `load_users()` reads user credentials from a file (`users.txt`) and loads them into memory.
- **Create TCP Socket:**  
  A TCP socket is created for network communication.
- **Bind Socket:**  
  The socket is bound to the IP address `127.0.0.1` and port `12345`.
- **Listen:**  
  The server listens for incoming connections.

### Accepting New Clients:
- **Accept Connection:**  
  The server waits for and accepts a new client connection.
- **Spawn Thread:**  
  For each new client, a new thread is spawned to handle the client's requests through the `handle_client()` function, enabling concurrent client handling.

### Client Authentication:
- **Handle Client Begins:**  
  The new thread enters the `handle_client()` function.
- **Authenticate Client:**  
  The `authenticate_client()` function is invoked to manage the login process.
- **Username Prompt:**  
  The client is prompted to enter a username.
- **Username Validation:**  
  - If the username is invalid, an error message is sent, and the process repeats.
  - If the username is valid but already logged in, an error message is sent, and the process repeats.
- **Password Prompt:**  
  Upon successful username validation, the client is prompted to enter a password.
- **Password Validation:**  
  - If the password is incorrect, an error message is sent, and the process repeats.
  - If correct, the client receives a welcome message.
- **Update Active Clients:**  
  The client's socket and username are added to the active clients map.

### Command Processing Loop:
- **Enter Loop:**  
  After authentication, the client enters a loop waiting for commands.
- **Receive Command:**  
  The server receives a command from the client.
- **Determine Command Type:**  
  The command is parsed to determine which operation to perform.
- **Command Handling:**  
  Depending on the command type:
  - **Private Messaging (`/msg`):**  
    The target username and message are parsed and passed to `send_private_message()`.
  - **Broadcast (`/broadcast`):**  
    The message is parsed and sent to all clients via `send_broadcast()`.
  - **Create Group (`/create_group`):**  
    The group name is parsed and a new group is created via `create_group()`.
  - **Join Group (`/join_group`):**  
    The group name is parsed, and the client is added to the group using `join_group()`.
  - **Group Messaging (`/group_msg`):**  
    The group name and message are parsed and sent via `message_group()`.
  - **Leave Group (`/leave_group`):**  
    The group name is parsed and the client is removed using `leave_group()`.
  - **Exit (`/exit`):**  
    The client is logged out, cleaned up using `remove_client()`, and the thread exits.
  - **Invalid Command:**  
    If an unknown command is received, an error message is sent.
- **Loop Continuity:**  
  After handling a command, the client returns to the command processing loop for further input.

## Testing

### Correctness Testing

#### Unit Testing
- Each function (e.g., user authentication, private messaging, group functions) was tested individually using mock client inputs.

#### Integration Testing
- Multiple clients were connected simultaneously to ensure that message delivery, group management, and synchronization worked correctly.

### Stress Testing

#### Simulated Load
- Scripts were used to simulate multiple client connections and simultaneous messaging.
- The thread-per-connection model was observed for proper handling under stress.

#### Edge Cases
- Attempting to log in with an already logged-in user.
- Sending messages to offline users.
- Creating groups with duplicate names.
- Handling empty or malformed commands.

---

## Challenges and Resolutions

### Race Conditions
- **Challenge:** Concurrent access to shared data structures led to race conditions.
- **Solution:** Introduced a mutex (`client_mutex`) to serialize modifications and ensure consistency.

### Thread Management
- **Challenge:** Properly managing detached threads and cleaning up client data upon disconnection.
- **Solution:** Ensured that each thread uses a cleanup function (`remove_client()`) to update shared structures.

### Parsing Client Input
- **Challenge:** Handling varied input formats and spaces in group names.
- **Solution:** Implemented strict checks (e.g., disallowing spaces in group names) and provided error feedback.

---

## Server Restrictions

- **Maximum Clients:**  
  Limited by system resources and the thread-per-connection model; typically supports up to a few hundred concurrent connections on modern hardware.
  
- **Maximum Groups:**  
  No explicit upper bound in the code; limited by available memory.
  
- **Maximum Group Members:**  
  No explicit limit; the group is implemented as an unordered set, limited by system memory.
  
- **Message Size:**  
  The buffer size is set to 1024 bytes. Messages larger than this size may be truncated or require additional handling.

- *Single instance of a user:*  
  If a user has logged in, another instance of the same user cannot log in unless the previous instance is closed. This is because allowing multiple instances creates inconsistencies in the groups of which the user is a member.
  
- **User names and group names:**  
  Usernames and groupnames are assumed to not have spaces in them.

---

## Individual Contributions

- **Aaditi Agrawal - 220006: (33.33% contribution)**  
  - Designed the overall architecture and implemented user authentication, ensured proper synchronization and mutex handling.
  
- **Ritesh Baviskar - 220286: (33.33% contribution)**  
  - Assisted in designing architecture, handled group management functions and client handling and messaging function.
  
- **Taneshq Zendey - 221123: (33.33% contribution)**  
  - Assisted in designing architecture, conducted integration of functionalities, stress and manual testing, wrote documentation, and assisted in debugging.

---

## Sources

- **Books:**
  - *"Unix Network Programming"* by W. Richard Stevens
- **Websites and Blogs:**
  - Tutorials on socket programming in C++.
  - Articles on multi-threading and synchronization using C++11 features.
- **Documentation:**
  - C++ Reference for STL containers and threading libraries.

---

## Declaration

We hereby declare that this project is our original work. We have not engaged in any form of plagiarism, and all external sources have been appropriately referenced.

---

## Feedback

### Suggestions
- Consider enhancing the server to support asynchronous I/O for better scalability.
- Implement logging and persistent group/message storage for production-level features.

### Feedback for the Assignment
- The assignment provided a challenging yet educational experience in managing network programming, multi-threading, and synchronization. Additional guidelines on handling large volumes of data would be beneficial.
