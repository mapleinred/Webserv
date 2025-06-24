# Webserv: A Custom HTTP Server

## Overview

**Webserv** is a custom HTTP/1.1 server implemented in C++. The project aims to deepen your understanding of the HTTP protocol, CGI (Common Gateway Interface), and the fundamentals of network programming. By building this server from scratch, you will learn how real-world web servers operate, handle requests, manage connections, and serve dynamic and static content.

---

## Table of Contents
- [Project Requirements Checklist](#project-requirements-checklist)
- [Features](#features)
- [Project Structure](#project-structure)
- [Key Concepts](#key-concepts)
- [How HTTP Works](#how-http-works)
- [How CGI Works](#how-cgi-works)
- [Chunked Transfer Encoding Explained](#chunked-transfer-encoding-explained)
- [How Requirements Are Implemented](#how-requirements-are-implemented)
- [Function Explanations (Key Files)](#function-explanations-key-files)
- [Build & Run](#build--run)
- [Testing](#testing)
- [Learning Resources](#learning-resources)
- [Tips for Writing Your Own HTTP Server](#tips-for-writing-your-own-http-server)

---

## Project Requirements Checklist

### General Rules
- The program must never crash or quit unexpectedly, even on memory errors.
- Provide a Makefile with at least the following rules: `$(NAME)`, `all`, `clean`, `fclean`, `re`.
- Use `c++` with `-Wall -Wextra -Werror` flags and ensure C++98 compliance (`-std=c++98`).
- Prefer C++ standard library features over C functions when possible.
- No external or Boost libraries allowed.

### Mandatory Features
- **Program Name:** `webserv`
- **Files to Turn In:** Makefile, all headers (`*.h`, `*.hpp`), sources (`*.cpp`, `*.tpp`, `*.ipp`), configuration files.
- **Arguments:** Accepts a configuration file as an argument (or uses a default path).
- **External Functions:** Only allowed to use standard C++98 and specified system calls (e.g., `execve`, `dup`, `pipe`, `fork`, `socket`, `select`, `poll`, `epoll`, `kqueue`, `accept`, `listen`, `send`, `recv`, `fcntl`, `close`, `read`, `write`, `waitpid`, `signal`, `stat`, `open`, `opendir`, `readdir`, `closedir`, etc.).
- **Libft:** Allowed.

### HTTP Server Requirements
- Must be a fully functional HTTP server in C++98.
- Must be non-blocking and use only one `poll()` (or equivalent) for all I/O (including `listen`).
- `poll()` (or equivalent) must check both read and write at the same time.
- Never read/write without going through `poll()` (or equivalent).
- Never check `errno` after read/write.
- Must be compatible with modern web browsers.
- HTTP response status codes must be accurate.
- Must provide default error pages if none are configured.
- Only use `fork()` for CGI execution.
- Must serve static websites.
- Must support file uploads from clients.
- Must implement at least GET, POST, and DELETE methods.
- Must be resilient under stress tests and remain available.
- Must support listening on multiple ports (configurable).

### Configuration File Requirements
- Inspired by NGINX's `server` block.
- Must allow:
  - Choosing port and host for each server.
  - Setting server names.
  - Defining default server for a host:port.
  - Setting up default error pages.
  - Limiting client body size.
  - Defining routes with:
    - Accepted HTTP methods
    - HTTP redirection
    - Root directory or file mapping
    - Directory listing on/off
    - Default file for directory requests
    - CGI execution based on file extension
    - File upload support and upload directory configuration
- Must work with at least one CGI (e.g., Python, PHP, etc.).
- Must provide configuration and default files to demonstrate all features.

### CGI and Chunked Transfer
- For chunked requests, the server must unchunk before passing to CGI.
- For CGI output, if no `Content-Length` is returned, EOF marks the end of data.
- CGI should be called with the requested file as the first argument and run in the correct directory.

### MacOS Specific
- May use `fcntl()` with `F_SETFL`, `O_NONBLOCK`, and `FD_CLOEXEC` only.
- All file descriptors must be non-blocking.

### Bonus Features (Optional)
- Cookie and session management.
- Support for multiple CGI.

---

## Features
- HTTP/1.1 compliant server
- Static file serving (HTML, CSS, images, etc.)
- CGI support for dynamic content (Python, C++, Shell scripts)
- Configurable via custom config files
- Custom error pages
- Multiple status codes (200, 204, 301, 400, 403, 404, 405, 409, 413, 414, 500, 501, 505)
- Support for multiple clients (concurrent connections)
- Chunked transfer encoding
- Cookie handling

---

## Project Structure

```
webserv/
├── main.cpp                # Entry point, server startup
├── Makefile                # Build instructions
├── include/                # Header files
│   ├── Cgi.hpp             # CGI handling
│   ├── Client.hpp          # Client connection logic
│   ├── ConfigFile.hpp      # Config file parsing
│   ├── Server.hpp          # Server logic
│   ├── SocketBuffer.hpp    # Buffer management
│   └── Websever.hpp        # Main server class
├── misc/                   # Miscellaneous resources
│   ├── cgi-bin/            # CGI scripts (Python, C++, Shell)
│   ├── conf/               # Example config files
│   ├── error_pages/        # Custom error HTML pages
│   ├── html/               # Static HTML/CSS files
│   └── resources/          # Images, icons, etc.
├── src/                    # Source code
│   ├── Cgi.cpp             # CGI implementation
│   ├── Client.cpp          # Client logic
│   ├── ConfigFile.cpp      # Config parsing
│   ├── Server.cpp          # Server implementation
│   └── ...                 # Other modules
```

### File/Folder Responsibilities
- **main.cpp**: Initializes and starts the server, parses command-line arguments.
- **include/**: Contains all header files for modularity and code organization.
- **src/**: Contains the main implementation files for server, client, CGI, and utilities.
- **misc/cgi-bin/**: CGI scripts for dynamic content (Python, C++, Shell).
- **misc/conf/**: Example configuration files for different server setups.
- **misc/error_pages/**: Custom error pages for various HTTP status codes.
- **misc/html/**: Static web pages and CSS.
- **misc/resources/**: Static resources (images, icons, etc.).

---

## Key Concepts

### HTTP Protocol
- **Request/Response Model**: Handles GET, POST, DELETE, etc.
- **Status Codes**: Implements standard HTTP status codes for various responses.
- **Headers**: Parses and generates HTTP headers (Content-Type, Content-Length, etc.).
- **Chunked Transfer Encoding**: Supports chunked responses for dynamic content.

### CGI (Common Gateway Interface)
- Executes external scripts (Python, C++, Shell) to generate dynamic responses.
- Passes environment variables and request data to scripts.
- Captures script output and sends it as HTTP response.

### Configuration
- Custom config files allow you to define server blocks, ports, root directories, error pages, and CGI handlers.
- Example: `misc/conf/default.conf`

### Error Handling
- Custom error pages for 400, 403, 404, 405, 409, 413, 414, 500, 501, 505, etc.
- Graceful handling of malformed requests and server errors.

### Concurrency
- Handles multiple clients using non-blocking sockets and select/poll.

---

## How HTTP Works

### HTTP Methods: GET, POST, DELETE
- **GET**: Requests data from the server. Used to retrieve static files (like HTML, images) or dynamic content (via CGI). The browser sends a GET request when you enter a URL or click a link.
- **POST**: Sends data to the server, often from a form (e.g., login, registration). The browser sends a POST request when you submit a form, with the form data included in the request body.
- **DELETE**: Requests that the server delete the specified resource. Used for RESTful APIs or admin actions.

### HTTP Status Codes Explained
Here are some of the main HTTP status codes supported by this server:

| Code | Name                  | Meaning                                                                 |
|------|-----------------------|------------------------------------------------------------------------|
| 200  | OK                    | The request was successful and the server returned the requested data.  |
| 204  | No Content            | The request was successful but there is no content to send in response. |
| 301  | Moved Permanently     | The resource has been permanently moved to a new URL.                  |
| 400  | Bad Request           | The server could not understand the request due to invalid syntax.      |
| 403  | Forbidden             | The server understood the request but refuses to authorize it.          |
| 404  | Not Found             | The requested resource could not be found on the server.                |
| 405  | Method Not Allowed    | The method is not allowed for the requested resource.                   |
| 409  | Conflict              | The request could not be completed due to a conflict with the resource. |
| 413  | Content Too Large     | The request entity is larger than the server is willing to process.     |
| 414  | URI Too Long          | The URI provided was too long for the server to process.                |
| 500  | Internal Server Error | The server encountered an unexpected condition.                         |
| 501  | Not Implemented       | The server does not support the requested functionality.                |
| 505  | HTTP Version Not Supported | The server does not support the HTTP protocol version used.      |

### How Browsers Send Requests to the Server
When you enter a URL in your browser or submit a form:
1. The browser creates an HTTP request (GET, POST, etc.) with headers and (if needed) a body.
2. The request is sent over TCP to the server's IP and port (e.g., localhost:8080).
3. The server receives the request, parses it, and determines how to respond (serve a file, run a CGI script, etc.).
4. The server sends back an HTTP response with a status code, headers, and (if applicable) a body (HTML, image, etc.).
5. The browser processes the response and displays the content or handles errors accordingly.

Example of a simple GET request sent by a browser:
```
GET /index.html HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 ...
Accept: text/html,application/xhtml+xml
...other headers...
```

Example of a POST request (e.g., form submission):
```
POST /login HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

username=foo&password=bar
```

---

## How CGI Works

CGI (Common Gateway Interface) allows the server to execute external scripts or programs to generate dynamic content. Here’s how CGI works in this project:

1. **Request Detection**: When a request targets a CGI-enabled path (e.g., `/cgi-bin/script.py`), the server recognizes it should use CGI.
2. **Environment Setup**: The server sets up environment variables (like `REQUEST_METHOD`, `QUERY_STRING`, `CONTENT_LENGTH`, etc.) according to the HTTP request.
3. **Process Forking**: The server forks a new process to execute the CGI script, passing the environment and any POST data.
4. **Script Execution**: The CGI script runs, processes input, and writes output (headers and body) to standard output.
5. **Response Handling**: The server reads the script’s output, parses the headers and body, and sends them as the HTTP response to the client.

This allows you to serve dynamic content (like database queries, form handling, etc.) using Python, C++, or shell scripts.

---

## Chunked Transfer Encoding Explained

Chunked transfer encoding is a mechanism in HTTP/1.1 that allows the server to send data in a series of chunks, useful when the total size of the response is not known in advance (for example, when generating dynamic content).

### How Chunked Encoding Works
1. **Server Response**: When the server wants to use chunked encoding, it sets the header:
   ```
   Transfer-Encoding: chunked
   ```
2. **Sending Chunks**: The response body is sent in a series of chunks. Each chunk is preceded by its size in hexadecimal, followed by a CRLF (carriage return and line feed), then the chunk data, and another CRLF. For example:
   ```
   4\r\n
   Wiki\r\n
   5\r\n
   pedia\r\n
   0\r\n
   \r\n
   ```
   - `4\r\nWiki\r\n` means a chunk of 4 bytes: "Wiki"
   - `5\r\npedia\r\n` means a chunk of 5 bytes: "pedia"
   - `0\r\n\r\n` signals the end of the response
3. **End of Message**: The last chunk is always size 0, indicating the end of the response.

### How the Server Checks and Handles Chunked Messages
- **Outgoing (Server to Client):**
  - The server checks if the response should be chunked (e.g., dynamic CGI output or unknown content length).
  - If so, it formats the response body in chunks as described above.
- **Incoming (Client to Server):**
  - For HTTP requests with chunked bodies (rare, but possible for POST), the server reads each chunk, assembles the full body, and processes it as normal.
- **Unchunking:**
  - The server (or client) reads the chunk size, then reads that many bytes, repeating until a chunk of size 0 is received.
  - The chunks are concatenated to reconstruct the original message body.

### Why Use Chunked Encoding?
- Allows the server to start sending a response before knowing its total size.
- Useful for streaming data or dynamically generated content.

---

## How Requirements Are Implemented
- **HTTP/1.1 Compliance**: Manual parsing and construction of HTTP requests and responses.
- **CGI Support**: Forks a process to execute scripts, sets up environment, and pipes output.
- **Configurable Server**: Parses custom config files to set up server blocks, routes, and error pages.
- **Custom Error Pages**: Loads and serves HTML files for error responses.
- **Chunked Encoding**: Implements chunked transfer for dynamic/large responses.
- **Static & Dynamic Content**: Serves files from disk or executes CGI scripts based on request path and config.
- **Multiple Clients**: Uses select/poll for handling concurrent connections.

---

## Function Explanations (Key Files)

### main.cpp
- **Purpose**: Entry point of the server. Parses command-line arguments, loads configuration, and starts the main server loop.

### include/Cgi.hpp & src/Cgi.cpp
- **Purpose**: Handles CGI logic. Detects CGI requests, sets up environment, executes scripts, and collects output.
- **Key Functions**:
  - `executeCgiScript()`: Runs the CGI script and returns its output.
  - `setupEnvironment()`: Prepares environment variables for the script.

### include/Client.hpp & src/Client.cpp
- **Purpose**: Manages client connections, request parsing, and response sending.
- **Key Functions**:
  - `parseRequest()`: Parses incoming HTTP requests.
  - `sendResponse()`: Sends HTTP responses to the client.

### include/ConfigFile.hpp & src/ConfigFile.cpp
- **Purpose**: Parses and stores server configuration from config files.
- **Key Functions**:
  - `parseConfigFile()`: Reads and interprets the config file.
  - `getServerBlocks()`: Returns server configuration blocks.

### include/Server.hpp & src/Server.cpp
- **Purpose**: Core server logic. Listens for connections, manages sockets, and dispatches requests.
- **Key Functions**:
  - `startServer()`: Initializes sockets and starts listening.
  - `handleConnection()`: Accepts and processes client connections.

### include/SocketBuffer.hpp
- **Purpose**: Manages buffering of incoming and outgoing data on sockets.
- **Key Functions**:
  - `readFromSocket()`: Reads data from a socket into a buffer.
  - `writeToSocket()`: Writes data from a buffer to a socket.

### include/Websever.hpp
- **Purpose**: Main server class, integrates all components and manages the server lifecycle.
- **Key Functions**:
  - `run()`: Main loop for running the server.

---

## Build & Run

### Dependencies
- C++ compiler (C++98 or later)
- For CGI: Python3, zsh (for shell scripts)

### Build
```bash
make
```

### Run
```bash
./webserv [config_file]
# Default: ./webserv misc/conf/default.conf
```

The server listens on the port specified in the config file (default: 8080).

### Access
Open your browser and navigate to `http://localhost:8080/`.

---

## Testing
- Use browsers or tools like `curl`/`httpie` to test endpoints.
- Try static files: `http://localhost:8080/index.html`
- Try CGI scripts: `http://localhost:8080/cgi-bin/database.py`
- Test error pages by requesting non-existent resources.
- Modify config files to test different server behaviors.

---

## Learning Resources
- [RFC 2616: HTTP/1.1](https://www.rfc-editor.org/rfc/rfc2616)
- [CGI Specification](https://datatracker.ietf.org/doc/html/rfc3875)
- [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/)
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

---

## Tips for Writing Your Own HTTP Server
- Start with simple static file serving, then add features incrementally.
- Use modular code: separate server, client, CGI, and config logic.
- Test each feature thoroughly (use curl, browsers, and scripts).
- Read and understand the HTTP and CGI specifications.
- Handle errors gracefully and provide useful logs.
