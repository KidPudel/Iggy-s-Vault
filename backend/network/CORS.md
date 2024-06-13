> Cross-Origin Resource Sharing. Mechanism that allows one site to request data from another in different [[URL]]

When making request from  one domain to another, first attaches its Origin to the header, and when we hit the other side (server), it has the set of "rules" or *validation checks* that the client should satisfy in order to get the data from the server. Server attaches the Header `Access-Control-Allow-Origin` and if Origin is satisfied, then data is fetched.

> So CORS is the mechanism *for safety*, where target can manage who can get the data, and client explicitly says who he is, to match the server's cross origin resource sharing rules.