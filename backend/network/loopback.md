# Network interfaces
Network interfaces (hardware or software) allows communicating end points (phone, laptop, server), by converting data into (electrical, light, radio) signals.  
> In short, network interface is the point of an interconnection between user computer, or a server and a network, at which converting is happening.

# Loopback address
Loopback address is a distinct reserved range of ip addresses 127.0.0.0 -> 127.255.255.255, that is used by devices to send and receive their own packets.  
**_In other words_, address that used to direct operations (communicate), _within itself!_**
> `127.0.0.1` is generally known as a loopback address in IPv4 standart, also known as `localhost` that is reserved for a loopback purposes

`localhost` is used to access itself (services that running on local host), for example when we running API on our machine and calling DB that are also running on our own machine. Basically it is used for testing, developing.  
<img width="933" alt="image" src="https://github.com/KidPudel/backend-notes/assets/63263301/b6e5ac47-b2c6-4daf-91d8-9067dc58f9da">


# Loopback interface
Loopback interface is a special network interface, that is used to communicate within a device itself. It is often assigned IP address of `127.0.0.1`
