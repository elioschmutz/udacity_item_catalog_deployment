# Deployment of the udacity-Item-Catalog

- URL: https://itemcatalog.elioschmutz.ch
- Github-URL: https://github.com/elioschmutz/udacity_item_catalog

- Type: AWS Lightsail Cloud Server
- IP: 18.185.100.161
- SSH-Port: 2200

## Installed software

- Apache (Webserver)
  - mod_wsgi
- PostgresSQL (Database Server)
- Finger (Displays user information)
- Virtualenv (Python version manager)
- Certbot (for generating let-encrypt certificates)

## Configuration

- VHost for itemcatalog.elioschmutz.ch
  - Forces SSL
  - Runs a wsgi-server
- Changed default SSH-port
- Disallow root-login
- Disallow password-login

## Third-party resources used to complete this project

- [Flask Configuration](http://flask.pocoo.org/docs/1.0/config/)
- [Flask Deployment](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)
- [Script activate_this.py](https://github.com/pypa/virtualenv/blob/master/virtualenv_embedded/activate_this.py)
- [Load WSGI-Application as python module without installing it](https://stackoverflow.com/questions/38378560/my-wsgi-application-cannot-be-loaded-as-python-module-what-am-i-doing-wrong/38392475#38392475)
