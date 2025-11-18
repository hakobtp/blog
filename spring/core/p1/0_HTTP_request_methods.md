# HTTP request methods and Status Codes

This blog post describes the most commonly used HTTP methods and their purposes when interacting with RESTful APIs.

**GET** `/api/v1/customers`
- **Description:** Retrieves data from the server. It should only fetch data and must not modify any resources.
- **Use case:** Return all customers.

**POST** `/api/v1/customers`
- **Description:** Sends data to the server to create a new resource. The data is included in the body of the request.
- **Use case:** Create a new customer.

**PUT** `/api/v1/customers/1223`
- **Description:** Used to replace an existing resource entirely with new data.
- **Use case:** Update an existing customer by ID.

**PATCH** `/api/v1/customers/1223`
- **Description:** Used to partially update an existing resource — only the specified fields are modified.
- **Use case:** Update specific fields (e.g., phone number or address) of a customer.

**DELETE** `/api/v1/customers/1223`
- **Description:** Removes the specified resource from the server.
- **Use case:** Delete a customer by ID.

**HEAD** `/api/v1/customers`
- **Description:** Similar to GET, but returns only the response headers without the response body.
Used to check metadata such as content type or length.
**Use case:** Check what a GET request would return before actually fetching the data.

**OPTIONS** `/api/v1/customers`
- **Description:** Returns the supported HTTP methods and operations for a specific resource.
Commonly used in CORS preflight requests.
- **Use case:** Determine which methods (GET, POST, PUT, etc.) are allowed.

**TRACE** `/api/v1/customers`
- **Description:** Performs a message loop-back test for diagnostic purposes.
It echoes the received request, helping developers see any modifications made by intermediate servers.
- **Use case:** Debug HTTP request routing.

**CONNECT** `/api/v1/customers`
- **Description:** Used to establish a network tunnel, typically for SSL (HTTPS) connections through an HTTP proxy.
- **Use case:** Create a secure connection between the client and the server.

### Summary Table

| **Method**  | **Endpoint Example**     | **Purpose**           | **Description**                                                     |
| ----------- | ------------------------ | --------------------- | ------------------------------------------------------------------- |
| **GET**     | `/api/v1/customers`      | Retrieve data         | Fetches data from the server without making any changes.            |
| **POST**    | `/api/v1/customers`      | Create data           | Sends data to the server to create a new resource.                  |
| **PUT**     | `/api/v1/customers/1223` | Replace data          | Replaces an existing resource entirely with new data.               |
| **PATCH**   | `/api/v1/customers/1223` | Update partially      | Updates only specific fields of an existing resource.               |
| **DELETE**  | `/api/v1/customers/1223` | Delete data           | Removes the specified resource from the server.                     |
| **HEAD**    | `/api/v1/customers`      | Retrieve headers      | Returns only response headers (no body). Used for metadata checks.  |
| **OPTIONS** | `/api/v1/customers`      | Discover capabilities | Returns allowed HTTP methods and operations. Commonly used in CORS. |
| **TRACE**   | `/api/v1/customers`      | Diagnostic loop-back  | Echoes the received request to test message routing.                |
| **CONNECT** | `/api/v1/customers`      | Establish tunnel      | Creates a network tunnel (often for HTTPS connections).             |


# HTTP Status Codes

HTTP status codes indicate whether a specific HTTP request has been successfully completed.
They are grouped into five categories, each representing a different type of response.

### 2xx – Success Codes

| **Code**           | **Message**      | **Description**                                                                   |
| ------------------ | ---------------- | --------------------------------------------------------------------------------- |
| **200 OK**         | Success          | The request succeeded. The result depends on the method used (GET, POST, etc.).   |
| **201 Created**    | Resource Created | A new resource has been successfully created.                                     |
| **202 Accepted**   | Request Accepted | The request has been accepted for processing, but the processing is not complete. |
| **204 No Content** | No Response Body | The request succeeded, but there is no content to send in the response.           |

### 4xx – Client Error Codes

| **Code**                     | **Message**             | **Description**                                                                        |
| ---------------------------- | ----------------------- | -------------------------------------------------------------------------------------- |
| **400 Bad Request**          | Invalid Request         | The server cannot process the request due to invalid syntax or parameters.             |
| **401 Unauthorized**         | Authentication Required | The client must authenticate itself to get the requested response.                     |
| **403 Forbidden**            | Access Denied           | The client does not have permission to access the requested resource.                  |
| **404 Not Found**            | Resource Not Found      | The server cannot find the requested resource.                                         |
| **405 Method Not Allowed**   | Invalid HTTP Method     | The method used is not allowed for the requested resource.                             |
| **409 Conflict**             | Resource Conflict       | The request conflicts with the current state of the resource.                          |
| **422 Unprocessable Entity** | Validation Failed       | The server understands the content but cannot process it (e.g., invalid field values). |


### 5xx – Server Error Codes

| **Code**                      | **Message**               | **Description**                                                       |
| ----------------------------- | ------------------------- | --------------------------------------------------------------------- |
| **500 Internal Server Error** | General Server Error      | A generic error occurred on the server.                               |
| **501 Not Implemented**       | Not Supported             | The server does not recognize or support the request method.          |
| **502 Bad Gateway**           | Invalid Response          | The server received an invalid response from an upstream server.      |
| **503 Service Unavailable**   | Server Down or Overloaded | The server is temporarily unable to handle the request.               |
| **504 Gateway Timeout**       | Response Timeout          | The server did not receive a timely response from an upstream server. |


### Tips for API Design

- Always return meaningful status codes that match the operation result.
- Include an error message or code in JSON format for clarity.
- Use 201 Created for successful POST requests.
- Use 204 No Content for successful DELETE or PUT requests when no body is returned.
- Use 400+ codes to indicate client-side mistakes, not server issues.


### Summary Table

| **Category** | **Code Range** | **Meaning**                                                                |
| ------------ | -------------- | -------------------------------------------------------------------------- |
| **1xx**      | 100–199        | Informational — Request received, continuing process.                      |
| **2xx**      | 200–299        | Success — The request was successfully received, understood, and accepted. |
| **3xx**      | 300–399        | Redirection — Further action must be taken to complete the request.        |
| **4xx**      | 400–499        | Client Error — The request contains bad syntax or cannot be fulfilled.     |
| **5xx**      | 500–599        | Server Error — The server failed to fulfill a valid request.               |

---

- [Home](./../../../README.md)
- [Spring Tutorials](./../../tutorials.md)
- [Spring Core Tutorials](./../core.md)
- [Spring Boot Annotations](./1_Spring_Boot_Annotations.md)

