Base64 encoding is a way of representing binary data in an ASCII string format.

# encoding
1. Converting a data into a binary
2. Binary data is broken down to the 6-bit chunks `101100 011011 000110 111100 100000...`
3. each of those 6-bit chunks then mapped to the one of 64 printable ASCII characters `SGVs bG8g d29y bGQ=`
4. if the binary data length is not a multiple of 3 bytes (24 bits), pad the end with one or two `=` characters

# decoding
1. Remove padding characters from the end `=`
2. Map each Base64 character into 6-bit binary chunk
3. Combine those 6-bits into 8-bit bytes to reconstruct the original binary data. `011010000110010101101100011011000...`
4. convert binary back to the original data



---
The character set used for Base64 encoding consists of:
- The uppercase letters A-Z
- The lowercase letters a-z
- The digits 0-9
- The plus (+) and forward slash (/) characters

It is commonly used in:
- binary data transmissions over protocols like HTTP, SMTP, XML-RPC
- encoding images, audio, or video data for embedding in HTML or other text-based format
- encoding authentication credentials for Basic Authentication type of [[Authorization header]]
- encoding encrypted data or digital signatures