## Portfolio

---

### /tg/Station 13 

#### Master Controller (Game loop/timing logic) Redesign.
Resign the system responsible for triggering internal processes to intelligently manage how long internal processes are allowed to run. Challenge mode: *Single threaded*

Dubbed "The death of lag" by our players.

---
#### tgdb: A Player Database Front-End
LAMP Stack front end used by game administrators for accessing information about /tg/station players stored by the game server to the database.

Reports on player connection count, playtime hours, account standing/ban history, admin interaction notes.

---
#### /tg/Station Server Framework (v1-v2)
Framework for managing a byond engine based game server in a production environment when backed by an authoritative git server.

Handles compilation, staging, selective tree presistants (e/g: config, data), and updating the underlying game code without restarting the game server using blue/green symlinks such that the new code will seamlessly take effect next game round.

---
#### Callback Framework
Designed and implemented a framework to add callback support to a closed source proprietary language without traditional first class function support, standardizing a frequently buggy workflow, paving the way for efforts to make the code more maintainable.

Expanded system to support synchronously running a collection of async callbacks (i/e: select() syscall).

---
<p><small>(Note: this page is currently being constructed and incomplete.)</small></p>







