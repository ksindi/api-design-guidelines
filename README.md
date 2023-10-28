# **Guidelines for API Design and Performance**

## **Design Principles**

### **Understanding HTTP Methods**

1. Clearly distinguish between POST, PATCH, and PUT to ensure proper usage and behavior in API interactions.

### **Endpoint Naming Conventions**

1. Adhere to RESTful naming conventions for external endpoints. Utilize nouns (e.g., **`accounts`**) rather than verbs (e.g., **`createAccount`**) for endpoint names.
    - Since HTTP methods (GET, POST, PUT, etc.) imply action, additional verbs in the endpoint URL are redundant.
2. Use plural nouns (e.g., **`accounts`**) for endpoints to intuitively indicate collections or groups of resources.
3. Implement nested endpoints to denote relationships, like **`api.foo.com/v1/accounts/{account_id}/users`**.
4. Avoid leading numerals in field names for broader language compatibility.

### **Response Strategy**

1. Employ standard HTTP status codes for feedback (2xx for success, 4xx for client errors, 5xx for server errors).
2. Ensure error messages are informative, guiding users on how to rectify issues.
3. Include a **`debugging_id`** in responses to assist in troubleshooting.
4. Maintain a consistent response schema, defaulting unpopulated fields to **`null`**.

### **API Documentation**

1. Use OpenAPI for API specification documentation, ensuring continuous validation against actual API requests.

### **Development Environments**

1. Prioritize the provision of sandbox environments for integration, mirroring production API behavior as closely as possible.

### **Error Reporting**

1. Make error responses meaningful and constructive, offering debugging tips or workarounds where possible.

### **Naming and Formatting Standards**

1. Use UPPER_SNAKE_CASE for enums and lower_snake_case for field names.
2. Adopt UTC for timezones, except for future dates.
3. Include **`created`** and **`updated`** timestamps in all responses.
4. Prefer UUIDv7 for resource identifiers, presenting them in Base62 encoding to users, prefixed with a resource identifier (e.g. `vid_HLlvJYKurU6taLvIyYiYWX6OndSzSymw`). Great article on why: https://buildkite.com/blog/goodbye-integers-hello-uuids.

### **Data Handling**

1. Ensure HTTP bodies are valid JSON and headers specify **`application/json`**.
2. Implement idempotency in POST requests using an **`Idempotency-Key`** header.
3. Show all fields in responses, even if data is absent (marked as **`null`**).
4. Standardize date formatting to RFC 3339.
5. Implement cursor-based pagination in API responses, ensuring chronological order and a standard pagination wrapper.
6. Utilize pre-signed URLs for file uploads, ensuring file validation.
7. Expose and request only essential data to minimize API surface and simplify changes.

## **Performance and Observability**

### **Efficiency and Monitoring**

1. Aim for sub-250ms p99 latency.
2. Log and monitor metrics (request counts, response times) at various system layers, redacting sensitive information.
3. Use distributed tracing to identify system bottlenecks.
4. Opt for OLAP/Snowflake solutions only for predictable query loads.

### **Scalability and Testing**

1. Design database models to accommodate anticipated query patterns and scalability.
2. Regularly load test systems to identify performance issues.
3. Use job IDs for long-running queries.
4. Set field data limits and consider rate limiting to protect system integrity.

## **Security**

### **Protecting Sensitive Data**

1. Avoid logging personal identifiable information (PII).
2. Restrict public access to admin API endpoints, isolating them in separate services.
3. Sanitize inputs to prevent XSS attacks.
4. Establish a bug bounty program for additional security oversight.

## **Cursor-Based Pagination Details**

### **Implementation**

1. Control pagination using **`limit`**, **`start_after`**, and **`ending_before`** parameters.
2. Default to 50 and cap page sizes at 100.
3. Calculate **`has_more`** by querying for one extra item beyond the page size.

### **Example Usage**

- Assuming rows A, B, C, D, E (ascending order):
    1. **`limit=2`** returns **`E, D`**.
    2. **`starting_after=D&limit=2`** returns **`C, B`**.
    3. **`ending_before=D&limit=2`** returns **`E`**.
    4. **`ending_before=A&limit=2`** returns **`C, B`**.
