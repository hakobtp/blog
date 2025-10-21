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

## HTTP Status Codes

HTTP status codes indicate whether a specific HTTP request has been successfully completed.
They are grouped into five categories, each representing a different type of response.

---

- [Home](./../../../README.md)
- [Spring Tutorials](./../../tutorials.md)
- [Spring Core Tutorials](./../core.md)
- [Spring Boot Annotations](./1_Spring_Boot_Annotations.md)

