# Setup nestjs nginx docker ssl

## Setup nestjs app
```sh
npx new nest-app
# localhost:3000
```

Dockerfile
```dockerfile
FROM node:12.19.0-alpine3.9 AS development

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install glob rimraf

RUN npm install --only=development

COPY . .

RUN npm run build

FROM node:12.19.0-alpine3.9 as production

ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install --only=production

COPY . .

COPY --from=development /usr/src/app/dist ./dist

CMD ["node", "dist/main"]

```

```sh
docker build -t nest-app .
docker run -p 3000:3000 nest-app -d
```


docker network create example-net

# Create SSL for localhost

### NoSSL config nginx
```sh
server {
  listen        80;
  server_name   dev.example.com;

  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_pass http://nodejsserver:3000;
  }
}
```
## SSL nginx config
### Add environment variable 

Using Git Windows version, need to config enviroment variables
2. OPENSSL_CONF=C:/Program Files/Git/mingw64/ssl/openssl.cnf
2. MSYS_NO_PATHCONV=1 git blame -L/pathconv/ msys2_path_conv.cc


### Generate RootCA.pem, RootCA.key & RootCA.crt:

```sh
openssl req -x509 -nodes -new -sha256 -days 1024 -newkey rsa:2048 -keyout RootCA.key -out RootCA.pem -subj "/C=VN/CN=dev.korinno.com"
openssl x509 -outform pem -in RootCA.pem -out RootCA.crt
```

### Config domains in file domains.ext
First, create a file domains.ext that lists all your local domains:

authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = localhost
DNS.2 = dev.korinno.com


### Register key and crt file

openssl req -new -nodes -newkey rsa:2048 -keyout localhost.key -out localhost.csr -subj "/C=VN/ST=THUDUC/L=HCM/O=Example-Certificates/CN=dev.korinno.com"
openssl x509 -req -sha256 -days 1024 -in localhost.csr -CA RootCA.pem -CAkey RootCA.key -CAcreateserial -extfile domains.ext -out localhost.crt

### Update local host file
C:\Windows\System32\drivers\etc
127.0.0.1 dev.korinno.com