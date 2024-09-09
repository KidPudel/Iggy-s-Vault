The difference between this [[pointer]] variable declaration:
```C
User* user;
```
And this:
```C
User *user;
```

Is that the *second* is more right considering logic part.

Because we declare ***variable pointer of***  type, and not a ~~variable *pointer of type*~~

and with that, we won't misunderstand the following line:
```C
User *user1, user2;
```
Here we clearly see that, we've declared two variables, where *one is being pointer, and other is not*
