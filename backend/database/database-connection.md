1. Application uses [[database-driver]] to open connection.
2. Network socket opened to connect application to database
3. User authenticated
4. Operation completes 
5. Network socket is closed

Opening and closing connections aren't that expensive, but when scaling it could turn into expensive operation, and for that [[connection-pooling]] is used

#database 