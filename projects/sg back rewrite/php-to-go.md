| php                                                      | go                                                                 |
| -------------------------------------------------------- | ------------------------------------------------------------------ |
| `class pgpdo extends PDO` PHP DB (postgresql) connection | `pgx/v5/pgxpool` connection pool of postgresql [[database-driver]] |
| `mypdo` PHP MySQL connection                             | `mysql` connector                                                  |
| `Redis`                                                  | `go-redis`                                                         |
| AuthSock is init auth if not authenticated               | JWT + possible singleton                                           |


brand 17 if SuperGood brand

# main
program starts from app.php


Application class, where it is a class that contains basic info, and spreads across the app
Auth class is the same, but we or remove it completely or, chop it to hold just some info, not important
# classes/user
classes/user has methods for init (create token and set filial and city id)
has methods
``` php
public $cart;
public $session;
public $token;
public $subscribe = false;
public static $filialId;
public static $filialcity;
```

# classes/auth (user)
has method for finding 






# api
[[getitems]]
