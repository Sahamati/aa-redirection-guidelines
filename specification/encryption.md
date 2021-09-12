---
description: Note about the encryption of data passed as URL parameters
---

# Encryption

### Notes

1. Data-in-transit security by encrypting the communication channel: This is a must-have anyway and TLS 1.2 and above will be used \(https\), for communication between the FIU app and the AA web app. 
2. Additional Data-in-transit security through encryption of data: 

All encryption must be done using AES 256. For the AES 256 encryption below will be used:

**IV** – This can be 0  
**SALT** – This will be the reqdate or resdate   
**FI** – This will be the unique FIU ID \( i.e. the FIU entity id \)  
**SECRETKEY** – This will be the secret passphrase shared by the AA with the FIU. 

### References

* https://mkyong.com/java/java-aes-encryption-and-decryption/





