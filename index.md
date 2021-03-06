# Portfolio

---

## /tg/Station 13 

At the open source project /tg/Station 13, GitHub's most active open source video game (that was ranked 7th in number of comments), I am responsible for overseeing the technical operations for all our web properties, while also being the domain expert on the game engine and its proprietary language, database programming, and the various frameworks I've created to aid common development tasks.

I revamped the scheduler that runs our game loop and all associated timed tasks such that players dubbed it "The death of lag"; Revamped deployment from a manual process involving manuallying compiling and copying of binaries on the production server to a operator initiated automated process utilizing blue-green deployments; Took a fine-tooth comb to clear out memory and cpu sinkholes; and authored a myriad of programmer QoL enhancements to the game's core framework.

### /tg/Station 13 Projects:

#### [Master Controller (Game loop/timing logic) Redesign.](/master_controller)
Resign the system responsible for triggering internal processes to intelligently manage how long internal processes are allowed to run. Challenge mode: *Single threaded*

Dubbed "The death of lag" by our players.

---
#### /tg/Station Server Framework (v1-v2)
Framework for managing a byond engine based game server in a production environment when backed by an authoritative git server.

Handles compilation, staging, selective tree presistants (e/g: config, data), and updating the underlying game code without restarting the game server using blue/green symlinks such that the new code will seamlessly take effect next game round.

---
#### [Extensive system for publishing game logs](/game_logs)

My Head In-Game-Administrators approached me about publicizing a filtered version of our game round logs for various ends.

I over engineered the system purely to avoid having to build a front end.

---
#### Developer Focused Testing Database Package
As /tg/station moved more and more systems to use our mariadb database instead of flat files, testing these systems became a nightmare for most of our open source contributors who were only just getting comfortable with coding in general, let alone setting up a RDMS server on their local machine to test with.

So I thought, what if that process was so easy a coder could do it?

Simply extract a zip, run a batch file, and enable the database module in the game server.

No configuration. No bootstrapping. No 7-part install process.

A second, included zip allows devs to reset their database by simply overwriting the data folder with a clean but still bootstrapped copy of the schema.

---
#### Github webhook framework
Created the framework used for processing webhook events from github, Added interface for maintainers to view error logs, implemented the following features:

* Discord announce of pr events.
* In-game announce of pr events.
* Generate changelog entries from specially formatted text in the pr body. (This change *significantly* increased engagement with the changelog system by open source contributors)

Further features were added by other contributors including auto-tagging based off of title, text, or what code directories were modified, as well as tracking features to fixes ratios for all contributors.

---
#### Single Sign-on System
Making users create a new account to engage with the wiki creates inertia and limits our contributor pool, especially if they already have a forum account. I decided this was unacceptable and made the wiki authenticate users to their forum account.

The forum account system has become our primary account system and how all admin tools authenticate users. 

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



