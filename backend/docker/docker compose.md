Tools to manage multi-container applications on an individual server using yml config file
If load is too high, we might want to utilize [[orchestration]]

```yml
services:
	frontend:
		images: example/webapp
		ports:
			- "443:8043"
		networks:
			- front-tier
			- back-tier
		...
	backend:
		images: example/database
		networks:
			- back-tier
		...
	database:
		images: example/
```


commands:
- `docker-compose up`: run containers
- `docker-compose down`: stop containers
