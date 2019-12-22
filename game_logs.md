## [Public Game Logs](https://github.com/tgstation/tgstation13.org/blob/master/public_log_parser.php)

This came from a Head Game Administrator initiative to increase transparency and trust in game administrators as well as stomp out abuse by making a filtered version of our game logs available to the players.

The first design iteration had me storing the filtered logs in a database (with the compressed `row_format`), and serving them with php, this had the issue that the database server had to uncompress the log file, then the web server had to http-compress the log file every time it was viewed, despite log files being immutable once the round is over. This iteration was also unacceptable because it meant I had to design a front end.

So I set out to define some goals in my mind and restarted the design process with them in... mind.

1. Read once
  * These files can sometimes get large during long or active rounds, and we won't process them until the round is over.
1. Store compressed
  * Logging files are insanely compressible and at the time the webserver had a very limited disk quota.
  * It would break `Read Once` if compressing required reading the processed version of the game logs again to compress them. Best to compress them inline to the processing... process.
1. Serve compressed
  * Save bandwidth
1. Allow the browser to display the uncompressed game log
  * Making users uncompress the logs to read them would create an unnecessary level of inertia that would decrease engagement.
1. Do not bog down the webserver by making it uncompress or recompress anything.
1. Take advantage of webserver's auto-index front end.


These goals were achieved through the use of the gz* suite of php functions, a NGINX module to serve .log.gz files as http-gz compressed .log files, and some efficient engineering.

To keep things simple, I built the system to process the logs in order of newest to oldest, skipping the newest one on each server (meaning it was a round in progress) and going until it found a log file that already had a processed version stored, indicating the end of new rounds to process. This keep me from having to create or query a database with info on these logs and which had been processed.

Getting NGINX to play nicely with my goals took a minor bit of wrangling.

```
location /parsed-logs {
	autoindex on;
	gzip_static on;
	
	rewrite ^/parsed-logs/(.*).gz$ /parsed-logs/$1 redirect;
	
	default_type text/plain;
}
```

Autoindex will render a directory listing of all of the .gz log files, attempting to view one of these files triggers a rewrite rule that redirects all `blah.ext.gz` requests to to `blah.ext`, `http_gzip_static` will then look for a file matching `blah.ext` `+` `.gz` and if it finds one, serves that as a gzip compressed http page (regardless of rather or not `blah.ext` actually exists)

Players can access filtered versions of the game logs to audit admin's use of round modifying game tools and aid in reporting rule breaking behavior by other players.

Admins can access unfiltered versions of the game logs to aid in enforcing rule compliance. (Previously they had to connect to the game server and use admin tools to download log files to be opened in notepad.)

Coders and open source contributors can access error and debugging logs.

This system was successfully used by players to report player and admin abuse. The worst example being one of my *Head Administrators* who was using admin tools for an in-game advantage.