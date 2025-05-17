# Insecure Direct Object References (IDOR)

## What is IDOR?

IDOR (Insecure Direct Object References) is a type of access control vulnerability that occurs when an application exposes direct access to objects (such as database records or files) based on user-supplied input, without properly verifying that the user is authorized to access the requested resource.

---

## Example scenarios

### 1. Accessing Another User's Data
- URL: `https://example.com/user/profile?id=123`

- **True Case**:  
  - Changing `id=123` to `id=124` allows the attacker to view another user's profile.

- **False Case**:  
  - The application checks ownership and returns an error: "You cannot access this user's data."

---

### 2. Modifying Another User's Data
- URL: `https://example.com/user/edit_profile?id=123`

- **True Case**:  
  - Changing `id=123` to `id=124` allows the attacker to edit another user's data.

- **False Case**:  
  - The application returns: "You are not authorized to modify this user's data."

---

### 3. Deleting Another User's Account
- URL: `https://example.com/admin/delete_user?id=123`

- **True Case**:  
  - Modifying `id=123` to `id=124` deletes another user's account.

- **False Case**:  
  - The application returns: "You are not authorized to delete this user's account."

---

### 4. Accessing Another User's Financial Report
- URL: `https://example.com/reports/view?user_id=123`

- **True Case**:  
  - Changing `user_id=123` to `user_id=124` exposes another user's financial data.

- **False Case**:  
  - The application prevents unauthorized access and returns: "You are not authorized to view this user's report."

---

### 5. Adding a Product to Another User's Cart
- URL: `https://example.com/cart/add_product?product_id=456&user_id=123`

- **True Case**:  
  - Modifying `user_id` to another value adds a product to another user's cart.

- **False Case**:  
  - The server returns: "You cannot add products to another user's cart."

---

### 6. Manipulating Delivery Requests
- URL: `https://example.com/order/update_delivery_address?order_id=789&address=NewAddress`

- **True Case**:  
  - Changing `order_id=789` to `order_id=790` changes another user's delivery address.

- **False Case**:  
  - The server denies access: "You are not authorized to modify the delivery address of this order."

---

### 7. Viewing Booking Details of Another User
- URL: `https://example.com/hotel/booking_details?booking_id=321`

- **True Case**:  
  - Modifying `booking_id` lets the attacker view another user's booking.

- **False Case**:  
  - The server returns: "You are not authorized to view this booking's details."

---

### 8. Accessing Another User's Private Photos
- URL: `https://example.com/images/view?image_id=567`

- **True Case**:  
  - Changing `image_id` allows access to another user's photo.

- **False Case**:  
  - The application returns: "You cannot view this user's photo."

---

### 9. Using Base64 Encoded ID
- URL: `https://example.com/user/profile?id=MTIz` (Base64 for 123)

- **True Case**:  
  - Changing the ID to `MTI0` (Base64 for 124) grants unauthorized access.

- **False Case**:  
  - The application checks ownership and denies access with: "Unauthorized access."

---

### 10. Response-Based IDOR (No Request Manipulation)
- URL: `https://example.com/dashboard`

- **True Case**:  
  - The server includes other users' data in the response:
```json
{
  "user": { "id": 123, "name": "Ahmed" },
  "other_users": [
    { "id": 124, "name": "Ali" },
    { "id": 125, "name": "Sarah" }
  ]
}
```

- **False Case**:  
  - The server only returns the current user's data:
```json
{
  "user": { "id": 123, "name": "Ahmed" }
}
```

---

## Impact of IDOR

- **Privacy violation**: Exposure of personal information like name, email, address, and phone number.
- **Sensitive data exposure**: Disclosure of financial records, medical data, or transaction history. May violate regulations like GDPR or HIPAA.
- **Account Takeover preparation**: Attacker may collect information to aid phishing or brute-force attacks.
- **Reconnaissance for bigger attacks**: May lead to data deletion, manipulation, or unauthorized actions.
- **Loss of user trust**: If users discover data exposure, the reputation and credibility of the platform can be severely damaged.

---

## Tools for IDOR testing

- **Burp Suite Repeater / Intruder**: For manual or automated parameter tampering.
- **Burp Autorize Extension**: Detects IDOR by comparing authorized vs. unauthorized session responses.
- **OWASP ZAP**: Includes access control and IDOR-related scanners.
- **ffuf / wfuzz**: For brute-forcing object IDs in URLs or parameters.
- **Multiple Sessions Testing**: Testing with both attacker and victim sessions for comparison.

---

## Advanced scenarios

### IDOR in GraphQL
```graphql
{
  user(id: 124) {
    name
    email
  }
}
```
- If authorization checks are missing, this allows viewing another user's data.

---

### IDOR in file access
- URL: `https://example.com/download?file=report123.pdf`
- Modifying the file name may lead to unauthorized file access.

---

### IDOR in API body
```json
{
  "user_id": 456
}
```
- Even if the ID is in the body, backend must validate ownership.

---

## Prevention and Mitigation

1. **Continuous user permission validation**  
   Always check that the authenticated user has permission to access or modify the resource.

2. **Use unguessable identifiers**  
   Replace sequential IDs with UUIDs or hashed references.

3. **Backend enforcement of authorization**  
   Don't trust frontend values. Ensure all access checks are done server-side.

4. **Implement strong authentication**  
   Use sessions, JWTs, and secure login flows to verify user identity.

5. **Centralized access control logic**  
   Use middleware to enforce access control consistently across the application.

6. **Log and Alert suspicious behavior**  
   Monitor for brute-force or sequential ID enumeration attempts.

