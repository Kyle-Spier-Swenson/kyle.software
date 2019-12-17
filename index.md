## Portfolio

---

### /tg/Station 13 

#### Master Controller (Game loop/timing logic) Redesign.
Resign the system responsible for triggering internal processes to intelligently manage how long internal processes are allowed to run. Challenge mode: *Single threaded*

Dubbed "The death of lag" by our players.

---
#### /tg/Station Server Framework (v1-v2)
Framework for managing a byond engine based game server in a production environment when backed by an authoritative git server.

Handles compilation, staging, selective tree presistants (e/g: config, data), and updating the underlying game code without restarting the game server using blue/green symlinks such that the new code will seamlessly take effect next game round.

---
#### Extensive system for publishing game logs.

The goals I set to achieve were:

1. Read once
  * These files can sometimes get large during long or active rounds, and we won't process them until the round is over.
1. Store compressed
  * Logging files are insanely compressible and at the time the webserver had a very limited disk quota.
1. Serve compressed
  * Save bandwidth
1. Allow the browser to display the uncompressed game log
  * Making users uncompress the logs to read them would create an unnecessary level of inertia that would decrease engagement.
1. Do not bog down the webserver by making it uncompress or recompress anything.
  * The first design iteration had me storing the filtered logs in a database, and serving them with php, this had the issue that the web server had to re-http-compress the log file every time it was viewed, despite log files being immutable once the round is over. 
1. Do not bog down my time by reimplementing already existing things like nginx's auto index
  * This mostly relates to locking down parts of them to in game admins only. 

This was achieved through the use of the gz* suite of php functions, a nginx module to serve .log.gz files as http-gz compressed .log files, and at least 5 new grey hairs.

Players can access filtered versions of the game logs to audit admin's use of round modifying game tools and aid in reporting rule breaking behavior.

Admins can access unfiltered versions of the game logs to aid in enforcing rule compliance. Authenticated via nginx's remote auth module.

Coders and open source contributors can access error and debugging logs

---
#### Developer Focused Testing Database Package
As /tg/station moved more and more systems to use our mariadb database instead of flat files, testing these systems became a nightmare for most of our open source contributors who were only just getting comfortable with coding in general, let alone setting up a RDMS server on their local machine to test with.

So I thought, what if that process was so easy a coder could do it?

Simply extract a zip, run a batch file, and enable the database module in the game server.

No configuration. No bootstrapping. No 7-part install process.

A second, included zip allows devs to reset their database by simply overriding the data folder with a clean but still bootstrapped copy of the schema.

---
#### Github webhook framework.
Created the framework used for processing webhook events from github, Added interface for maintainers to view error logs, implemented the following features:

* Discord announce of pr events.
* In-game announce of pr events.
* Generate changelog entries from specially formatted text in the pr body. (This change *significantly* increased engagement with the changelog system by open source contributors) 

---
#### Single Sign-on System
Making users create a new account to engage with the wiki creates inertia and limits our contributor pool, especially if they already have a forum account. I decided this was unacceptable and made the wiki authenticate users to their forum account.

The forum account system has become more of our primary account system.

---
#### tgdb: A Player Database Front-End
LAMP Stack front end used by game administrators for accessing information about /tg/station players stored by the game server to the database.

Reports on player connection count, playtime hours, account standing/ban history, admin interaction notes.

---
#### Callback Framework
Designed and implemented a framework to add callback support to a closed source proprietary language without traditional first class function support, standardizing a frequently buggy workflow, paving the way for efforts to make the code more maintainable.

Expanded system to support synchronously running a collection of async callbacks (i/e: select() syscall).

---
<p><small>(Note: this page is still being constructed but is mostly complete. Individual pages for each project with shorter overviews are planned.)</small></p>



