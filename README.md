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
- [Project Architecture Analysis](#project-architecture-analysis)
- [Advanced Features Implementation](#advanced-features-implementation)
- [Performance Considerations](#performance-considerations)
- [Security Features](#security-features)

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
- **Outgoing (Server to Client)**:
  - The server checks if the response should be chunked (e.g., dynamic CGI output or unknown content length).
  - If so, it formats the response body in chunks as described above.
- **Incoming (Client to Server)**:
  - For HTTP requests with chunked bodies (rare, but possible for POST), the server reads each chunk, assembles the full body, and processes it as normal.
- **Unchunking:**:
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

---

## Project Architecture Analysis

### Overall Design Philosophy

This HTTP server follows a **non-blocking, event-driven architecture** using epoll on Linux systems. The core design principles include:

- **Single-threaded with event multiplexing**: Uses one epoll instance to handle all I/O operations
- **State machine-based parsing**: HTTP requests are parsed through well-defined states
- **Modular component design**: Separation of concerns across different classes
- **Configuration-driven behavior**: NGINX-inspired configuration system
- **Process isolation for CGI**: Fork-based CGI execution for security and stability

### Core Components Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   main.cpp      │────│  ConfigFile     │────│  ServerInfo     │
│ (Entry Point)   │    │  (Config Parser)│    │  (Server Data)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client        │────│      CGI        │────│   SocketBuffer  │
│ (Connection     │    │  (Script Exec)  │    │  (Buffer Mgmt)  │
│  Management)    │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## Detailed Component Analysis

### 1. Main Server Loop (`main.cpp`)

**Purpose**: Entry point that orchestrates the entire server lifecycle.

**Key Functions & Implementation**:

- **`sigchldHandler(int sig)`**: 
  - Handles zombie process cleanup for CGI execution
  - Uses `waitpid(-1, NULL, WNOHANG)` for non-blocking child reaping
  - Prevents accumulation of zombie processes

- **`main()` Function Flow**:
  1. **Signal Setup**: Installs `SIGCHLD` handler and ignores `SIGPIPE`
  2. **Configuration Loading**: Parses config file and creates `ServerInfo` objects
  3. **Socket Initialization**: Creates listening sockets for each server block
  4. **Epoll Setup**: Initializes epoll instance for event-driven I/O
  5. **Main Event Loop**: Continuously processes network events

**Event Loop Architecture**:
```cpp
while (1) {
    nfds = epoll_wait(Client::getEpollfd(), events.data(), events.size(), -1);
    for (int i = 0; i < nfds; ++i) {
        if (events[i].data.fd > max_connfd) {
            // Handle existing client data (read/write)
            client->socketRecv() / client->socketSend()
        } else {
            // Accept new connection
            sockfd = accept(events[i].data.fd, &addr, &addrlen);
            // Create new Client object
        }
    }
    // Cleanup closed connections
    // Process CGI zombie processes
}
```

### 2. Configuration Management (`ConfigFile.hpp/cpp`)

**Purpose**: Parses NGINX-style configuration files and stores server settings.

**Key Data Structures**:

- **`ServerInfo`**: Contains complete server configuration
  ```cpp
  struct ServerInfo {
      std::vector<int> connfds;           // Listening socket FDs
      std::vector<RouteInfo> routes;      // Location blocks
      std::vector<std::string> names;     // Server names
      std::vector<std::pair<std::string, std::string>> ip_addrs; // Host:port pairs
      static std::map<int, std::string> error_pages;  // Error page mappings
      static size_t max_bodysize;         // Maximum request body size
  };
  ```

- **`RouteInfo`**: Defines behavior for specific URL paths
  ```cpp
  struct RouteInfo {
      bool dir_list;                      // Enable directory listing
      int http_methods;                   // Allowed HTTP methods (bitmask)
      std::string root;                   // Document root path
      std::string dfl_file;              // Default index file
      std::string redirect;              // Redirect URL
      std::string upload_dir;            // Upload directory
      std::string prefix_str;            // URL prefix to match
      std::vector<std::string> cgi_extensions; // CGI file extensions
  };
  ```

**Parsing Implementation**:
- **Token-based parsing**: Uses keyword-value pairs separated by '='
- **Context-aware**: Supports nested `server` and `location` blocks
- **Error handling**: Validates configuration syntax and throws exceptions
- **Method bitmasks**: Uses bit flags for HTTP method permissions

### 3. Client Connection Management (`Client.hpp/cpp`)

**Purpose**: Manages individual client connections through their entire lifecycle.

**State Machine Design**:

The Client class implements two interrelated state machines:

**Parse States** (HTTP parsing progression):
```cpp
enum ParseState {
    ERROR = -1,      // Parse error occurred
    START_LINE,      // Parsing "GET /path HTTP/1.1"
    HEADERS,         // Parsing HTTP headers
    MSG_BODY,        // Reading request body (POST)
    FINISHED,        // Ready to generate response
};
```

**I/O States** (Communication flow):
```cpp
enum IOState {
    RECV_HTTP = 0,   // Receiving HTTP request from client
    SEND_HTTP,       // Sending HTTP response to client
    RECV_CGI,        // Receiving output from CGI script
    SEND_CGI,        // Sending input to CGI script
    CONN_CLOSED,     // Connection terminated
};
```

**Key Methods & Implementation**:

- **`socketRecv()`**: 
  - Handles incoming data based on current I/O state
  - Non-blocking receive using `MSG_DONTWAIT`
  - Routes data to HTTP parser or CGI handler
  
- **`socketSend()`**:
  - Sends response data incrementally
  - Tracks send progress with iterators
  - Handles partial sends gracefully

- **`parseHttpRequest()`**:
  - Implements incremental HTTP parsing
  - Handles chunked transfer encoding
  - Validates HTTP compliance

### 4. HTTP Request Processing (`ClientParseHttp.cpp`, `ClientPerformRequest.cpp`)

**HTTP Parsing Flow**:

1. **Start Line Parsing** (`parseStartLine()`):
   - Extracts HTTP method, URI, and version
   - Validates method support (GET, POST, DELETE)
   - Finds matching route configuration
   - Resolves file paths and checks permissions

2. **Header Parsing** (`parseHeaders()`):
   - Builds header map from key-value pairs
   - Validates Host header for virtual hosting
   - Configures transfer encoding (chunked/content-length)
   - Determines if request body expected

3. **Body Processing** (`parseMsgBody()`):
   - Handles Content-Length based body reading
   - Implements chunked encoding decoder
   - Enforces maximum body size limits

**Request Processing Flow**:

1. **Path Resolution** (`filterRequestUri()`):
   - Extracts query parameters and path info
   - Handles directory requests with index files
   - Checks for CGI extensions
   - Sanitizes paths to prevent directory traversal

2. **Method Handlers**:
   - **GET**: Serves static files or directory listings
   - **POST**: Handles file uploads and form data
   - **DELETE**: Removes specified files

### 5. CGI Script Execution (`Cgi.hpp/cpp`, `ClientRunCgi.cpp`)

**Purpose**: Executes external scripts to generate dynamic content.

**CGI Architecture**:
```
Client Process                    CGI Process
┌─────────────────┐              ┌─────────────────┐
│                 │  socketpair  │                 │
│   HTTP Parser   │◄────────────►│  Script Engine  │
│                 │              │                 │
└─────────────────┘              └─────────────────┘
```

**Implementation Details**:

- **`runCgiScript()`**:
  - Creates bidirectional socket pair using `socketpair()`
  - Forks child process to execute the CGI script
  - Registers CGI socket with epoll for event handling
  - Creates CGI object to manage communication

- **`executeCgi()`** (Child Process):
  - Changes directory to script location
  - Sets up environment variables (REQUEST_METHOD, QUERY_STRING, etc.)
  - Redirects stdin/stdout to socket pair
  - Executes script using `execve()`

- **Environment Setup** (`initCgiEnv()`):
  - Converts HTTP headers to CGI environment variables
  - Sets standard CGI variables (PATH_INFO, CONTENT_LENGTH)
  - Handles file upload directory configuration

**CGI Communication Flow**:
1. **POST Requests**: Send body data to script's stdin
2. **Script Processing**: CGI script processes input and generates output
3. **Response Parsing**: Parse CGI output headers and body
4. **Client Response**: Forward processed data to HTTP client

### 6. Error Handling and HTTP Status Codes

**Error State Management**:
- **Graceful degradation**: Invalid requests generate appropriate HTTP error responses
- **Custom error pages**: Configurable HTML pages for different error codes
- **Logging**: Debug output for troubleshooting (can be disabled)

**Supported Status Codes**:
- **2xx Success**: 200 (OK), 204 (No Content)
- **3xx Redirection**: 301 (Moved Permanently)
- **4xx Client Error**: 400 (Bad Request), 403 (Forbidden), 404 (Not Found), 405 (Method Not Allowed), 409 (Conflict), 413 (Content Too Large), 414 (URI Too Long)
- **5xx Server Error**: 500 (Internal Server Error), 501 (Not Implemented), 505 (HTTP Version Not Supported)

---

## Advanced Features Implementation

### Chunked Transfer Encoding

**Outgoing (Server to Client)**:
- Automatic chunking for CGI responses without Content-Length
- Proper chunk formatting with hexadecimal size headers
- Zero-length chunk termination

**Incoming (Client to Server)**:
- Chunk size parsing and validation
- Reassembly of chunked request bodies
- Unchunking before CGI processing

### Virtual Host Support

- **Host header validation**: Matches incoming requests to server configurations
- **Server name matching**: Supports multiple server names per configuration
- **Default server selection**: Falls back to first server if no match found

### File Upload Handling

- **Multipart form data**: Supports file uploads via POST requests
- **Upload directory configuration**: Configurable destination directories
- **Size limits**: Enforces maximum body size restrictions

### Connection Management

- **Keep-alive support**: Handles persistent connections
- **Timeout handling**: Closes inactive connections
- **Resource cleanup**: Proper file descriptor and memory management

---

## Performance Considerations

### Non-blocking I/O
- All socket operations use `MSG_DONTWAIT` flag
- Epoll edge-triggered mode for efficient event notification
- Incremental parsing to handle partial requests

### Memory Management
- Buffer reuse and efficient string operations
- Iterator-based sending to avoid large memory copies
- Automatic cleanup of closed connections

### Process Management
- Fork only for CGI execution (minimal process overhead)
- Proper zombie process cleanup
- Signal handling for robust operation

---

## Security Features

### Path Sanitization
- `realpath()` and `canonicalize_file_name()` for path resolution
- Prevention of directory traversal attacks
- Access permission checking

### CGI Isolation
- Process isolation through forking
- Working directory changes for script execution
- Environment variable sanitization

### Input Validation
- HTTP protocol compliance checking
- Header size and count limits
- Request body size restrictions

---

## Technical Implementation Highlights

### Advanced HTTP Features

#### Chunked Transfer Encoding Implementation

**Encoding (Server to Client)**:
```cpp
// In CGI response handling:
// 1. When CGI doesn't provide Content-Length
// 2. Format: [hex-size]\r\n[data]\r\n...\0\r\n\r\n
// 3. Automatic chunking for dynamic content

std::string formatChunk(const std::string& data) {
    std::ostringstream oss;
    oss << std::hex << data.size() << "\r\n";
    oss << data << "\r\n";
    return oss.str();
}
```

**Decoding (Client to Server)**:
```cpp
// In request body parsing:
// 1. Parse hex chunk size
// 2. Read exactly that many bytes
// 3. Concatenate chunks until size 0
// 4. Reconstruct original message body

int unchunkRequest() {
    // Parse chunk size from hex
    // Read chunk data
    // Append to message body
    // Continue until zero-sized chunk
}
```

#### Virtual Host Implementation

**Host Header Processing**:
```cpp
// 1. Extract Host header from request
// 2. Match against configured server names
// 3. Fall back to IP:port matching
// 4. Use default server if no match

int checkServerName() {
    std::string hostHeader = _headers["Host"];
    // Check server names first
    for (auto& name : _server.names) {
        if (hostHeader.find(name) != std::string::npos) {
            return 0; // Match found
        }
    }
    // Check IP:port combinations
    // Return 404 if no match
}
```

### Memory Management Strategy

#### Buffer Management
- **Receive buffers**: Dynamic string buffers that grow as needed
- **Send buffers**: Iterator-based progressive sending
- **File buffers**: Efficient file content streaming
- **CGI buffers**: Separate buffers for CGI communication

#### Resource Cleanup
```cpp
// RAII-style resource management:
class Client {
    ~Client() {
        closeFds();     // Close all file descriptors
        cleanup();      // Free allocated memory
    }
};

// Static cleanup for server shutdown:
void signalHandler(int sig) {
    Client::closeOpenFds();    // Close all tracked FDs
    exit(0);
}
```

### Event-Driven Architecture Details

#### Epoll Integration
```cpp
// Event registration for new connections:
void registerEvent(int fd, uint32_t events) {
    struct epoll_event ev;
    ev.events = events;
    ev.data.fd = fd;
    epoll_ctl(epollfd, EPOLL_CTL_ADD, fd, &ev);
}

// Event modification for state changes:
void modifyEvent(int fd, uint32_t events) {
    struct epoll_event ev;
    ev.events = events;
    ev.data.fd = fd;
    epoll_ctl(epollfd, EPOLL_CTL_MOD, fd, &ev);
}
```

#### State Transition Management
```cpp
// Client state transitions:
RECV_HTTP -> parseRequest() -> SEND_HTTP
RECV_HTTP -> parseRequest() -> SEND_CGI -> RECV_CGI -> SEND_HTTP

// Each transition updates epoll events accordingly
void setIOState(int newState) {
    _iostate = newState;
    switch (newState) {
        case RECV_HTTP:
            registerEvent(_clientfd, EPOLLIN);
            break;
        case SEND_HTTP:
            registerEvent(_clientfd, EPOLLOUT);
            break;
        // ... other states
    }
}
```

### CGI Security and Isolation

#### Process Isolation
```cpp
// Fork creates isolated process:
switch (fork()) {
    case 0: // Child process
        // Change working directory
        chdir(_route->root.c_str());
        
        // Set up minimal environment
        clearenv();
        setEnvVariables();
        
        // Execute with minimal privileges
        execve(scriptPath, argv, envp);
        break;
        
    default: // Parent process
        // Continue serving other clients
        // Monitor child via SIGCHLD
}
```

#### Environment Sanitization
```cpp
// Only safe environment variables are passed:
std::vector<char*> initCgiEnv() {
    std::vector<char*> env;
    
    // Standard CGI variables only
    env.push_back(strdup("REQUEST_METHOD=GET"));
    env.push_back(strdup("QUERY_STRING=..."));
    env.push_back(strdup("CONTENT_LENGTH=..."));
    
    // No system environment inheritance
    // No PATH modification
    // No dangerous variables
    
    return env;
}
```

---

## Project Development Journey and Lessons Learned

### Design Evolution

#### Initial Approach vs Final Implementation

**Original Design**: Simple socket-based server with basic file serving
**Evolution**: Full HTTP/1.1 compliant server with CGI support

**Key Design Decisions**:
1. **Event-driven architecture**: Chosen for scalability over threaded model
2. **State machine parsing**: Ensures robust HTTP protocol handling
3. **Process isolation for CGI**: Security and stability over performance
4. **Configuration-driven behavior**: Flexibility for different use cases

#### Technical Challenges Overcome

**1. Non-blocking I/O Complexity**:
- **Challenge**: Managing partial reads/writes
- **Solution**: Iterator-based progressive processing
- **Learning**: Importance of state tracking in network programming

**2. CGI Integration**:
- **Challenge**: Bidirectional communication with external processes
- **Solution**: Socketpair-based IPC with epoll integration
- **Learning**: Process management and IPC mechanisms

**3. HTTP Protocol Compliance**:
- **Challenge**: Handling all HTTP edge cases and malformed requests
- **Solution**: Strict state machine with comprehensive error handling
- **Learning**: Protocol implementation requires meticulous attention to detail

**4. Memory Management**:
- **Challenge**: Preventing leaks in long-running server
- **Solution**: RAII principles and careful resource tracking
- **Learning**: System programming requires disciplined resource management

### Code Quality and Best Practices

#### C++98 Compliance
- **No modern C++ features**: Standard library algorithms and containers only
- **Manual memory management**: No smart pointers, careful new/delete pairing
- **Iterator usage**: Extensive use of STL iterators for efficiency
- **Function objects**: Using std::bind2nd and std::mem_fun_ref

#### Error Handling Strategy
```cpp
// Comprehensive error checking:
if (socketcall() == -1) {
    perror("socketcall");
    // Graceful degradation
    return error_response();
}

// Exception safety:
try {
    risky_operation();
} catch (std::exception& e) {
    cleanup_resources();
    log_error(e.what());
    return error_state();
}
```

#### Performance Optimizations
- **Minimal copying**: Use of iterators and references
- **Buffer reuse**: Avoiding frequent allocations
- **Efficient algorithms**: O(log n) lookups where possible
- **System call optimization**: Batching operations when possible

---

## Future Enhancement Opportunities

### Potential Improvements

#### Performance Enhancements
1. **Connection pooling**: Reuse connections for keep-alive
2. **Buffer optimization**: Circular buffers for reduced copying
3. **Sendfile support**: Zero-copy file transfers
4. **Compression**: Gzip compression for text content

#### Feature Extensions
1. **HTTPS support**: SSL/TLS encryption
2. **HTTP/2 support**: Modern protocol features
3. **WebSocket support**: Real-time communication
4. **Authentication**: Basic/digest authentication

#### Scalability Improvements
1. **Multi-threading**: Thread pool for CPU-intensive tasks
2. **Process pooling**: Pre-forked CGI processes
3. **Load balancing**: Multiple server instances
4. **Caching**: Static content caching

### Architecture Refinements

#### Code Organization
1. **Module separation**: Better separation of concerns
2. **Plugin architecture**: Loadable modules for extensions
3. **Configuration API**: Runtime configuration changes
4. **Monitoring API**: Built-in performance metrics

#### Testing Framework
1. **Unit test suite**: Automated testing for all components
2. **Integration tests**: End-to-end scenario testing
3. **Performance benchmarks**: Automated performance regression testing
4. **Fuzzing**: Automated security testing

---

## Summary and Conclusion

### Project Achievements

This HTTP server implementation successfully demonstrates:

1. **Complete HTTP/1.1 compliance**: Proper handling of methods, headers, and status codes
2. **CGI integration**: Dynamic content generation through external script execution
3. **Configuration flexibility**: NGINX-inspired configuration system
4. **Robust error handling**: Graceful handling of all error conditions
5. **Performance optimization**: Event-driven, non-blocking architecture
6. **Security awareness**: Input validation and process isolation

### Technical Mastery Demonstrated

#### Systems Programming Skills
- **Network programming**: Socket management, epoll usage
- **Process management**: Fork/exec, signal handling, IPC
- **File system operations**: File access, directory manipulation
- **Memory management**: Efficient allocation and deallocation

#### Software Engineering Practices
- **Modular design**: Clean separation of responsibilities
- **Error handling**: Comprehensive error detection and recovery
- **Resource management**: Proper cleanup and leak prevention
- **Documentation**: Thorough code documentation and user guides

#### Protocol Implementation
- **HTTP/1.1 standard**: Full compliance with RFC specifications
- **CGI standard**: Proper environment setup and communication
- **Chunked encoding**: Correct implementation of transfer encoding
- **Virtual hosting**: Multi-domain server capability

### Educational Value

This project serves as an excellent learning resource for:

1. **Understanding web servers**: How HTTP servers actually work internally
2. **Network programming**: Practical application of socket programming
3. **System-level programming**: Process management and IPC
4. **Protocol implementation**: Real-world protocol compliance
5. **Software architecture**: Scalable, maintainable system design

The codebase demonstrates that building a production-quality HTTP server requires careful attention to protocol details, robust error handling, efficient resource management, and thoughtful architectural decisions. This implementation provides a solid foundation for understanding how modern web servers operate and can serve as a starting point for more advanced features and optimizations.

---
