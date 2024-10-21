```bash
# install
sudo apt install nginx

# test configuration
sudo nginx -t

# restart service
sudo service nginx restart

# view logs
	## error logs
	sudo tail -f /var/log/nginx/error.log
	## access logs
	sudo tail -f /var/log/nginx/access.log

# default server location
sudo vim /etc/nginx/sites-available/default

# add new site
sudo vim /etc/nginx/sites-available/new-site
sudo ln -s /etc/nginx/sites-available/new-site /etc/nginx/sites-enabled/new-site

```