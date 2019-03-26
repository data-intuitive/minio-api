This repository is a fork of [`minio-db`](https://github.com/alexellis/minio-db).

# `minio-api`

`minio-api` is a thin layer to store JSON objects and binary blobs in a Minio object storage server with a GET/PUT interface over HTTP. For more on Minio, checkout <https://minio.io>

## Dependencies and Installation

### Minio

Minio can be started as a Docker instance. 

Here we pin the secret and access keys in order to have a self-consistent setup procedure.

```
docker run -p 9000:9000 --name minio \
  -e "MINIO_ACCESS_KEY=<access_key>" \
  -e "MINIO_SECRET_KEY=<secret_key>" \
  -v /data/minio:/data \
  -v /data/config:/root/.minio \
  minio/minio server /data
```

Please check the paths, this approach works on my system but may be inconsistent in other cases.

The output of this command gives information about the internal and external endpoints available. Note down the internal endpoint, we will soon need it.

### `minio-api`

You can check out this repo and build the Docker container yourself.

Alternatively, the following should get you up and running fast:

```
docker run -it \
  -p 8080:8080 \
  -e "access=<access_key>" \
  -e "secret=<secret_key>" \
  -e "host=172.17.0.2:9000" \
  -e "bucket=tables" \
  tverbeiren/minio-api
```

Please update the host variable if you got a different internal endpoint after starting the Minio container.

That's it.

## Usage

### General

Putting an object:

- URI: `/put/{object:[a-zA-Z0-9.-_]+}`
- Request Body: contents of object

Getting an object:

- URI: `/get/{object:[a-zA-Z0-9.-_]+}`
- Response body: contents of object if found

### Example

Now, you can easily test is using `httpie`

```
> http PUT localhost:8080/put/testblob @README.md
HTTP/1.1 200 OK
Content-Length: 17
Content-Type: text/plain; charset=utf-8
Date: Thu, 17 Jan 2019 09:40:58 GMT

Put testblob. OK.
```

What happened?

1. You `PUT` a file to `minio-api`
2. The path defines the _name_ of the blob, in this case `testblob`
3. The `README.md` content is stored in that blob

In order to fetch the information, just to a `GET` request providing the name/key of the blob:

```
> http GET localhost:8080/get/testblob
HTTP/1.1 200 OK
Content-Length: 558
Content-Type: application/json
Date: Thu, 17 Jan 2019 09:44:12 GMT

# minio-db

Minio-DB is a thin layer to store JSON objects and binary blobs in a Minio object storage server with a GET/PUT interface over HTTP. For more on Minio, checkout https://minio.io

(truncated)
```

