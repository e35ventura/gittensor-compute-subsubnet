# Spot market design notes (Discord thread, 2026-10-09)

Status: **open question, not a decision.** Captured from a Discord discussion between
Grant and Landyn. Grant: "i dont have answer of how" and "tbh maybe this idea is a
layer on top of the subnet like it isnt baked into the subnet." No mechanism here is
in the current build contract (`docs/compute-subsubnet.md`); it would be an addition.

## The question Kimbo asked to be summarized

How would a Lium-style GPU rental market behave if we added:

1. **Fungible pools.** Machines of the same spec in the same region are
   interchangeable. A buyer rents "a 5090 in India" instead of a specific box.
   Pool key: (spec, region). Possibly other dimensions (storage) later.
2. **Dynamic / surge pricing.** Price moves with pool utilization so there is
   always an on-demand machine available for the highest-marginal buyer. The
   today-state Grant pointed at: Lium is at 100% utilization, which means
   price is too low (money left on the table) and nobody can rent in a hurry.
3. **Re-rental.** A pool machine can be moved from a lower-paying renter to a
   higher-paying one when surge demand arrives, so the pool keeps serving the
   top of the demand curve. On Lium today a renter can sit on a machine forever
   and the operator has no legitimate way to be paid to kick them off.

## Positions in the thread

- Grant: start from the pricing side. If we set the price we set the margin. The
  pool / interchangeability piece is what makes dynamic pricing coherent, because
  you cannot run surge pricing on one-of-a-kind machine listings. Pools first,
  then a scaling rule on top.
- Landyn: wants the simplest possible scaling rule, not a full tokenized spot
  market. Example: a 30-line "ghetto" function
  ```text
  util > 80% -> +20% price
  util > 90% -> +40% price
  util > 95% -> +60% price
  ```
  tuned over time. Core concern: do not over-engineer an "idealistic" design
  that nobody can point to real Gittensor users or miners who need it. Prefers
  shipped and crude over elegant and theoretical.
- Grant conceded: the ghetto rule and the "fancy" design are the same thing done
  with different polish. To write even the ghetto rule you still have to decide
  which machines are in the same bucket, which is defining the pools. So pools
  are not optional, surge-pricing complexity is.
- Both agreed: nothing in this is strictly necessary. The practical wins are
  (a) higher capture of willing-to-pay demand via surge, and (b) always-on
  availability for a premium buyer who needs a specific GPU right now.

## Where it would live

Grant: possibly a layer on top of the subnet, not in the subnet. But the subnet
"allows for re-rental," which pulled him toward subnet-level. Undecided.

The current `docs/compute-subsubnet.md` builds something different: an inference
gateway with one global 5090 pool, slot reservation, and emission-funded target
scaling. It does **not** sell raw rentals, and it has no notion of per-pool price,
region pools, or preempting a renter. Adding this would be either:

- A new product tier alongside the inference tier (a rental tier that uses the
  same verified fleet when the inference pool is slack), or
- A separate market layer that routes rentals and runs the price curve, treating
  the subnet only as the proof-of-hardware substrate.

Neither route is picked yet.

## Open questions before this becomes a build task

1. Minimum interesting pool size. Below some count, "pool" is one box with a
   fancy name and no real fungibility. What is the floor spec+region count that
   justifies the extra plumbing?
2. Pool key dimensions. Spec + region is the start. Storage, NIC, idle vs
   datacenter class, uptime SLA. Which of these matter for day-one buyers?
3. Price rule. Ghetto step function vs. a continuous curve. Any reason this is
   not just a tunable constants file we edit in prod?
4. Re-rental mechanics. How much notice does the preempted renter get, what is
   the compensation (if any), and does the top bidder get the first machine or
   a sampled one?
5. Who actually asks for this on day 90. The thread kept returning to this and
   nobody named a buyer who needs it. Worth answering before writing code.
6. Does this need to be baked into the subnet or does it sit above it. See the
   "Where it would live" section; depends on whether re-rental and settlement
   can be enforced from an off-chain layer or require subnet state.

## Next step (suggested, not decided)

Pick one real buyer who would use surge-priced rentals on day 1, then write the
smallest possible version that serves them (one pool, one price rule, no
re-rental). If no buyer surfaces, this stays a note.
