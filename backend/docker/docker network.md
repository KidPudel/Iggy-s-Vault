
We can create a user-defined bridge between 2 and more containers to run in a shared network or docker network.  
Create a docker network:

```bash
docker network create your-network
```

If those containers share a docker network, then when specifying hostname / address in database, we can simply put name of a docker network

#backend
