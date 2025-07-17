# Platform NGINX + PHP 8.3 + Maria DB 11.8

## Requirements

- Makefile
<br><br>

## Starting up

Copy the local automation Makefile to manage Docker containers easily
```bash
$ cp ./resources/automation/local/Makefile .
```

On every platform repository update
```bash
$ rm Makefile; cp ./resources/automation/local/Makefile .
```

Create the local .env file from the example *(remove its description comments)*
```bash
$ cp .env.example .env
```

Now, Makefile recipes can be executed

See the list of recipes with their usage description
```bash
$ make help
```

Set the proper project's ownership
```bash
$ make local-ownership-set
```
<br><br>

## Configuration

Open the .env file and define the ports to expose the APIREST.


*(more documentation in development)*