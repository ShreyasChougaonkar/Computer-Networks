# TCP Handshake 

This submission contains two source files:

- **server.cpp**: The server-side code.
- **client.cpp**: The client-side code that implements a simple TCP handshake using raw sockets.

A Makefile is provided to compile these files into executables.

## Prerequisites

- A C++ compiler (e.g., `g++`).
- Linux system (raw sockets require root privileges).

> **Note:** Because the code uses raw sockets and manually crafts packets, administrative privileges are required to run both the server and the client.

## How to Build the Code

1. Open a terminal in the repository's root directory.
2. Run the following command to compile both `server.cpp` and `client.cpp`:

    ```bash
    make
    ```

    This will generate two executables: `server` and `client`.

## How to Run the Code

1. **Run the Server:**

   First, start the server in one terminal. Since the server might use networking capabilities that require administrator privileges, run it as root if necessary:

    ```bash
    sudo ./server
    ```

2. **Run the Client:**

   Open a second terminal and start the client. Again, run as root if necessary:

    ```bash
    sudo ./client
    ```

3. **Observe the Handshake:**

   The client will send a SYN packet, wait for the SYN-ACK from the server, and finally send an ACK to complete the handshake process. Both programs will output status messages to the terminal as the handshake proceeds.

## Cleaning Up

To remove the generated executables, run:

```bash
make clean
