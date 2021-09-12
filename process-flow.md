# Process Flow

1. FIU securely captures \(preferably in a authenticated web session\) user’s AA id from the FIU web user interface and submits to the FIU server.
2. FIU server generates a consent request \(as per ReBIT specifications\) with a unique consent request id.
3. FIU server sends redirection request to the web browser/webview along with FIU id, other parameters and call back URL for redirecting to AA Webview.
4. Browser/webview redirects/points the user to the AA domain.
5. AA webview performs user aa id authentication and processes consent request by seeking consent from user.
6. Based on user’s acceptance/rejection to the consent request, AA server sends status notification to FIU server along with consent request id.
7. AA then redirects/points the user back to the FIU domain to the call back URL passed in the original request.
8. FIU can now proceed with checking the consent status with FIU server.
9. If the user has provided consent, the FIU server can proceed to data request and data fetch from AA server.

