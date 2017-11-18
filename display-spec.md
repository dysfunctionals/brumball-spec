# [DEFUNCT] Display Specification
---
**NOTE: This spec is now defunct, as the architecture has changed significantly, so don't expect the
display to conform to it. The current server spec is not formally defined anywhere and is pretty
much just ad-hoc.**

The meaning of **must**, **should**, **could** and other similar terms is to be as specified in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Display dimensions and objects
- The display shall be 2048 units by 2048 units in size. These units may correspond to an arbitrary number of pixels, but should be scaled such that a 5u by 5u polygon is clearly visible.
- The display shall be on a vertically oriented screen, such that the top left corner is (0, 0), the bottom left corner is (2048, 0), the bottom right corner is (2048, 2048), and the top right corner is (0, 2048).
- Bearings may be given in communications and shall be defined as a bearing clockwise from the up direction.
- The display shall contain between 2 and 8 paddles, and may support an arbitrary number of paddles.
- The display shall contain at least one ball, must be able to support at least 4 balls, and may support an arbitrary number of balls.
- The display shall support the creation of an arbitrary number of arbitrarily sized rectangles within the confines of the display.
- The display may support the creation of an arbitrary number of arbitrarily sized regular polygons, with an arbitrary number of sides, within the confines of the display.
- The behaviour of the display in a region where two objects overlap is not specified.

## Communication with server
- The display shall receive communications from the server over a socket in the form of JSON strings (compliant with [RFC 4627](https://www.ietf.org/rfc/rfc4627.txt)), in a UTF-8 encoded form.
    - Communication shall be carried out via TCP port 1729.
    - When the server intends to transmit a new communication, it shall send a 16-bit unsinged big-endian integer representing the length of the communication in bits.
    - Following the length of the communication, the server shall then send the string containing the JSON communication.
    - The server may send multiple JSON objects or a single JSON object in each communication, the display must correctly receive either.
- The display shall not send communications to the server.

## Object instantiation
- New objects are instantiated via JSON objects with fields for each attribute of the object. All objects shall have a "type" field which will be either "paddle", "ball", "rectangle" or "polygon". All objects shall also have an "id" field which will be a string that uniquely identifies each object.
    - The **paddle** will have a "height" which is its height in units, "width" which is its width in units, "bearing" which is its rotation in degrees, and "colour" which is its colour in a hex triplet form.
    - The **ball** will have a "diameter" which is both its height and width in units, and "colour" which is its colour in a hex triplet form.
    - The **rectangle** will have a "height" which is its height in units, "width" which is its width in units, and "colour" which is its colour in a hex triplet form.
    - The **polygon** will have a "height" which is its height in units, "width" which is its width in units, and "colour" which is its colour in a hex triplet form.

## Object removal
- Display objects shall be deleted upon receiving a JSON object with a field "command" of value "remove". This object shall also have an "id" field, which designates the identifier of the object that shall be removed.
- Removal of the object means it must immediately cease being rendered on the display.
- Unique identifiers must be freed upon their removal, such that the display continues to function correctly if a new object with the same identifier (which may or may not be of the same type as the previous object) is later created.

## Object display and movement
- Objects must not be displayed if their location has not been communicated.
- If an object's bearing has never been communicated, it must be assumed to be zero.
- Objects at zero bearing have their height dimension in the vertical direction and width dimension in the horizontal direction.
- Communications will be sent as JSON objects with a "command" field of value "update"
    - An update may have fields of type "v" which represents the object's vertical position in units, "h" which represents the object's horizontal position in units, "bearing" which represents the object's bearing in degrees, "colour" which represents the object's colour as a hex triplet.
    - An update may contain any number of these fields from one to all, and the display must correctly process all these cases.
- If an update occurs and one of the object's fields is not specified, it must be assumed to have the value it had prior to the update.
- The location specified for an object must be where the centre of the object is located.
- If an object is partially outside the display but its centre is within the bounds of the display, the parts within the display must be correctly displayed and any part beyond the display must not be displayed.
- If an object is partially inside the display but its centre is outside the display, it may have (only) the parts within the display rendered. Any part outside the display must not be rendered.
- The display may interpolate the motion of objects for a smoother viewing experience. Objects should move in straight lines, so interpolation should be done in a way which assumes objecs travel in straight lines, and should cease or restart interpolation if the direction of motion changes unexpectedly.
