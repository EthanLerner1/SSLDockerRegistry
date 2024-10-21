# Docker registry

- Running registry currently on localhost as a poc.

```bash
docker run -d --restart=always --name registry -p 5000:5000 registry:2
```

- This commands run a registry docker image on port 5000. 
- The data is stored inside the container at `/var/lib/registry/docker/registry` so it is not persistent. (of course it can be fixed with docker volume)
- You can test the docker registry now

```bash
docker pull hello-world:latest
docker tag hello-world:latest localhost:5000/hello-world
docker push localhost:5000/hello-world
```

## SSL Certificates

- Generate a root CA

```bash
# Generate the CA KEY
  openssl genrsa -out rootCA.key 2048

# Generate the CA .crt
	openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.crt
```

- Generate certificates with SAN (subject alternative name) [[Certificates with san]]
- Trust the root ca in your machine
  ```bash
  sudo cp rootCA.crt /usr/share/ca-certificates
  sudo dpkg-reconfigure ca-certificates
```

# NGINX Configuration

## General Nginx Configuration
- make sure you are familiar with [[Nginx cheet-sheet]]
- edit `/etc/nginx/sites-available/default` to contain
```nginx
server {
        listen 443 ssl;
        server_name domain1.com;

        ssl_certificate /mnt/c/Projects/Vault/registry/certs/domain1.com.crt;
        ssl_certificate_key /mnt/c/Projects/Vault/registry/certs/domain1.com.key;
		location /v2/ {
			# Reverse proxy to the internal Docker registry
			proxy_pass http://172.21.96.1:5000;  # Assuming Docker registry runs on port 5000

			# Set necessary headers for Docker registry
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;

			# Disable buffering to prevent delays with large image layers
			proxy_buffering off;

			# If you're using Docker Registry authentication, you may need to allow certain methods
			proxy_http_version 1.1;
			proxy_set_header Connection "";
        }
        
```

# Perform NGINX rewrite
If you want to pull a docker image from domain1.com/library/public and it would be routed to the registry as localhost:5000/projects
```bash
docker pull domain1.com/library/public/python:3 -> localhost:5000/projects/python:3
```

you can simply add another rewrite location before the one from before
```nginx
server {
        listen 443 ssl;
        server_name domain1.com;

        ssl_certificate /mnt/c/Projects/Vault/registry/certs/domain1.com.crt;
        ssl_certificate_key /mnt/c/Projects/Vault/registry/certs/domain1.com.key;

        location /v2/library/public/ {
	        # Rewrite the URI, replacing "library/public" with "projects"
	        rewrite ^/v2/library/public/(.*)$ /v2/projects/$1 break;
	
	        # Proxy pass to the internal Docker registry at 172.21.96.1
	        proxy_pass http://172.21.96.1:5000;
	
	        # Set necessary headers
	        proxy_set_header Host $host;
	        proxy_set_header X-Real-IP $remote_addr;
	        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	        proxy_set_header X-Forwarded-Proto $scheme;
	
	        # Disable buffering for large Docker image layers
	        proxy_buffering off;
	
	        # If Docker Registry authentication is required, allow it
	        proxy_http_version 1.1;
	        proxy_set_header Connection "";
	    }


        location /v2/ {
                # Reverse proxy to the internal Docker registry
                proxy_pass http://172.21.96.1:5000;  # Assuming Docker registry runs on port 5000

                # Set necessary headers for Docker registry
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;

                # Disable buffering to prevent delays with large image layers
                proxy_buffering off;

                # If you're using Docker Registry authentication, you may need to allow certain methods
                proxy_http_version 1.1;
                proxy_set_header Connection "";
        }
}

```
