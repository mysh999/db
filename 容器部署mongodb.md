```bash
# cat docker-compose.yml 
version: '3'
services:
  mongo:
    image: mongo:3.4.2
    container_name: mongodb
    environment:
        - MONGO_INITDB_ROOT_USERNAME=root
        - MONGO_INITDB_ROOT_PASSWORD=123456
    restart: always
    volumes:
      - "/data/mongo:/data/db"
      - "/data/mongo-entrypoint/:/docker-entrypoint-initdb.d/"
    ports:
      - 27017:27017
    command: mongod 
```

