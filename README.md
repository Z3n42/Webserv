<div align="center">

# 🌐 Webserv

### HTTP/1.1 Server Implementation in C++98

<p>
  <img src="https://img.shields.io/badge/Score-100%2F100-success?style=for-the-badge&logo=42" alt="Score"/>
  <img src="https://img.shields.io/badge/Language-C++98-00599C?style=for-the-badge&logo=c%2B%2B&logoColor=white" alt="C++98"/>
  <img src="https://img.shields.io/badge/Team-3_Developers-blue?style=for-the-badge" alt="Team"/>
  <img src="https://img.shields.io/badge/Circle-05-purple?style=for-the-badge&logo=42&logoColor=white" alt="Circle 05"/>
  <img src="https://img.shields.io/badge/42-Urduliz-000000?style=for-the-badge&logo=42&logoColor=white" alt="42 Urduliz"/>
</p>

*A non-blocking, event-driven HTTP/1.1 web server with CGI support, virtual hosts, and file upload capabilities.*

[Overview](#-overview) • [Team](#-team) • [Architecture](#-architecture) • [Features](#-features) • [Usage](#-usage)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Team](#-team)
- [Architecture](#-architecture)
- [Key Components](#-key-components)
- [Features](#-features)
- [Configuration](#-configuration)
- [HTTP Methods](#-http-methods)
- [CGI Support](#-cgi-support)
- [Error Handling](#-error-handling)
- [Compilation](#-compilation)
- [Usage](#-usage)
- [Testing](#-testing)
- [Technical Challenges](#-technical-challenges)
- [Resources](#-resources)

---

## 🎯 Overview

**Webserv** is a fully functional HTTP/1.1 server written in C++98 that implements the **Reactor design pattern** for efficient I/O multiplexing. The server is capable of serving static files, executing CGI scripts, handling file uploads, and managing multiple virtual hosts simultaneously, all while maintaining non-blocking I/O operations.

### Why This Project Matters

- **Real-world networking**: Hands-on experience with TCP/IP, sockets, and HTTP protocol
- **Scalable architecture**: Event-driven design for handling multiple simultaneous connections
- **Production concepts**: Configuration parsing, virtual hosts, CGI execution, error handling
- **C++98 constraints**: Modern patterns implemented with C++98 limitations

### Project Specifications

- **HTTP/1.1 compliance**: GET, POST, DELETE methods
- **Non-blocking I/O**: Single-threaded event loop using `select()` or `poll()`
- **CGI execution**: Python script execution with environment variable passing
- **Virtual hosts**: Multiple server configurations per port
- **File uploads**: Multipart form data handling
- **Configuration file**: nginx-style configuration format
- **Custom error pages**: Configurable error responses (400, 404, 405, 413, 500, 504)

---

## 👥 Team

This project was collaboratively developed by three developers:

<table>
<tr>
<td align="center" width="33%">
<img src="https://avatars.githubusercontent.com/Z3n42" width="100px;" alt="Iñigo Gonzalez"/><br />
<b>Iñigo Gonzalez</b><br />
<a href="https://github.com/Z3n42">@ingonzal</a><br />
</td>
<td align="center" width="33%">
<img src="https://avatars.githubusercontent.com/manu-garcia" width="100px;" alt="Manu Garcia"/><br />
<b>Manu Garcia</b><br />
<a href="https://github.com/manu-garcia">@manugarc</a><br />
</td>
<td align="center" width="33%">
<img src="https://avatars.githubusercontent.com/mikgarci42" width="100px;" alt="Mik Garcia"/><br />
<b>Mik Garcia</b><br />
<a href="https://github.com/mikgarci42">@mikgarci</a><br />
</td>
</tr>
</table>

**Collaboration Model**: Weekly meetings, feature branch workflow, code reviews, shared debugging sessions

---

## 🏗️ Architecture

### Reactor Design Pattern

The server implements the **Reactor pattern**, a synchronous event demultiplexing architecture that handles concurrent service requests with a single thread.

**Pattern Components**:

| Component | Implementation | Purpose |
|-----------|----------------|---------|
| **Reactor** | `Reactor.cpp` | Central event loop, manages file descriptors with `select()` |
| **Event Handler** | `EventHandler.cpp` (abstract) | Interface for all event processing handlers |
| **Concrete Handlers** | `AcceptConnectionEventHandler`, `ServeRequestEventHandler` | Specific logic for connections and requests |
| **Handle** | File descriptors (sockets) | I/O handles monitored by the reactor |

**Event Flow**:
```
1. Reactor waits for events (select/poll)
2. Event detected on file descriptor
3. Reactor dispatches to appropriate EventHandler
4. Handler processes event (accept, read, write)
5. Handler updates reactor state
6. Loop continues
```

<details>
<summary><b>Reactor Implementation Details</b></summary>

**Reactor Core** (`Reactor.cpp`):
- Maintains `std::map<int, EventHandler*>` mapping FD → Handler
- Uses `fd_set` for `select()` multiplexing
- Calculates max FD dynamically
- Dispatches `handleEvent()` when FD ready

**Why Reactor?**:
- **Scalability**: Handle thousands of connections with single thread
- **Determinism**: No race conditions (no threading)
- **Resource efficient**: One thread, minimal context switching
- **Separation of concerns**: Handler logic independent of event loop

</details>

---

## 🔑 Key Components

### Core Classes

<table>
<tr>
<td width="50%" valign="top">

### Configuration Layer
- **ConfigFileParser**: Reads `httpd.conf` file
- **Parse**: Parses server/location blocks
- **ServerConfig**: Server-level configuration
- **LocationConfig**: Location-specific rules
- **VirtualHostServer**: Virtual host management

</td>
<td width="50%" valign="top">

### Request Processing
- **HTTPRequestFactory**: Creates appropriate request handlers
- **HTTPRequest** (abstract): Base class for requests
- **StaticFileHTTPRequest**: Serves files
- **CGIHTTPRequest**: Executes CGI scripts
- **UploadFileRequest**: Handles file uploads
- **RedirectionHTTPRequest**: HTTP redirects
- **ErrorHTTPRequest**: Error responses

</td>
</tr>
<tr>
<td width="50%" valign="top">

### HTTP Components
- **HTTPMethod**: GET/POST/DELETE enum
- **HTTPHeader**: Request/response headers
- **HTTPBody**: Request/response body
- **HTTPRequestStatus**: Request state machine
- **HTTPResponse** + 6 error subclasses: Response generation

</td>
<td width="50%" valign="top">

### Utilities
- **MIMETypes**: Content-Type mapping (60+ types)
- **StringTools**: String manipulation utilities
- **Path**: Path validation and manipulation
- **Log**: Logging system
- **RequestTools**: Request parsing utilities

</td>
</tr>
</table>

### Project Structure

```
webserv/
├── inc/                          # Headers (31 files)
│   ├── Reactor.hpp               # Event loop core
│   ├── EventHandler.hpp          # Handler interface
│   ├── AcceptConnectionEventHandler.hpp
│   ├── ServeRequestEventHandler.hpp
│   ├── ConfigFileParser.hpp      # Config reader
│   ├── Parse.hpp                 # Config parser
│   ├── ServerConfig.hpp          # Server config
│   ├── LocationConfig.hpp        # Location config
│   ├── VirtualHostServer.hpp     # Virtual host
│   ├── HTTPRequest.hpp           # Request base class
│   ├── HTTPRequestFactory.hpp    # Request factory
│   ├── StaticFileHTTPRequest.hpp # Static files
│   ├── CGIHTTPRequest.hpp        # CGI execution
│   ├── UploadFileRequest.hpp     # File uploads
│   ├── RedirectionHTTPRequest.hpp
│   ├── ErrorHTTPRequest.hpp
│   ├── HTTPResponse.hpp          # Response base
│   ├── HTTPResponse400-504.hpp   # Error responses
│   ├── HTTPMethod.hpp            # HTTP methods
│   ├── HTTPHeader.hpp            # Headers
│   ├── HTTPBody.hpp              # Body
│   ├── HTTPRequestStatus.hpp     # Request state
│   ├── MIMETypes.hpp             # Content types
│   ├── StringTools.hpp           # String utils
│   ├── Path.hpp                  # Path utils
│   ├── Log.hpp                   # Logging
│   └── RequestTools.hpp          # Parsing utils
├── src/                          # Implementations (31 .cpp files)
├── main.cpp                      # Server entry point
├── Makefile                      # Build system
├── httpd.conf                    # Configuration file
└── www/                          # Document root
    ├── html/                     # Static files
    │   ├── index.html
    │   ├── styles.css
    │   └── uploader/
    ├── cgi-bin/                  # CGI scripts
    │   ├── getcgi.py             # GET CGI
    │   ├── postcgi.py            # POST CGI
    │   ├── error.py              # Error test
    │   └── infinite.py           # Timeout test
    ├── error/                    # Error pages
    │   ├── 400.html              # Bad Request
    │   ├── 403.html              # Forbidden
    │   ├── 404.html              # Not Found
    │   ├── 405.html              # Method Not Allowed
    │   ├── 413.html              # Payload Too Large
    │   ├── 500.html              # Internal Server Error
    │   └── 504.html              # Gateway Timeout
    └── server2/                  # Virtual host 2
        └── server2_index.html
```

**Statistics**: 31 headers, 31 implementations, ~12,000+ lines of code

---

## ✨ Features

### HTTP/1.1 Compliance

**Supported Methods**:
- **GET**: Retrieve resources (static files, CGI output)
- **POST**: Send data to server (CGI input, file uploads)
- **DELETE**: Delete resources from server

**HTTP Features**:
- Persistent connections (Connection: keep-alive)
- Chunked transfer encoding
- Content-Type negotiation (60+ MIME types)
- Custom error pages (400, 404, 405, 413, 500, 504)
- Redirections (301, 302)

### Non-Blocking I/O

- **Event-driven architecture**: Single-threaded reactor pattern
- **Multiplexing**: `select()` monitors multiple file descriptors
- **No blocking operations**: All I/O operations are non-blocking
- **Timeout handling**: Configurable timeouts for CGI scripts

### Virtual Hosts

**Multiple servers per port**:
```nginx
server {
    listen 8080;
    server_name example.com;
    docroot ./www/html;
}

server {
    listen 8080;
    server_name api.example.com;
    docroot ./www/api;
}
```

**Request routing**: Host header determines which server configuration to use

### CGI Execution

**Python CGI support**:
- Environment variable passing (REQUEST_METHOD, QUERY_STRING, CONTENT_LENGTH, etc.)
- stdin/stdout piping for request/response body
- Timeout protection (504 Gateway Timeout)
- Error handling (500 Internal Server Error)

**CGI Scripts** (`www/cgi-bin/`):
- `getcgi.py`: Displays GET parameters
- `postcgi.py`: Processes POST data
- `error.py`: Triggers CGI errors
- `infinite.py`: Tests timeout mechanism

### File Uploads

**Multipart form data handling**:
- Configurable upload directory per location
- File size limit (client_max_body_size)
- Boundary parsing
- Upload enable/disable per location

**Configuration**:
```nginx
location /upload {
    method POST;
    upload_enable on;
    upload_path ./www/uploads;
}
```

### Configuration System

**nginx-style configuration**:
- Server blocks for virtual hosts
- Location blocks for path-specific rules
- Directive validation and error checking
- Support for multiple listen ports

**Directives**:
- `listen`: Port to bind
- `server_name`: Virtual host name
- `host`: Bind address (IP or localhost)
- `error_page`: Custom error page paths
- `client_max_body_size`: Max request body (KB)
- `docroot`: Document root directory
- `index`: Default index file
- `location`: Path-specific configuration
- `method`: Allowed HTTP methods
- `upload_enable`: Enable file uploads
- `upload_path`: Upload directory
- `autoindex`: Directory listing
- `redirection`: HTTP redirect

---

## ⚙️ Configuration

### Configuration File Format

<details>
<summary><b>Example httpd.conf</b></summary>

```nginx
server {
    listen 8080 8081;
    server_name webserv_main;
    host localhost;
    error_page 404 ./www/error/404.html;
    error_page 500 ./www/error/500.html;
    client_max_body_size 100;
    docroot ./www/html;
    index index.html;

    location / {
        method GET POST;
        autoindex off;
        docroot ./www/html;
        index index.html;
    }

    location /upload {
        method GET POST DELETE;
        upload_enable on;
        upload_path ./www/html/uploader;
        docroot ./www/html/uploader;
        autoindex on;
    }

    location .py {
        method GET POST;
        docroot ./www/cgi-bin;
    }
}

server {
    listen 9090;
    server_name server2;
    host localhost;
    docroot ./www/server2;
    index server2_index.html;

    location / {
        method GET;
        docroot ./www/server2;
        index server2_index.html;
    }
}
```

</details>

### Configuration Parsing

**Parse.cpp** (18,588 chars):
- Reads configuration file line by line
- Validates server block syntax and scope
- Parses listen ports (reserved port check: 1024-65534)
- Validates IP addresses (manual parsing, Big-Endian)
- Checks error page paths exist
- Validates location paths and methods
- Inherits global directives into locations
- Builds `ServerConfig` and `LocationConfig` objects

**Validation**:
- Server name uniqueness check
- Port duplication prevention
- Path existence verification
- Method validation (GET, POST, DELETE)
- Boolean flags (on/off)

---

## 📡 HTTP Methods

### GET - Retrieve Resources

**Static Files**:
- Serves files from docroot directory
- Content-Type header based on file extension (MIMETypes)
- Supports directory listings (autoindex on)
- Returns 404 if file not found

**CGI Scripts**:
- Executes Python script with environment variables
- QUERY_STRING from URL parameters
- Returns CGI output as response body

**Example**:
```http
GET /index.html HTTP/1.1
Host: localhost:8080

HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 397

<!DOCTYPE html>
<html>...
```

### POST - Send Data

**CGI Scripts**:
- Request body passed to CGI via stdin
- CONTENT_LENGTH environment variable
- CONTENT_TYPE environment variable

**File Uploads**:
- Multipart form data parsing
- Boundary detection and extraction
- File saved to upload_path directory

**Example**:
```http
POST /cgi-bin/postcgi.py HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

name=John&message=Hello

HTTP/1.1 200 OK
Content-Type: text/html
...
```

### DELETE - Remove Resources

**File Deletion**:
- Deletes file from server filesystem
- Returns 200 OK on success
- Returns 404 if file not found
- Returns 403 if permission denied

**Example**:
```http
DELETE /uploads/file.txt HTTP/1.1
Host: localhost:8080

HTTP/1.1 200 OK
Content-Length: 0
```

---

## 🐍 CGI Support

### CGI Environment Variables

**Passed to Python scripts**:

| Variable | Description | Example |
|----------|-------------|---------|
| `REQUEST_METHOD` | HTTP method | GET, POST, DELETE |
| `QUERY_STRING` | URL query parameters | name=John&age=30 |
| `CONTENT_LENGTH` | Request body size | 1024 |
| `CONTENT_TYPE` | Request content type | application/x-www-form-urlencoded |
| `PATH_INFO` | Path after script name | /extra/path |
| `SCRIPT_NAME` | Script filename | getcgi.py |
| `SERVER_NAME` | Server name | webserv_main |
| `SERVER_PORT` | Server port | 8080 |
| `SERVER_PROTOCOL` | HTTP version | HTTP/1.1 |

### CGI Execution Flow

1. **Request arrives** for `.py` location
2. **Fork process** and set up pipes (stdin/stdout)
3. **Set environment variables** from HTTP request
4. **Execute Python script** with `execve()`
5. **Write request body** to child stdin
6. **Read response** from child stdout
7. **Parse CGI output** (headers + body)
8. **Kill child** if timeout exceeded (504)
9. **Return response** to client

<details>
<summary><b>CGIHTTPRequest Implementation</b></summary>

**CGIHTTPRequest.cpp** (9,447 chars):
- Forks child process for Python interpreter
- Sets up pipes: parent → child (stdin), child → parent (stdout)
- Passes environment variables via `char *envp[]`
- Non-blocking read from child output
- Timeout detection (default: 3 seconds)
- Signal handling for zombie process cleanup
- Parse CGI headers (Content-Type, Status, etc.)
- Error handling: 500 for exec errors, 504 for timeouts

**Timeout Protection**:
```cpp
// Check if CGI script exceeded timeout
if (currentTime - startTime > 3) {  // 3 seconds
    kill(childPid, SIGKILL);
    waitpid(childPid, NULL, 0);
    return HTTPResponse504();  // Gateway Timeout
}
```

</details>

---

## ⚠️ Error Handling

### HTTP Error Responses

The server implements **6 custom error responses**:

| Code | Name | Handler | Trigger |
|------|------|---------|---------|
| **400** | Bad Request | `HTTPResponse400` | Malformed HTTP request |
| **404** | Not Found | `HTTPResponse404` | File doesn't exist |
| **405** | Method Not Allowed | `HTTPResponse405` | Method not in location config |
| **413** | Payload Too Large | `HTTPResponse413` | Body exceeds client_max_body_size |
| **500** | Internal Server Error | `HTTPResponse500` | CGI execution error |
| **504** | Gateway Timeout | `HTTPResponse504` | CGI script timeout |

### Custom Error Pages

**Configuration**:
```nginx
error_page 404 ./www/error/404.html;
error_page 500 ./www/error/500.html;
```

**Error Page Structure** (`www/error/`):
- Simple HTML pages with consistent styling
- Dark theme with centered content
- Error code and descriptive message
- `styles.css` for uniform appearance

### Error Request Factory

**ErrorHTTPRequest.cpp**:
- Reads custom error page from filesystem
- Falls back to default error message if file missing
- Sets appropriate Content-Type (text/html)
- Returns HTTPResponse with error body

---

## 🛠️ Compilation

### Build System

**Makefile** structure:
```makefile
NAME = webserv
CXX = c++
CXXFLAGS = -Wall -Wextra -Werror -std=c++98 -Iinc

SRCDIR = src
INCDIR = inc

SRCS = main.cpp \
       $(SRCDIR)/Reactor.cpp \
       $(SRCDIR)/EventHandler.cpp \
       $(SRCDIR)/AcceptConnectionEventHandler.cpp \
       $(SRCDIR)/ServeRequestEventHandler.cpp \
       ... (31 source files)

OBJS = $(SRCS:.cpp=.o)

all: $(NAME)

$(NAME): $(OBJS)
	$(CXX) $(CXXFLAGS) $(OBJS) -o $(NAME)

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS)

fclean: clean
	rm -f $(NAME)

re: fclean all

.PHONY: all clean fclean re
```

**Compilation**:
```bash
make        # Compile server
make clean  # Remove object files
make fclean # Remove executable
make re     # Recompile from scratch
```

**Compiler flags**:
- `-Wall -Wextra -Werror`: All warnings as errors
- `-std=c++98`: Enforce C++98 standard
- `-Iinc`: Include directory for headers

---

## 🚀 Usage

### Starting the Server

```bash
# Compile
make

# Run with configuration file
./webserv httpd.conf
```

**Server Output**:
```
Registering event (fd = 3) AcceptConnectionEventHandler
Registering event (fd = 4) AcceptConnectionEventHandler
Registering event (fd = 5) AcceptConnectionEventHandler
Server listening on ports: 8080, 8081, 9090
Press Ctrl+C to stop
```

### Testing with curl

**Static file**:
```bash
curl http://localhost:8080/
curl http://localhost:8080/index.html
```

**CGI script (GET)**:
```bash
curl "http://localhost:8080/cgi-bin/getcgi.py?name=John&age=30"
```

**CGI script (POST)**:
```bash
curl -X POST -d "name=John&message=Hello" \
     http://localhost:8080/cgi-bin/postcgi.py
```

**File upload**:
```bash
curl -X POST -F "file=@test.txt" \
     http://localhost:8080/upload
```

**File deletion**:
```bash
curl -X DELETE http://localhost:8080/upload/test.txt
```

**Directory listing** (autoindex on):
```bash
curl http://localhost:8080/upload/
```

### Testing with Browser

1. Open browser: `http://localhost:8080`
2. Navigate to upload page: `http://localhost:8080/uploader`
3. Test CGI: `http://localhost:8080/cgi-bin/getcgi.py?test=value`
4. Trigger errors:
   - 404: `http://localhost:8080/nonexistent.html`
   - 405: `DELETE` on non-upload location
   - 413: Upload file larger than client_max_body_size

---

## 🧪 Testing

### Test Suite Structure

**www/cgi-bin/** test scripts:

| Script | Purpose | Test Case |
|--------|---------|-----------|
| `getcgi.py` | Display GET parameters | QUERY_STRING parsing |
| `postcgi.py` | Process POST data | stdin reading, form data |
| `error.py` | Trigger CGI error | 500 Internal Server Error |
| `infinite.py` | Infinite loop | 504 Gateway Timeout |

**Error page testing** (`www/error/`):
- Custom error pages for all 6 error codes
- Verify error page loading
- Test fallback to default errors

### Test Scenarios

**Connection handling**:
1. Multiple simultaneous connections
2. Persistent connections (keep-alive)
3. Connection timeout
4. Large number of rapid requests

**HTTP methods**:
1. GET static files (HTML, CSS, images)
2. GET CGI scripts with query parameters
3. POST CGI scripts with form data
4. POST file uploads (multipart/form-data)
5. DELETE files from upload directory

**Error conditions**:
1. Malformed HTTP requests → 400
2. Non-existent files → 404
3. Method not allowed in location → 405
4. Request body too large → 413
5. CGI script error → 500
6. CGI script timeout → 504

**Configuration**:
1. Multiple listen ports
2. Multiple server blocks (virtual hosts)
3. Host header routing
4. Location matching (exact, prefix, extension)
5. Method restrictions per location
6. Upload enable/disable per location

**CGI execution**:
1. Environment variable passing
2. stdin/stdout piping
3. Timeout detection and process killing
4. Error output handling

### Stress Testing

**Siege** (load testing):
```bash
# Install siege
sudo apt-get install siege

# Test server performance
siege -c 100 -r 10 http://localhost:8080/

# Test specific endpoint
siege -c 50 -t 1M http://localhost:8080/cgi-bin/getcgi.py
```

**Expected Results**:
- No crashes or memory leaks
- Graceful handling of connection limits
- Consistent response times under load

---

## 🔬 Technical Challenges

### Challenge 1: Non-Blocking I/O State Machine

**Problem**: HTTP requests may arrive in multiple chunks. Must parse incrementally without blocking.

**Solution**: `HTTPRequestStatus` state machine tracks parsing progress:
- `HEADERS_PENDING`: Reading headers line by line
- `BODY_PENDING`: Reading body (Content-Length or chunked)
- `COMPLETE`: Request fully parsed
- `ERROR`: Malformed request detected

**Implementation**: Each `read()` call processes available bytes, updates state, returns control to reactor.

### Challenge 2: CGI Timeout Without Threads

**Problem**: CGI script may hang indefinitely. Must kill after timeout without blocking main thread.

**Solution**: Non-blocking `waitpid()` with timestamp comparison:
```cpp
pid_t result = waitpid(childPid, &status, WNOHANG);  // Non-blocking
if (result == 0) {  // Still running
    if (time(NULL) - startTime > 3) {  // Timeout
        kill(childPid, SIGKILL);
    }
}
```

### Challenge 3: Virtual Host Routing

**Problem**: Multiple server blocks on same port. Must route based on `Host` header.

**Solution**: 
1. Parse `Host` header from request
2. Match against `server_name` in each `ServerConfig`
3. If no match, use first server on that port (default)
4. Pass matched `ServerConfig` to request handler

### Challenge 4: Configuration File Parsing

**Problem**: nginx-style config with nested blocks and validation.

**Solution**: **Parse.cpp** recursive descent parser:
- Split file into server blocks (scope tracking with `{` and `}`)
- Parse each server block line by line
- Extract location blocks within server
- Validate directive syntax and values
- Build `ServerConfig` with nested `LocationConfig` vector
- Check for duplicates (ports, server names)

### Challenge 5: Multipart Form Data Parsing

**Problem**: File uploads use multipart/form-data with boundaries.

**Solution**: **UploadFileRequest.cpp**:
- Extract boundary from `Content-Type` header
- Split body by boundary markers
- Parse each part headers (Content-Disposition, filename)
- Extract file content between headers and boundary
- Write to upload_path directory

---

## 📚 Resources

### HTTP Protocol

**RFC 2616** - HTTP/1.1 specification:
- [RFC 2616 - HTTP/1.1](https://tools.ietf.org/html/rfc2616)
- [HTTP Request Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [HTTP Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

**CGI Specification**:
- [RFC 3875 - CGI 1.1](https://tools.ietf.org/html/rfc3875)
- [CGI Environment Variables](https://en.wikipedia.org/wiki/Common_Gateway_Interface#Environment_variables)

### Design Patterns

**Reactor Pattern**:
- [Reactor Pattern (Douglas Schmidt)](http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf)
- [Reactor Pattern Explained](https://en.wikipedia.org/wiki/Reactor_pattern)
- [Event-Driven Architecture](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch02.html)

**Factory Pattern**:
- [HTTPRequestFactory implementation](https://refactoring.guru/design-patterns/factory-method)

### Network Programming

**Socket Programming**:
- [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/html/)
- [socket(7) - Linux man page](https://man7.org/linux/man-pages/man7/socket.7.html)
- [select(2) - Linux man page](https://man7.org/linux/man-pages/man2/select.2.html)

**Non-Blocking I/O**:
- [Non-blocking I/O with select/poll](https://www.scottklement.com/rpg/socktut/nonblocking.html)
- [fcntl(2) - Linux man page](https://man7.org/linux/man-pages/man2/fcntl.2.html)

### Configuration Parsing

**nginx Configuration**:
- [nginx Configuration](https://nginx.org/en/docs/beginners_guide.html)
- [Understanding nginx Configuration Contexts](https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts)

### Testing Tools

**HTTP Testing**:
- [curl](https://curl.se/docs/manual.html) - Command-line HTTP client
- [Postman](https://www.postman.com/) - API testing platform
- [siege](https://github.com/JoeDog/siege) - Load testing tool

**Browser DevTools**:
- Network tab for request/response inspection
- Console for debugging

---

## 💡 What We Learned

**Networking Fundamentals**:
- ✅ TCP/IP socket programming (bind, listen, accept, recv, send)
- ✅ HTTP/1.1 protocol implementation (methods, headers, status codes)
- ✅ Non-blocking I/O with `select()` multiplexing
- ✅ File descriptor management and resource cleanup

**Software Architecture**:
- ✅ Reactor pattern for event-driven I/O
- ✅ Factory pattern for request handler creation
- ✅ State machine for incremental HTTP parsing
- ✅ Separation of concerns (config, parsing, handling, response)

**System Programming**:
- ✅ Process forking and execution (`fork()`, `execve()`)
- ✅ Inter-process communication (pipes)
- ✅ Signal handling (SIGCHLD, SIGPIPE)
- ✅ File system operations (open, read, write, stat)

**Configuration & Parsing**:
- ✅ Custom configuration file format (nginx-style)
- ✅ Recursive descent parsing with validation
- ✅ Scope tracking and nested block handling
- ✅ Error reporting and exception handling

**HTTP Features**:
- ✅ Static file serving with MIME type detection
- ✅ CGI script execution with environment passing
- ✅ Multipart form data parsing for file uploads
- ✅ Custom error pages and redirections
- ✅ Virtual host routing based on Host header

**Team Collaboration**:
- ✅ Git workflow with feature branches and code reviews
- ✅ Task distribution and integration planning
- ✅ Debugging complex multi-component systems
- ✅ Documentation and code maintainability

**C++98 Constraints**:
- ✅ No C++11 features (auto, nullptr, lambdas, move semantics)
- ✅ STL containers only (vector, map, string)
- ✅ Manual memory management (new/delete)
- ✅ Function pointers instead of std::function

---

## 🎯 Key Takeaways

### For System Programming

1. **Non-blocking I/O is essential** for scalable servers
2. **Event-driven architecture** eliminates threading complexity
3. **File descriptor management** requires careful tracking and cleanup
4. **Timeout handling** without threads needs timestamp comparison
5. **Process forking** for CGI is simpler than threading

### For HTTP Implementation

1. **Incremental parsing** handles partial requests
2. **State machines** simplify complex parsing logic
3. **MIME types** are essential for proper Content-Type headers
4. **Error handling** must be comprehensive (6+ error codes)
5. **Virtual hosts** enable multiple sites per IP/port

### For Configuration

1. **nginx-style config** is human-readable and powerful
2. **Validation** catches errors early (ports, paths, methods)
3. **Inheritance** (global → location) reduces duplication
4. **Scope tracking** ensures correct nesting

### For Team Projects

1. **Clear interfaces** between components enable parallel work
2. **Regular integration** catches conflicts early
3. **Code reviews** improve quality and knowledge sharing
4. **Shared debugging** sessions solve complex issues faster
5. **Documentation** is critical for team understanding

---

## 🔗 Related Projects

**Prerequisites**:
- **NetPractice**: TCP/IP networking fundamentals
- **ft_irc**: Socket programming and protocols

**Related**:
- **ft_transcendence**: Full-stack web application
- **inception**: Docker and nginx configuration

**Skills Apply To**:
- Backend web development (Node.js, Python, Go)
- Microservices architecture
- Load balancers and reverse proxies
- Real-time systems (chat, games)
- DevOps and infrastructure management

---

<div align="center">

**Made with ☕ by [Iñigo Gonzalez](https://github.com/Z3n42), [Manu Garcia](https://github.com/manu-garcia), and [Mikel Garcia](https://github.com/mikgarci42)**

*42 Urduliz | Circle 05*

[![42 Profile - ingonzal](https://img.shields.io/badge/42_Intra-ingonzal-000000?style=flat&logo=42&logoColor=white)](https://profile.intra.42.fr/users/ingonzal)
[![GitHub - manugarc](https://img.shields.io/badge/GitHub-manugarc-181717?style=flat&logo=github)](https://github.com/manu-garcia)
[![GitHub - mikgarci](https://img.shields.io/badge/GitHub-mikgarci-181717?style=flat&logo=github)](https://github.com/mikgarci42)

</div>
