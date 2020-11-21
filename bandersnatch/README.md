https://packaging.python.org/guides/index-mirrors-and-caches/#complete-mirror-with-bandersnatch

```
docker pull pypa/bandersnatch
docker run --rm pypa/bandersnatch bandersnatch --help
```

Docker file is 
https://github.com/pypa/bandersnatch/blob/master/Dockerfile

https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/

sudo docker build --build-arg PYTHON_VERSION=3.8 -t nil/bandersnatch:option .
sudo docker build --build-arg PYTHON_VERSION=3.8 --build-arg WITH_SWIFT=yes  -t nil/bandersnatch:option .

https://sourcery.ai/blog/python-docker/

get host IP from build
/sbin/ip route|awk '/default/ { print $3 }'
