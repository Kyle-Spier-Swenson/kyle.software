## Master Controller Redesign

### Background

The Master Controller on /tg/Station 13 is the name given to the framework responsable for running tasks that have to be ran every n seconds. One example of such a task would the `mob` subsystem that handles applying status effects that have a time component, like losing health while in a room with no oxygen, or the `atmos` subsystem, which handles calculating the spread and equalization of atmospheric pressure (This is a game about being on a space station after all).

What follows is a 1 year adventure where I attempt to improve the Master Controller on /tg/Station purely so ~~we~~ I can make atmospheric calculations happen more often without lagging the server.

### Prologue - I inherent ownership.

At this point, the MC had undergone a major rewrite, along with the first few rounds of bug fixes, but it still had one bug that was becoming increasingly annoying. The previous maintainer of the MC had left over personal differences, so our options were to either revert, or fix. At this point I had never interacted with the MC.

**Problem:** In production it was observed that for a short period after the round starts, all subsystems would trigger in rapid succession, not delaying for their configured time between runs. This issue could not be reproduced outside of production.

**Root cause found:** 

```dm
/datum/controller/game_controller/proc/process()
	//-snip-
	for(var/datum/subsystem/SS in subsystems)
		if(SS.can_fire > 0)
			if(SS.next_fire <= world.time)
				SS.next_fire += SS.wait
				SS.fire()
							

/datum/controller/game_controller/proc/roundHasStarted()
	for(var/datum/subsystem/SS in subsystems)
		SS.can_fire = 1
```

The system was designed to account for any delay between when a subsystem can run again, and when it actually runs again, but did not take into account that it intentionally doesn't run most subsystems during the pre-game lobby. When the round started, it would attempt to make up for any missed fires on these subsystems, 2 minutes worth, by firing them in rapid succession.

This did not show up in debugging because most coders would initiate an immediate round start when testing/debugging. This was also the clue that allowed me to find the cause where my colleagues failed. By focusing on what was different about when I was testing the game vs production, I honed in on that lack of pre-game lobby delay, and was able to reproduce it simply by allowing the pre-game lobby timer to count down.

**Solution:** Limit how quickly the system will fire a subsystem that is running behind; make subsystem start and stops a formalized method instead of inferred by state so state can be maintained properly.

From this point forth I became the subject matter expert of this framework, mainly because I seemed to have less trouble understanding it then the other contributors did. 

### Dynamic wait - I apply bandaids until their collective weight almost causes them to all fall off.

**Problem:** Some subsystems have their fire rate throttled purely for performance reasons, but this value is static while the actual amount of processing that a subsystem will require is heavily dynamic.

**Solution:** Create a system to dynamically control the fire rate of certain subsystems based off of how long they took to run last run.

**Problem:** Dynamic wait doesn't take the overall load on the server into account.

**Solution:** Add heuristics to take both of these things into account.

**Problem:** The system can create oscillations as multiple DWait-enabled subsystems react to the changes in server load they are themselves influencing.

**Solution:** Apply a rolling average to all heuristics values.

**Problem:** __None of the past few enhances are even tackling the right problem__ as all perceptional "lag" thats created server side (as opposed to network side) stems from control not being returned to to the underlying game engine in time for it to send network messages to clients with changed world state in their view. 

**Solution:** Implement feature where the MC will defer running a subsystem until the start of the next game tick if running it now would likely create a tick overrun.

### `tick_usage` - Teamwork saves the day.

**Problem:** Subsystems can in fact need *more* then 1 game tick (50-100ms, depending on that server's settings) to run. 

**Solution:** ~~Design improvement to allow subsystems to pause when they are gettin-~~

**Problem:** We don't have any time apis that are accurate beyond 1/10 of a second, nor any way to know where we are within a game tick, so we can't easily solve the above problem.

**Solution:** Petition the game engine developers to add features giving us more accurate time info, as well as info about the timing of the game tick.

This was a team effort between two other servers to get this feature added to the BYOND game engine.

What we got was a single calculate on read property of the `world` object: `tick_usage`

This was a float that represented how far (in a range of 1% to 100%) we were into the game tick, with 100% being the point in which the next tick would start processing. Since BYOND calculated this by looking at how many milliseconds had elapsed since the start of the game tick (as well as how long a game tick is), this same property could be used to extract a millisecond accurate stop-watch.

To use this property to solve lag, all we needed to do was pause long running operations until the next tick when we got to the end of the current game tick.

To aid in adaption I created a few macros that could be included in loops to check if it was time to pause, and standardized a few convetions to make this easy for new devs to utilize. This set of macros support both a return method of pausing, where static function variables are used to save place/position, as well as a sleep method of pausing, achievable because BYOND's Single Thread;Multiple Stacks backend allows us to control how far up the stack a soft-block like a `sleep()` call will propergate.

This was announced as *"the death of lag"* by our players, but it still had some issues.

### StonedMC - I took everything I learned and refactored the Master Controller

https://github.com/tgstation/tgstation/pull/17987

Over the course of about a month, I refactored our MC around the existence of the new `tick_usage` datapoint. I extended the pausing tool to all subsystems, and built the MC around the assumption that by default, subsystems can be told to pause after a given number of milliseconds, and they would do so, give or take a millisecond.

The headlining feature of this new Master Controller was its weighted priority heuristic that was used to calculate how much of the free game tick a subsystem can consume. Rather then a first come; first serve allocation of the game tick, the MC would take into account the priority values of all other subsystems that will have to fire during the same tick, and divvy up the game tick proportionally.

This made certain time sensitive but quick to run subsystems, like lighting (which operates on a queue of lights with changes to calculate, meaning it doesn't necessarily use more cpu just because you run it more often) feel much more responsive to users because these subsystems could get time in every tick to do a few things rather then let a back log build because something took 5 game ticks to calculate an explosion or a pressure leak.

With it I also set to solve a few other issues by taking assumptions the old MC was making about all subsystems, and moving them to subsystem flags.

For instance: The bug at the beginning of this adventure came (in part) from a feature of the MC that assumed that every subsystem should be ran *exactly* every n milliseconds, and any delays in firing it on time should be made up next fire by running it early. It turns out that most subsystems do not require this level of timing accuracy, so I moved it to a flag.

Beyond weighted priorities, it had became clear that its more appropriate if some subsystems could only take up "idle" cpu rather then make other subsystems compete with it for cpu time. So a way of specifying this was added and applied to the more superficial subsystems, like atmos.

Reception was a hit! Every major ss13 server ported over all or part of this refactored Master Controller, beating out other recently released redesigns.

