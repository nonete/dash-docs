
## delete.py

This script purges Review Apps and their associated services after a set 
grace period or once they merge into production. It must be run on a schedule.

It will initially query the Dash Enterprise host for all Dash apps and
associated services that could be Redis or Postgres databases. 
The data is then parsed and sorted. After it completes those steps, it
filters results for apps matching the criteria specified in ``settings.py`. 
In our case, that would be `PREFIX` which specifies your review app naming 
pattern, and `LAST_UPDATE` determines the amount of time that the review apps 
remain on the server. We delete apps first, then the services.

-----