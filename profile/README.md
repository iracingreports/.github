# iRacing Reports

https://iracingreports.com

https://discordbot.iracingreports.com

- debian 12 vm on azure
- [python3.12](https://docs.brucerenner.com.au/posts/Debian-Buster-Python-Install/)
- postgresql15
- django 4.2


### set

- install debian 12
- install required apt packages
- install python3.12
- install postgresql15
- prepare database for django website

```
CREATE DATABASE iracing_reports;
CREATE USER c7yjuqxjod4q WITH PASSWORD 'pass';
CREATE SCHEMA SCHEMA AUTHORIZATION c7yjuqxjod4q;
ALTER ROLE c7yjuqxjod4q SET client_encoding TO 'utf8';
ALTER ROLE c7yjuqxjod4q SET default_transaction_isolation TO 'read committed';
ALTER ROLE c7yjuqxjod4q SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE iracing_reports TO c7yjuqxjod4q;
CREATE EXTENSION pg_trgm SCHEMA SCHEMA;
CREATE USER pound0837 WITH PASSWORD 'pass';
GRANT connect ON DATABASE iracing_reports TO pound0837;
GRANT usage ON iracing_reports TO pound0837;
GRANT usage ON SCHEMA schema TO pound0837;
GRANT SELECT ON ALL tables IN SCHEMA schema TO pound0837;
ALTER DEFAULT PRIVILEGES IN SCHEMA schema GRANT SELECT ON tables TO pound0837;
\q
```

```
git clone git@github.com:iracingreports/website.git
cd /home/fuz/website
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic
mkdir /home/fuz/iracingreports/website/staticfiles/tmp
```
