In golang philosophy for the errors is that errors are data as well, and therefore you can interact with it
In golang it is idiomatic to manually and explicitly manage your errors, so that every case in your program is covered, and if error happens you know exactly where that happened

Golang's design lets you always take into account scenarios of the error

it is designed to pass the error down the stream decorating it to track each place