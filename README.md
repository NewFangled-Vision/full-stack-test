Create LexiPool — a deterministic promise pool with deps & cancellation

Build a small, deterministic async job runner with priorities, dependencies, and cancellation. 

Rules (read carefully)

 - Deterministic picking. At any moment you may start up to maxConcurrency - runningCount new jobs. Among all eligible jobs (see #2), pick in this order:

    *  priority: higher first
    *  eligibility time: FIFO by the moment they became eligible (not creation order)
    *  id: lexicographic ascending as final tiebreaker
      
 - Eligibility. A job is eligible iff:
    *  It isn’t cancelled,
    *  It hasn’t started yet,
    *  All dependencies have succeeded.
    *  (If it’s in a cycle, it will never become eligible—see #6.)

 - Start & concurrency. Starting a job immediately consumes a concurrency slot. Already-running jobs don’t block reordering beyond the maxConcurrency limit.

 - Dependency failure propagation. If any dependency fails, every dependent job that hasn’t run becomes skipped: "dependency-failed" (and can never run).

 - Cancellation. cancel(tag) immediately and atomically cancels all not-yet-started jobs with that tag; they become skipped: "cancelled". Running jobs continue.

 - Cycle handling. Before running, detect cycles in the dependency graph. Any job that is part of (or downstream of) a cycle that prevents success must be skipped: "cycle" unless it can still become eligible strictly via non-cyclic paths (i.e., only mark cyclic SCCs whose deps cannot be satisfied).

 - Events. Call onEvent(e) for:
   * { type:"start", id }
   * { type:"finish", id, result }
   * { type:"fail", id, errorMessage }
   * { type:"skip", id, reason }
   * { type:"cancelled", tag, count }
  

****Deliverables****

 - lexipool.js (or .ts)
   
 *  Implements LexiPool with clean data structures:
 *  Graph build + SCC cycle detection (Tarjan/Kosaraju or a simpler DFS marking works)
 *  Ready set with deterministic comparator (priority → eligibleAt → id)
 *  State maps: status, eligibleAtTick, running, dependents

 - lexipool.test.js

  * Use node:test or assert:
    a) Determinism: same inputs ⇒ same schedule array
    b) Tie-breaks: priority, then eligibility time, then id
    c) Eligibility time: jobs that become eligible later must not cut the line
    d) Dependency failure: propagate dependency-failed correctly
    e) Cancellation: before/after eligibility; doesn’t affect running jobs
    f) Cycles: simple 2-node cycle, 3-node SCC, mixed acyclic/acyclic subgraphs
    g) Concurrency: never exceed maxConcurrency; order of start events matches schedule
      
 * No sleeping or timers—tests complete fast.

 - README.md
    * Provide brief reasoning: how you compute eligibility time, how you ensure determinism, how cycles are handled, and what invariants your tests assert.
