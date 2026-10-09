# TP1 Dockers

## Commands


### Database:
```
docker rm -f database
docker run -e POSTGRES_USER=usr -e POSTGRES_DB=db -e POSTGRES_PASSWORD=pwd --network app-network --name database -d raphaelaurent/database
```

### Backend:
```
docker rm -f backend
docker run --network app-network -p 8080:80 --name backend -d raphaelaurent/backend
```
Sans ports :
```
docker run --network app-network --name backend -d raphaelaurent/backend
```

### my-running-app :

```
docker rm -f my-running-app 
docker run -d -p 80:80 --network app-network --name my-running-app my-apache2
```


### Docker compose :
```
docker rm -f database backend my-running-app
docker compose up --build -d
```
