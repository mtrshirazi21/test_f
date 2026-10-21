Front-End Engineer Assignment: IDICorp Service Console
Handoff from the backend team
Hi — the service is ready for a browser client. We need an internal console where a signed-in user can work with sample tasks and try a few service capabilities: a calculator, a text echo, and a streamed echo. Please build the client against the API described below. The API base URL will be provided with your environment. For the local mock, the username and password default to admin and admin; an environment may provide different test credentials.

The goal is to make these flows clear and dependable for a user. We have not prescribed a framework, visual design, or application structure; choose an approach that suits the work.

Connecting to the service
Sign-in
Method	Path	What it does
GET	/api/auth/login	Returns the CAPTCHA image. Read the X-Captcha-Salt response header and send its value with the sign-in request.
POST	/api/auth/login	Accepts JSON fields username, password, salt, and captcha_answer. On success, returns access_token and token_type, and sets an HTTP-only session cookie.
Protected HTTP requests need the access token as a Bearer token and the session cookie. The browser must include credentials when making those requests. A failed or missing session returns 401.

Tasks
All task routes require a signed-in session.

Method	Path	What it does
GET	/api/tasks	Returns the sample tasks. Optional completed=true or completed=false filters by completion; optional q searches titles and descriptions without case sensitivity.
GET	/api/tasks/{id}	Returns a sample task by ID, or 404 if that ID is not in the sample set.
POST	/api/tasks	Accepts title, optional description, and optional completed; returns 201 with a task-shaped response.
PUT	/api/tasks/{id}	Replaces a task. Requires title, description, and completed; returns the updated task-shaped response for a positive ID.
PATCH	/api/tasks/{id}	Updates one or more of title, description, and completed; returns the task-shaped response for a positive ID.
DELETE	/api/tasks/{id}	Returns 204 for a positive ID.
A task response contains id, title, description, completed, created_at, and updated_at. Titles can be up to 120 characters and descriptions up to 2,000 characters. Invalid input returns 422; requests may return 429 when rate-limited.

Task changes are simulated: a successful write response acknowledges that request, but does not change the sample collection returned by later reads.

WebSocket authentication
The two WebSocket endpoints use different authentication flows.

For /ws/calculator, open a standard WebSocket connection and send the access token as the first JSON message within one second:

{"token":"<access_token>"}
If the token is valid, the service replies with {"type":"ready"}. If the token is invalid, the message is malformed, or no message arrives within one second, it sends {"error":"Unauthorized"} and closes the connection.

For /ws/echo, authenticate during the opening HTTP handshake. Include the access token and the session cookie from sign-in in the handshake headers:

Authorization: Bearer <access_token>
Cookie: session=<session_cookie>
The service rejects unauthenticated handshakes. After a successful connection, it sends {"type":"ready"}.

Calculator over WebSocket
Connect to /ws/calculator on the API host, using wss for a secure API host and ws for a local one. After the ready message, send calculator requests with an operation and a list of numbers. The supported operations are add, subtract, multiply, and divide:

{"operation":"add","numbers":[10,20]}
A valid request returns {"result": <number>}. An invalid request returns {"error":"Invalid calculator request"} and leaves the connection open.

Text echo over WebSocket
Connect to /ws/echo on the API host. After the ready message, send text; the service echoes each text message on the same connection. The connection can be used for multiple messages.

HTTP streaming echo
POST /api/echo/stream accepts an authenticated request body and streams the same bytes back. The response uses the request's content type when provided, or application/octet-stream otherwise.

What we need from the client
A user can sign in using the CAPTCHA and credentials supplied for the environment.
A user can browse and search tasks, filter by completion, and try creating, editing, and deleting tasks.
A user can use each calculator operation and understand the result or an invalid request.
A user can send multiple messages through the text echo connection and see its connection state.
A user can send content through the streaming echo service and see the returned content.
Loading, empty, validation, authentication, network, and service errors are communicated clearly.
The client works on common screen sizes and can be operated with a keyboard, with clear labels and visible focus states.
The API's interactive reference is available at /docs on the API host. Account for the service's actual responses and the fact that task writes are not persisted.

Submission
Submit a runnable, standalone front-end application with concise setup instructions. It must run without being integrated into the backend code. The browser must be able to load the application's assets directly from the front-end app or its static host; do not rely on the backend to serve or proxy those assets. Include the API base URL configuration needed to run the client and note any assumptions you made about the service.
