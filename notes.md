# Notes for setting the backend in a local

# Installation
## Create a docker network
```
$ docker network create kong-net
```

## Start your database
```
$ docker run -d --name kong-database \
               --network=kong-net \
               -p 5432:5432 \
               -e "POSTGRES_USER=kong" \
               -e "POSTGRES_DB=kong" \
               postgres:9.6
```
## Prepare your database
```
$ docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     kong:latest kong migrations bootstrap
```

## Installing Kong (hybrid installation)
```
$ brew tap kong/kong
$ brew install kong
```

Finally start kong service
```
$ kong start
```

source: https://docs.konghq.com/install/macos/

## Installing Kong (full-docker) NOT WORKING BECOUSE CAN'T RESOLVE LOCALHOST
```
docker run -d --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 8000:8000 \
     -p 8443:8443 \
     -p 8001:8001 \
     -p 8444:8444 \
     kong:latest
```

## check if kong is runing
```
$ curl -i http://localhost:8001/
```

# Configuration

## Add the address book service

Method: POST
URL: http://localhost:8001/services
Body:
```
{
  "name": "address-book",
  "url": "http://localhost:3010"
}
```

## Add a route of address book endpoint

Method: POST
URL: http://localhost:8001/services/address-book/routes
Body:
```
{
	"hosts": ["localhost"],
	"paths": ["/api/contacts"],
	"methods": ["GET"],
	"strip_path": false
}
```

## Secure the address book service

### Add the JWT plugin to the service
Method: POST
URL: http://localhost:8001/services/address-book/plugins
Body:
```
{
  "name": "jwt"
}
```

### Create a consumer for this endpoint
Method: POST
URL: http://localhost:8001/consumers
Body:
```
{
	"username": "demo"
}
```

###
Download the .pem file from Auth0
COMPANYNAME=YourAuth0UserHere
```
$ curl -o key.pem "https://<auth0-company-name>.auth0.com/pem"
```

Extract the public key from the pem certificate
```
$ openssl x509 -pubkey -noout -in key.pem > pubkey.pem
```

### Setup the JWT auth fonr the consumer

Method: POST
URL: http://localhost:8001/consumers/demo/jwt
Body:
```
{
  "algorithm": "RS256",
  "rsa_public_key": "@./pubkey.pem",
  "key": "https://<auth0-company-name>.auth0.com"
}
```
or 

```
$ curl -i -X POST http://localhost:8001/consumers/demo/jwt \
    -F "algorithm=RS256" \
    -F "rsa_public_key=@./pubkey.pem" \
    -F "key=https://<auth0-company-name>.auth0.com"
```



```
docker run -p 1337:1337 \
             --network=kong-net \
             -e "TOKEN_SECRET={{somerandomstring}}" \
             -e "DB_ADAPTER=postgres" \
             -e "DB_HOST=kong-database" \
             -e "DB_PORT=5432" \
             -e "DB_USER=kong" \
             -e "DB_PASSWORD=kong" \
             -e "DB_DATABASE=kong-database" \
             --name konga \
             pantsel/konga
```
