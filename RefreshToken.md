# **Refresh Token Flow in Authentication**

### **Example Flow for Refresh Token Usage**

1. **User Login**  
   - The user logs in and receives both an **access token** and a **refresh token**.  

2. **Access Token Expiry**  
   - The **access token** expires after a predefined period (e.g., 15 minutes).  

3. **Requesting a New Access Token**  
   - The client sends the **refresh token** to the `/refresh-token` endpoint to request a new access token.  

4. **Server Response**  
   - The server validates the refresh token and responds with:  
     - A **new access token**  
     - Optionally, a **new refresh token**  

5. **Continued API Access**  
   - The client uses the **new access token** for subsequent API requests.  

This process ensures a seamless authentication experience without requiring frequent logins while maintaining security.  
