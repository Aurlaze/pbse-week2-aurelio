# Week 2 Notes & Checkpoints

## Reading Checkpoints

### Checkpoint 1 
- **Screen-shaped address:** `GET /dashboard`
- **Real resources:** `GET /courts?isAvailable=true` available courts and `GET /bookings?userId=...` user's bookings.

### Checkpoint 2 
- **Rule 5 Address:** `POST /bookings/{id}/cancellation`
- **Verb version:** `POST /cancelBooking`
- **Why verb is worse:** A verb URL doesn't leave a history record that you can query later, it doesn't give you a place to return a response body like a refund status, and it breaks the rule where the HTTP method is the verb and the path is the noun.

### Checkpoint 3
- **GET /courts?isAvailable=true:** If retried after a network drop, the user loses nothing. Hence, it is safe and idempotent where reading changes no server state.
- **GET /bookings?status=confirmed:** If retried after a network drop, the user loses nothing. Hence, it is safe and idempotent.
- **POST /bookings (the creation action):** If retried after a network drop without protection, the user loses money and gets a duplicate court booking. This operation is unsafe and non-idempotent, so it requires the `Idempotency-Key` header.

---

## Self-Check

1. **POST /v1/orders/{id}/markReady rejection:** This is because it puts a verb directly into the URL. This means there is nowhere to attach a request or response body. This is so that someone can record who marked the order ready and at what time. The correct address to use is POST /v1/orders/{id}/readiness.
Grade Result: PASS
2. **Retried PUT /v1/menu-items/itm_3Bn:** When a client sends a full PUT /v1/menu-items/itm_3Bn representation and tries again after a timeout, the operation is not read only which is safe because it changes the state on the server. However, it is safe to repeat which is idempotent because PUT completely replaces the target resource with the payload provided. This means that the server is left in exactly the same state whether the request is processed once or multiple times.
Grade Result: PASS
3. **Sold out menu item:** If a product is out of stock, the system should return a 409 error message. This is known as a domain rejection, which means the request was valid but the business rules did not allow it. Using 500 Internal Server Error is wrong because it shows there has been an internal crash. This fills the error logs and tells clients to keep trying a request that can't be fulfilled. Using 200 OK with {"success": false} is also problematic because it hides the failure from HTTP caching, proxies and client error handlers by pretending to be a success.
Grade Result: PASS
4. **GorBooking Dangerous Operation:** In GorBooking, the dangerous operation is POST /v1/bookings because creating a reservation and taking advance payment must never happen twice. If the network drops right after the server processes the request, the user won't know what happened. This could lead to a second court booking and another money transfer. To protect against this, our file says that clients must send an Idempotency-Key header holding a standard version-4 hyphenated UUID on POST /v1/bookings and it is ignored everywhere else. The server keeps these keys for 24 hours before treating them as new. If a client tries to reuse a previous key with a different body payload, the server rejects it with a 409 Conflict error. This error points to the idempotency-key-reuse problem type.

5. **One thing I am unsure about:** I'm not sure how replay should work. Especially, when a client tries again to access a part of a website like POST /v1/bookings/{id}/cancellation on a booking that is already cancelled, I am not completely sure whether the server should respond with 200 OK using the existing record or return 201 Created again.
