
# Table of Contents

1.  [What AI helped me improve](#orgcb4522d)
2.  [Where I think AI miseld me](#orged5336e)
3.  [Thinking AI can&rsquo;t do](#org87a933c)

Note: Using Codex, provided my notes and the task description received via Linkedin and asked to improve

-   First pass was a failure, AI generated both improvements and the sections I&rsquo;m supposed to judge on my own lmao
-   Second pass: Only provide notes, point at repo, and ask for improvments and analysis


<a id="orgcb4522d"></a>

# What AI helped me improve

-   &ldquo;Data structures should be keyed by protocol entities, not token-pair string&rdquo; - This one I can genuinely agree on. I felt iffy while using that mechanism too. It suggested:

Instead of HashMap<Hash(TokenPair), (MarketModel, Vec<OpenPosition>)>, use:

-   MarketKey -> MarketState
-   MarketKey -> PositionsBySide
-   Owner/Market/Collateral/Side -> Position
-   OrderKey -> OrderPlan
-   TokenMint -> FeedState
-   MarketKey -> FeedSet
-   MarketKey -> VirtualInventorySet

Which I had completely missed, and would have to change my data structures around.

-   &ldquo;Health checks and market assignment are good, but automatic role removal is risky. Removing ORDER<sub>KEEPER</sub> based on runtime error
    rate can punish honest keepers during oracle outages, market config changes, blockhash expiry, account contention, or stale local
    state.&rdquo;

This is good direction. I overdesigned on this part. But, I would disagree with stale local state as a punishment metric. In my doc I mentioned taking down (resetting) underperforming keepers which is what most of the above would fall into. Hence I don&rsquo;t totally agree with the specifics, but the automatic role removal and punishment would be a good idea to remove from the design.


<a id="orged5336e"></a>

# Where I think AI miseld me

-   &ldquo;RPC should be used for v1&rdquo; - That would defeat the entire purpose of this design.
-   &ldquo;v1 scope is too narrow&rdquo; - I don&rsquo;t agree with this. It mentions Deposits/Withdrawals/Shifts etc. But I made them generic over Order Settling to avoid repeating myself 4 times. The flow of data across them are similar, in my opinion, and laying out generic data flow is better to express how it would flow.
-   &ldquo;should defer keeper fleets&rdquo; - I don&rsquo;t agree with this. I made a simplified version of this for v1 to address some of the major concerns around keepers completely blind to what&rsquo;s happening. In contrast it would be, each keeper taking on full responsibility for every market and update, which I don&rsquo;t think will scale well in terms of collision and workload for the machine.
-   &ldquo;Liquidation should be described as position<sub>cut</sub>&rdquo; - Again a bit too report focused instead of focusing on correctness of the report.
-   &ldquo;You mention signed blobs, but this repo already has a pull-oracle abstraction where builders expose required feed IDs, fetch
    price updates, post them, consume them, and close them: crates/sdk/src/client/pull<sub>oracle.rs</sub>:1.&rdquo;

Very very helpful, I missed this in the sdk entirely. Would adjust my design around this.

-   Missing idempotency and reconciliation model
    
    Add a section for the local state machine:
    
    Seen -> Planned -> Simulated -> Sent -> Landed -> Reconciled
                     -> FailedRecoverable -> Retry
                     -> FailedTerminal -> Drop/WaitForState
    
    Reconciliation should come from Geyser account updates and emitted events, not only RPC confirmation. Confirmation tells you a
    transaction landed; account/event reconciliation tells you whether your local cache should remove, retry, or re-plan the action.

Don&rsquo;t agree with this at all, the design revolves around seperation of concerns. It&rsquo;s mentioned in the docs that updates will invalidate and update local state, while the TPU unit is only responsible for measuring transaction landing and confirmations.

-   &ldquo; Your hot-path cache model is over-optimized too early&rdquo; -> Hard disagree. In my experience, the design for any performant service has to have the direction of optimization early, otherwise abstractions and tech debt make it much harder as time goes on.


<a id="org87a933c"></a>

# Thinking AI can&rsquo;t do

-   It wasn&rsquo;t able to connect the dots in my proposal design that a human would be able to. Like some points in &ldquo;Where I think Ai misled me&rdquo;.
-   No consideration around New infra like &ldquo;cloudbreak&rdquo; and &ldquo;tpu-client-next&rdquo;.
-   No judgement around tech debt and future rewrites.
-   Very very focused on specific naming and conventions rather than missed angles, possible points of failure (for example in my doc I name only 1 manager, which makes it a SPOF) or any better alternatives

There&rsquo;s a possibility that my prompt was horrible, but this is what I&rsquo;ve received and commented on

