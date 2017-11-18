# HTTP Client Spec
---
## Connection
- Client sends GET request to /connect
- We return JSON containing:
    - "token": a unique string identifier
    - "colour": team colour as a hex triplet
    - "rotation": bearing in degrees
## State Change
- Client sends POST request to /inform containing:
    - "token": the client's unquie string identifier passed during the connection.
    - "motion": containing True for clockwise and False for anticlockwise.
- We return JSON containing:
    - "valid": true if the token and data was accepted, false otherwise.
- If "valid" was false, client reconnection is required.
## Heartbeat
- Every 7 seconds client sends POST reqest to /alive containing:
    - "token": the client's unique string identifier passed during the connection.
- We return JSON containing:
    - "valid": true if the token is still valid, false otherwise.
- If "valid" was false, client reconnection is required.
## Admin
- We may implement an admin panel, but this will come later if we have time

## Notes
- When the client reconnects, it might be on a different team. Please do something to inform the user of this if they suffer a disconnect.
