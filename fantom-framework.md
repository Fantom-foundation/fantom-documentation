Introduction
============

Blockchain has emerged as a technology for secure decentralized transaction ledgers with broad applications in financial systems, supply chains and health care. *Byzantine* fault tolerance  is addressed in distributed database systems, in which up to one-third of the participant nodes may be compromised. Consensus algorithms  ensures the integrity of transactions between participants over a distributed network  and is equivalent to the proof of *Byzantine* fault tolerance in distributed database systems .

A large number of consensus algorithms have been proposed. For example; the original Nakamoto consensus protocol in Bitcoin uses Proof of Work (PoW) . Proof Of Stake (PoS)  uses participants’ stakes to generate the blocks respectively. Our previous paper gives a survey of previous DAG-based approaches .

In the previous paper , we introduced a new consensus protocol, called *L*<sub>0</sub>. The protocol *L*<sub>0</sub> is a DAG-based asynchronous non-deterministic protocol that guarantees pBFT. *L*<sub>0</sub> generates each block asynchronously and uses the OPERA chain (DAG) for faster consensus by confirming how many nodes share the blocks.

The Lachesis protocol as previously proposed is a set of protocols that create a directed acyclic graph for distributed systems. Each node can receive transactions and batch them into an event block. An event block is then shared with it’s peers. When peers communicate they share this information again and thus spread this information through the network. In BFT systems we would use a broadcast voting approach and ask each node to vote on the validity of each block. This event is synchronous in nature. Instead we proposed an asynchronous system where we leverage the concepts of distributed common knowledge, dominator relations in graph theory and broadcast based gossip to achieve a local view with high probability of being a global view. It accomplishes this asynchronously, meaning that we can increase throughput near linearly as nodes enter the network.

In this work, we propose a further enhancement on these concepts and we formalize them so that they can be applied to any asynchronous distributed system.

Contributions
-------------

In summary, this paper makes the following contributions:

-   We introduce the n-row flag table for faster root selection of the Lachesis Protocol

-   We define continuous consistent cuts of a local view to achieve consensus

-   We present proof of how domination relationships can be used for share information

-   We formalize our proofs that can be applied to any generic asynchronous DAG solution

Paper structure
---------------

The rest of this paper is organised as follows. Section \[se:prelim\] describes our Fantom framework. Section \[se:lca\] presents the protocol implementation. Section \[se:con\] concludes our paper with some future directions. Proof of Byzantine fault tolerance is described in Section \[se:proof\].

Preliminaries
=============

The protocol is run via nodes representing users’ machines which together create a network. The basic units of the protocol are called event blocks - a data structure created by a single node to share transaction and user information with the rest of the network. These event blocks reference previous event blocks that are known to the node. This flow or stream of information creates a sequence of history.

The history of the protocol can be represented by a directed acyclic graph *G* = (*V*, *E*), where *V* is a set of vertices and *E* is a set of edges. Each vertex in a row (node) represents an event. Time flows left-to-right of the graph, so left vertices represent earlier events in history.

For a graph *G*, a path *p* in *G* is a sequence of vertices (*v*<sub>1</sub>, *v*<sub>2</sub>, …, *v*<sub>*k*</sub>) by following the edges in *E*. Let *v*<sub>*c*</sub> be a vertex in *G*. A vertex *v*<sub>*p*</sub> is the *parent* of *v*<sub>*c*</sub> if there is an edge from *v*<sub>*p*</sub> to *v*<sub>*c*</sub>. A vertex *v*<sub>*a*</sub> is an *ancestor* of *v*<sub>*c*</sub> if there is a path from *v*<sub>*a*</sub> to *v*<sub>*c*</sub>.

<embed src="OPERA_chain.pdf" height="188" />

Figure \[fig:operachain\] shows an example of an OPERA chain (DAG) constructed through the Lachesis protocol. Event blocks are representated by circles. Blocks of the same frame have the same color.

Basic Definitions
-----------------

<span>  ***(Lachesis)*** <span>The set of *protocols*</span></span>

<span>  ***(Node)*** <span>Each machine that participates in the Lachesis protocol is called a *node*. Let *n* denote the total number of nodes.</span></span>

<span>  ***(*k*)*** <span>A *constant* defined in the system.</span></span>

<span>  ***(Peer node)*** <span>A node *n*<sub>*i*</sub> has *k* *peer nodes*.</span></span>

<span>  ***(Process)*** <span>A process *p*<sub>*i*</sub> represents a machine or a *node*. The process identifier of *p*<sub>*i*</sub> is *i*. A set *P* = {1,...,*n*} denotes the set of process identifiers.</span></span>

<span>  ***(Channel)*** <span>A process *i* can send messages to process *j* if there is a channel (*i*,*j*). Let *C* ⊆ {(*i*,*j*) s.t. *i*, *j* ∈ *P*} denote the set of channels.</span></span>

<span>  ***(Event block)*** <span>Each node can create event blocks, send (receive) messages to (from) other nodes.The structure of an event block includes the signature, generation time, transaction history, and hash information to references.</span></span>

All nodes can create event blocks. The information of the referenced event blocks can be copied by each node. The first event block of each node is called a *leaf event*.

Suppose a node *n*<sub>*i*</sub> creates an event *v*<sub>*c*</sub> after an event *v*<sub>*s*</sub> in *n*<sub>*i*</sub>. Each event block has exactly *k* references. One of the references is self-reference, and the other *k*-1 references point to the top events of *n*<sub>*i*</sub>’s *k*-1 peer nodes.

<span>  ***(Top event)*** <span>An event *v* is a top event of a node *n*<sub>*i*</sub> if there is no other event in *n*<sub>*i*</sub> referencing *v*.</span></span>

<span>  ***(Height Vector)*** <span>The height vector is the number of event blocks *created* by the *i*<sup>*t**h*</sup> node.</span></span>

<span>  ***(In-degree Vector)*** <span>The in-degree vector refers to the number of *edges* from other event blocks created by other nodes to the top event block of this node. The top event block indicates the most recently created event block by this node.</span></span>

<span>  ***(Ref)*** <span>An event *v*<sub>*r*</sub> is called “ref" of event *v*<sub>*c*</sub> if the reference hash of *v*<sub>*c*</sub> points to the event *v*<sub>*r*</sub>. Denoted by *v*<sub>*c*</sub>↪<sup>*r*</sup>*v*<sub>*r*</sub>. For simplicity, we can use ↪ to denote a reference relationship (either ↪<sup>*r*</sup> or ↪<sup>*s*</sup>).</span></span>

<span>  ***(Self-ref)*** <span>An event *v*<sub>*s*</sub> is called “self-ref" of event *v*<sub>*c*</sub>, if the self-ref hash of *v*<sub>*c*</sub> points to the event *v*<sub>*s*</sub>. Denoted by *v*<sub>*c*</sub>↪<sup>*s*</sup>*v*<sub>*s*</sub>.</span></span>

<span>  ***(*k* references)*** <span>Each event block has at least *k* references. One of the references is self-reference, and the other *k*-1 references point to the top events of *n*<sub>*i*</sub>’s *k*-1 peer nodes.</span></span>

<span>  ***(Self-ancestor)*** <span>An event block *v*<sub>*a*</sub> is self-ancestor of an event block *v*<sub>*c*</sub> if there is a sequence of events such that *v*<sub>*c*</sub>↪<sup>*s*</sup>*v*<sub>1</sub>↪<sup>*s*</sup>…↪<sup>*s*</sup>*v*<sub>*m*</sub>↪<sup>*s*</sup>*v*<sub>*a*</sub>. Denoted by *v*<sub>*c*</sub>↪<sup>*s**a*</sup>*v*<sub>*a*</sub>.</span></span>

<span>  ***(Ancestor)*** <span>An event block *v*<sub>*a*</sub> is an ancestor of an event block *v*<sub>*c*</sub> if there is a sequence of events such that *v*<sub>*c*</sub> ↪ *v*<sub>1</sub> ↪ … ↪ *v*<sub>*m*</sub> ↪ *v*<sub>*a*</sub>. Denoted by *v*<sub>*c*</sub>↪<sup>*a*</sup>*v*<sub>*a*</sub>.</span></span>

For simplicity, we simply use *v*<sub>*c*</sub>↪<sup>*a*</sup>*v*<sub>*s*</sub> to refer both ancestor and self-ancestor relationship, unless we need to distinguish the two cases.

<span>  ***(Flag Table)*** <span>The flag table is a *n**x**k* matrix, where n is the number of nodes and k is the number of roots that an event block can reach. If an event block *e* created by *i*<sup>*t**h*</sup> node can reach *j*<sup>*t**h*</sup> root, then the flag table stores the hash value of the *j*<sup>*t**h*</sup> root.</span></span>

Lamport timestamps
------------------

Our Lachesis protocols relies on Lamport timestamps to define a topological ordering of event blocks in OPERA chain. By using Lamport timestamps, we do not rely on physical clocks to determine a partial ordering of events.

The “happened before“ relation, denoted by →, gives a partial ordering of events from a distributed system of nodes. Each node *n*<sub>*i*</sub> (also called a process) is identified by its process identifier *i*. For a pair of event blocks *v* and *v*′, the relation ”→" satisfies: (1) If *v* and *v*′ are events of process *P*<sub>*i*</sub>, and *v* comes before *v*′, then *b* → *v*′. (2) If *v* is the send(*m*) by one process and *v*′ is the receive(*m*) by another process, then *v* → *v*′. (3) If *v* → *v*′ and *v*′→*v*″ then *v* → *v*″. Two distinct events *v* and *v*′ are said to be concurrent if *v* ↛ *v*′ and *v*′↛*v*.

For an arbitrary total ordering ≺ of the processes, a relation ⇒ is defined as follows: if *v* is an event in process *P*<sub>*i*</sub> and *v*′ is an event in process *P*<sub>*j*</sub>, then *v* ⇒ *v*′ if and only if either (i) *C*<sub>*i*</sub>(*v*)&lt;*C*<sub>*j*</sub>(*v*′) or (ii) *C*(*v*)=*C*<sub>*j*</sub>(*v*′) and *P*<sub>*i*</sub> ≺ *P*<sub>*j*</sub>. This defines a total ordering, and that the Clock Condition implies that if *v* → *v*′ then *v* ⇒ *v*′.

We use this total ordering in our Lachesis protocol. This ordering is used to determine consensus time, as described in Section \[se:lca\].

<span>  ***(Happened-Immediate-Before)*** <span>An event block *v*<sub>*x*</sub> is said Happened-Immediate-Before an event block *v*<sub>*y*</sub> if *v*<sub>*x*</sub> is a (self-) ref of *v*<sub>*y*</sub>. Denoted by *v*<sub>*x*</sub> ↦ *v*<sub>*y*</sub>.</span></span>

<span>  ***(Happened-before)*** <span>An event block *v*<sub>*x*</sub> is said Happened-Before an event block *v*<sub>*y*</sub> if *v*<sub>*x*</sub> is a (self-) ancestor of *v*<sub>*y*</sub>. Denoted by *v*<sub>*x*</sub> → *v*<sub>*y*</sub>.</span></span>

Happened-before is the relationship between nodes which have event blocks. If there is a path from an event block *v*<sub>*x*</sub> to *v*<sub>*y*</sub>, then *v*<sub>*x*</sub> Happened-before *v*<sub>*y*</sub>. “*v*<sub>*x*</sub> Happened-before *v*<sub>*y*</sub>" means that the node creating *v*<sub>*y*</sub> knows event block *v*<sub>*x*</sub>. This relation is the transitive closure of happens-immediately-before. Thus, an event *v*<sub>*x*</sub> happened before an event *v*<sub>*y*</sub> if one of the followings happens: (a) *v*<sub>*y*</sub>↪<sup>*s*</sup>*v*<sub>*x*</sub>, (b) *v*<sub>*y*</sub>↪<sup>*r*</sup>*v*<sub>*x*</sub>, or (c) *v*<sub>*y*</sub>↪<sup>*a*</sup>*v*<sub>*x*</sub>. The happened-before relation of events form an acyclic directed graph *G*′=(*V*, *E*′) such that an edge (*v*<sub>*i*</sub>, *v*<sub>*j*</sub>)∈*E*′ has a reverse direction of the same edge in *E*.

<span>  ***(Concurrent)*** <span>Two event blocks *v*<sub>*x*</sub> and *v*<sub>*y*</sub> are said concurrent if neither of them happened before the other. Denoted by *v*<sub>*x*</sub> ∥ *v*<sub>*y*</sub>.</span></span>

Given two vertices *v*<sub>*x*</sub> and *v*<sub>*y*</sub> both contained in two OPERA chains (DAGs) *G*<sub>1</sub> and *G*<sub>2</sub> on two nodes. We have the following:
(1) *v*<sub>*x*</sub> → *v*<sub>*y*</sub> in *G*<sub>1</sub> if *v*<sub>*x*</sub> → *v*<sub>*y*</sub> in *G*<sub>2</sub>;
(2)*v*<sub>*x*</sub> ∥ *v*<sub>*y*</sub> in *G*<sub>1</sub> if *v*<sub>*x*</sub> ∥ *v*<sub>*y*</sub> in *G*<sub>2</sub>.

State Definitions
-----------------

Each node has a local state, a collection of histories, messages, event blocks, and peer information, we describe the components of each.

<span>  ***(State)*** <span>A state of a process *i* is denoted by *s*<sub>*j*</sub><sup>*i*</sup>.</span></span>

<span>  ***(Local State)*** <span>A local state consists of a sequence of event blocks *s*<sub>*j*</sub><sup>*i*</sup> = *v*<sub>0</sub><sup>*i*</sup>, *v*<sub>1</sub><sup>*i*</sup>, …, *v*<sub>*j*</sub><sup>*i*</sup>.</span></span>

In a DAG-based protocol, each *v*<sub>*j*</sub><sup>*i*</sup> event block is valid only if the reference blocks exist before it. From a local state *s*<sub>*j*</sub><sup>*i*</sup>, one can reconstruct a unique DAG. That is, the mapping from a local state *s*<sub>*j*</sub><sup>*i*</sup> into a DAG is *injective* or one-to-one. Thus, for Fantom, we can simply denote the *j*-th local state of a process *i* by the DAG *g*<sub>*j*</sub><sup>*i*</sup> (often we simply use *G*<sub>*i*</sub> to denote the current local state of a process *i*).

<span>  ***(Action)*** <span>An action is a function from one local state to another local state.</span></span>

Generally speaking, an action can be one of: a *s**e**n**d*(*m*) action where *m* is a message, a *r**e**c**e**i**v**e*(*m*) action, and an internal action. A message *m* is a triple ⟨*i*, *j*, *B*⟩ where *i* ∈ *P* is the sender of the message, *j* ∈ *P* is the message recipient, and *B* is the body of the message. Let *M* denote the set of messages. In the Lachesis protocol, *B* consists of the content of an event block *v*.
Semantics-wise, in Lachesis, there are two actions that can change a process’s local state: creating a new event and receiving an event from another process.
<span>  ***(Event)*** <span>An event is a tuple ⟨*s*, *α*, *s*′⟩ consisting of a state, an action, and a state. Sometimes, the event can be represented by the end state *s*′.</span></span>

The *j*-th event in history *h*<sub>*i*</sub> of process *i* is ⟨*s*<sub>*j* − 1</sub><sup>*i*</sup>, *α*, *s*<sub>*j*</sub><sup>*i*</sup>⟩, denoted by *v*<sub>*j*</sub><sup>*i*</sup>.

<span>  ***(Local history)*** <span>A local history *h*<sub>*i*</sub> of process *i* is a (possibly infinite) sequence of alternating local states — beginning with a distinguished initial state. A set *H*<sub>*i*</sub> of possible local histories for each process *i* in *P*.</span></span>

The state of a process can be obtained from its initial state and the sequence of actions or events that have occurred up to the current state. In the Lachesis protocol, we use append-only semantics. The local history may be equivalently described as either of the following:
*h*<sub>*i*</sub> = *s*<sub>0</sub><sup>*i*</sup>, *α*<sub>1</sub><sup>*i*</sup>, *α*<sub>2</sub><sup>*i*</sup>, *α*<sub>3</sub><sup>*i*</sup>…
*h*<sub>*i*</sub> = *s*<sub>0</sub><sup>*i*</sup>, *v*<sub>1</sub><sup>*i*</sup>, *v*<sub>2</sub><sup>*i*</sup>, *v*<sub>3</sub><sup>*i*</sup>…
*h*<sub>*i*</sub> = *s*<sub>0</sub><sup>*i*</sup>, *s*<sub>1</sub><sup>*i*</sup>, *s*<sub>2</sub><sup>*i*</sup>, *s*<sub>3</sub><sup>*i*</sup>, …

In Lachesis, a local history is equivalently expressed as:
*h*<sub>*i*</sub> = *g*<sub>0</sub><sup>*i*</sup>, *g*<sub>1</sub><sup>*i*</sup>, *g*<sub>2</sub><sup>*i*</sup>, *g*<sub>3</sub><sup>*i*</sup>, …
 where *g*<sub>*j*</sub><sup>*i*</sup> is the *j*-th local DAG (local state) of the process *i*. <span>  ***(Run)*** <span>Each asynchronous run is a vector of local histories. Denoted by *σ* = ⟨*h*<sub>1</sub>, *h*<sub>2</sub>, *h*<sub>3</sub>, ...*h*<sub>*N*</sub>⟩.</span></span>

Let *Σ* denote the set of asynchronous runs. We can now use Lamport’s theory to talk about global states of an asynchronous system. A global state of run *σ* is an *n*-vector of prefixes of local histories of *σ*, one prefix per process. The happens-before relation can be used to define a consistent global state, often termed a consistent cut, as follows.

Consistent Cut
--------------

Consistent cuts represent the concept of scalar time in distributed computation, it is possible to distinguish between a “before" and an “after"

In the Lachesis protocol, an OPERA chain *G* = (*V*, *E*) is a directed acyclic graph (DAG). *V* is a set of vertices and *E* is a set of edges. DAG is a directed graph with no cycle. There is no path that has source and destination at the same vertex. A path is a sequence of vertices (*v*<sub>1</sub>, *v*<sub>2</sub>, ..., *v*<sub>*k* − 1</sub>, *v*<sub>*k*</sub>) that uses no edge more than once.

An asynchronous system consists of the following sets: a set *P* of process identifiers; a set *C* of channels; a set *H*<sub>*i*</sub> is the set of possible local histories for each process *i*; a set *A* of asynchronous runs; a set *M* of all messages.

Each process / node in Lachesis selects *k* other nodes as peers. For certain gossip protocol, nodes may be constrained to gossip with its *k* peers. In such a case, the set of channels *C* can be modelled as follows. If node *i* selects node *j* as a peer, then (*i*, *j*)∈*C*. In general, one can express the history of each node in DAG-based protocol in general or in Lachesis protocol in particular, in the same manner as in the CCK paper .

<span>  ***(Consistent cut)*** <span>A consistent cut of a run *σ* is any global state such that if *v*<sub>*x*</sub><sup>*i*</sup> → *v*<sub>*y*</sub><sup>*j*</sup> and *v*<sub>*y*</sub><sup>*j*</sup> is in the global state, then *v*<sub>*x*</sub><sup>*i*</sup> is also in the global state. Denoted by **c**(*σ*).</span></span>

The concept of consistent cut formalizes such a global state of a run. A consistent cut consists of all consistent DAG chains. A received event block exists in the global state implies the existence of the original event block. Note that a consistent cut is simply a vector of local states; we will use the notation **c**(*σ*)\[*i*\] to indicate the local state of *i* in cut **c** of run *σ*.

A message chain of an asynchronous run is a sequence of messages *m*<sub>1</sub>, *m*<sub>2</sub>, *m*<sub>3</sub>, …, such that, for all *i*, *r**e**c**e**i**v**e*(*m*<sub>*i*</sub>) → *s**e**n**d*(*m*<sub>*i* + 1</sub>). Consequently, *s**e**n**d*(*m*<sub>1</sub>) → *r**e**c**e**i**v**e*(*m*<sub>1</sub>) → *s**e**n**d*(*m*<sub>2</sub>) → *r**e**c**e**i**v**e*(*m*<sub>2</sub>) → *s**e**n**d*(*m*<sub>3</sub>) ….

The formal semantics of an asynchronous system is given via the satisfaction relation ⊢. Intuitively **c**(*σ*)⊢*ϕ*, “**c**(*σ*) satisfies *ϕ*,” if fact *ϕ* is true in cut **c** of run *σ*.

We assume that we are given a function *π* that assigns a truth value to each primitive proposition *p*. The truth of a primitive proposition *p* in **c**(*σ*) is determined by *π* and **c**. This defines **c**(*σ*)⊢*p*.
<span>  ***(Equivalent cuts)*** <span>Two cuts **c**(*σ*) and **c****′**(*σ*′) are equivalent with respect to *i* if:
**c**(*σ*)∼<sub>*i*</sub>**c****′**(*σ*′) ⇔ **c**(*σ*)\[*i*\]=**c****′**(*σ*′)\[*i*\]
.</span></span>

<span>  ***(*i* knows *ϕ*)*** <span>*K*<sub>*i*</sub>(*ϕ*) represents the statement “*ϕ* is true in all possible consistent global states that include *i*’s local state”.
**c**(*σ*)⊢*K*<sub>*i*</sub>(*ϕ*)⇔∀**c****′**(*σ*′)(**c****′**(*σ*′)∼<sub>*i*</sub>**c**(*σ*)  ⇒  **c****′**(*σ*′) ⊢ *ϕ*)
</span></span>

<span>  ***(*i* partially knows *ϕ*)*** <span>*P*<sub>*i*</sub>(*ϕ*) represents the statement “there is some consistent global state in this run that includes *i*’s local state, in which *ϕ* is true.”
**c**(*σ*)⊢*P*<sub>*i*</sub>(*ϕ*)⇔∃**c****′**(*σ*)(**c****′**(*σ*)∼<sub>*i*</sub>**c**(*σ*)  ∧  **c****′**(*σ*)⊢*ϕ*)
</span></span>

  ***(Majority concurrently knows)***

The next modal operator is written *M*<sup>*C*</sup> and stands for “majority concurrently knows.” The definition of *M*<sup>*C*</sup>(*ϕ*) is as follows.

*M*<sup>*C*</sup>(*ϕ*)=<sub>*d**e**f*</sub>⋀<sub>*i* ∈ *S*</sub>*K*<sub>*i*</sub>*P*<sub>*i*</sub>(*ϕ*),
 where *S* ⊆ *P* and |*S*|&gt;2*n*/3.

This is adapted from the “everyone concurrently knows” in CCK paper . In the presence of one-third of faulty nodes, the original operator “everyone concurrently knows” is sometimes not feasible. Our modal operator *M*<sup>*C*</sup>(*ϕ*) fits precisely the semantics for BFT systems, in which unreliable processes may exist.
<span>  ***(Concurrent common knowledge)*** <span>The last modal operator is concurrent common knowledge (CCK), denoted by *C*<sup>*C*</sup>. *C*<sup>*C*</sup>(*ϕ*) is defined as a fixed point of *M*<sup>*C*</sup>(*ϕ* ∧ *X*).</span></span>

CCK defines a state of process knowledge that implies that all processes are in that same state of knowledge, with respect to *ϕ*, along some cut of the run. In other words, we want a state of knowledge *X* satisfying: *X* = *M*<sup>*C*</sup>(*ϕ* ∧ *X*). *C*<sup>*C*</sup> will be defined semantically as the weakest such fixed point, namely as the greatest fixed-point of *M*<sup>*C*</sup>(*ϕ* ∧ *X*).It therefore satisfies:

*C*<sup>*C*</sup>(*ϕ*)⇔*M*<sup>*C*</sup>(*ϕ* ∧ *C*<sup>*C*</sup>(*ϕ*))

Thus, *P*<sub>*i*</sub>(*ϕ*) states that there is some cut in the same asynchronous run *σ* including *i*’s local state, such that *ϕ* is true in that cut.
Note that *ϕ* implies *P*<sub>*i*</sub>(*ϕ*). But it is not the case, in general, that *P*<sub>*i*</sub>(*ϕ*) implies *ϕ* or even that *M*<sup>*C*</sup>(*ϕ*) implies *ϕ*. The truth of *M*<sup>*C*</sup>(*ϕ*) is determined with respect to some cut **c**(*σ*). A process cannot distinguish which cut, of the perhaps many cuts that are in the run and consistent with its local state, satisfies *ϕ*; it can only know the existence of such a cut.
<span>  ***(Global fact)*** <span>Fact *ϕ* is valid in system *Σ*, denoted by *Σ* ⊢ *ϕ*, if *ϕ* is true in all cuts of all runs of *Σ*.
*Σ* ⊢ *ϕ* ⇔ (∀*σ* ∈ *Σ*)(∀**c**)(**c**(*a*)⊢*ϕ*)
</span></span>

Fact *ϕ* is valid, denoted ⊢*ϕ*, if *ϕ* is valid in all systems, i.e. (∀*Σ*)(*Σ* ⊢ *ϕ*).
<span>  ***(Local fact)*** <span>A fact *ϕ* is local to process *i* in system *Σ* if *Σ* ⊢ (*ϕ* ⇒ *K*<sub>*i*</sub>*ϕ*).</span></span>

Dominator (graph theory)
------------------------

In a graph *G* = (*V*, *E*, *r*) a dominator is the relation between two vertices. A vertex *v* is dominated by another vertex *w*, if every path in the graph from the root *r* to *v* have to go through *w*. Furthermore, the immediate dominator for a vertex *v* is the last of *v*’s dominators, which every path in the graph have to go through to reach *v*.

<span>  ***(Pseudo top)*** <span>A pseudo vertex, called top, is the parent of all top event blocks. Denoted by ⊤.</span></span>

<span>  ***(Pseudo bottom)*** <span>A pseudo vertex, called bottom, is the child of all leaf event blocks. Denoted by ⊥.</span></span> With the pseudo vertices, we have ⊥ happened-before all event blocks. Also all event blocks happened-before ⊤. That is, for all event *v*<sub>*i*</sub>, ⊥ → *v*<sub>*i*</sub> and *v*<sub>*i*</sub> → ⊤.

<span>  ***(Dom)*** <span>An event *v*<sub>*d*</sub> dominates an event *v*<sub>*x*</sub> if every path from ⊤ to *v*<sub>*x*</sub> must go through *v*<sub>*d*</sub>. Denoted by *v*<sub>*d*</sub> ≫ *v*<sub>*x*</sub>.</span></span>

<span>  ***(Strict dom)*** <span>An event *v*<sub>*d*</sub> strictly dominates an event *v*<sub>*x*</sub> if *v*<sub>*d*</sub> ≫ *v*<sub>*x*</sub> and *v*<sub>*d*</sub> does not equal *v*<sub>*x*</sub>. Denoted by *v*<sub>*d*</sub>≫<sup>*s*</sup>*v*<sub>*x*</sub>.</span></span>

<span>  ***(Domfront)*** <span>A vertex *v*<sub>*d*</sub> is said “domfront” a vertex *v*<sub>*x*</sub> if *v*<sub>*d*</sub> dominates an immediate predecessor of *v*<sub>*x*</sub>, but *v*<sub>*d*</sub> does not strictly dominate *v*<sub>*x*</sub>. Denoted by *v*<sub>*d*</sub>≫<sup>*f*</sup>*v*<sub>*x*</sub>.</span></span>

<span>  ***(Dominance frontier)*** <span>The dominance frontier of a vertex *v*<sub>*d*</sub> is the set of all nodes *v*<sub>*x*</sub> such that *v*<sub>*d*</sub>≫<sup>*f*</sup>*v*<sub>*x*</sub>. Denoted by *D**F*(*v*<sub>*d*</sub>).</span></span>

From the above definitions of domfront and dominance frontier, the following holds. If *v*<sub>*d*</sub>≫<sup>*f*</sup>*v*<sub>*x*</sub>, then *v*<sub>*x*</sub> ∈ *D**F*(*v*<sub>*d*</sub>).

OPERA chain (DAG)
-----------------

The core idea of the Lachesis protocol is to use a DAG-based structure, called the OPERA chain for our consensus algorithm. In the Lachesis protocol, a (participant) node is a server (machine) of the distributed system. Each node can create messages, send messages to, and receive messages from, other nodes. The communication between nodes is asynchronous.

Let *n* be the number of participant nodes. For consensus, the algorithm examines whether an event block is *dominated* by 2*n*/3 nodes, where *n* is the number of all nodes. The Happen-before relation of event blocks with 2*n*/3 nodes means that more than two-thirds of all nodes in the OPERA chain know the event block.

The OPERA chain (DAG) is the local view of the DAG held by each node, this local view is used to identify topological ordering, select Clotho, and create time consensus through Atropos selection. OPERA chain is a DAG graph *G* = (*V*, *E*) consisting of *V* vertices and *E* edges. Each vertex *v*<sub>*i*</sub> ∈ *V* is an event block. An edge (*v*<sub>*i*</sub>, *v*<sub>*j*</sub>)∈*E* refers to a hashing reference from *v*<sub>*i*</sub> to *v*<sub>*j*</sub>; that is, *v*<sub>*i*</sub> ↪ *v*<sub>*j*</sub>.

<span>  ***(Leaf)*** <span>The first created event block of a node is called a leaf event block.</span></span>

<span>  ***(Root)*** <span>The leaf event block of a node is a root. When an event block *v* can reach more than 2*n*/3 of the roots in the previous frames, *v* becomes a root.</span></span>

<span>  ***(Root set)*** <span>The set of all first event blocks (leaf events) of all nodes form the first root set *R*<sub>1</sub> (|*R*<sub>1</sub>| = *n*). The root set *R*<sub>*k*</sub> consists of all roots *r*<sub>*i*</sub> such that *r*<sub>*i*</sub> ∉ *R*<sub>*i*</sub>, ∀ *i* = 1..(*k*-1) and *r*<sub>*i*</sub> can reach more than 2n/3 other roots in the current frame, *i* = 1..(*k*-1).</span></span>

<span>  ***(Frame)*** <span>Frame *f*<sub>*i*</sub> is a natural number that separates Root sets. The root set at frame *f*<sub>*i*</sub> is denoted by *R*<sub>*i*</sub>.</span></span>

<span>  ***(Consistent chains)*** <span>OPERA chains *G*<sub>1</sub> and *G*<sub>2</sub> are consistent if for any event *v* contained in both chains, *G*<sub>1</sub>\[*v*\]=*G*<sub>2</sub>\[*v*\]. Denoted by *G*<sub>1</sub> ∼ *G*<sub>2</sub>.</span></span>

When two consistent chains contain the same event *v*, both chains contain the same set of ancestors for *v*, with the same reference and self-ref edges between those ancestors.

If two nodes have OPERA chains containing event *v*, then they have the same *k* hashes contained within *v*. A node will not accept an event during a sync unless that node already has *k* references for that event, so both OPERA chains must contain *k* references for *v*. The cryptographic hashes are assumed to be secure, therefore the references must be the same. By induction, all ancestors of *v* must be the same. Therefore, the two OPERA chains are consistent.
<span>  ***(Creator)*** <span>If a node *n*<sub>*x*</sub> creates an event block *v*, then the creator of *v*, denoted by *c**r*(*v*), is *n*<sub>*x*</sub>.</span></span>

<span>  ***(Consistent chain)*** <span>A global consistent chain *G*<sup>*C*</sup> is a chain if *G*<sup>*C*</sup> ∼ *G*<sub>*i*</sub> for all *G*<sub>*i*</sub>.</span></span>

We denote *G* ⊑ *G*′ to stand for *G* is a subgraph of *G*′. Some properties of *G*<sup>*C*</sup> are given as follows:

1.  ∀*G*<sub>*i*</sub> (*G*<sup>*C*</sup> ⊑ *G*<sub>*i*</sub>).

2.  ∀*v* ∈ *G*<sup>*C*</sup> ∀*G*<sub>*i*</sub> (*G*<sup>*C*</sup>\[*v*\]⊑*G*<sub>*i*</sub>\[*v*\]).

3.  (∀*v*<sub>*c*</sub> ∈ *G*<sup>*C*</sup>) (∀*v*<sub>*p*</sub> ∈ *G*<sub>*i*</sub>) ((*v*<sub>*p*</sub> → *v*<sub>*c*</sub>)⇒*v*<sub>*p*</sub> ∈ *G*<sup>*C*</sup>).

<span>  ***(Consistent root)*** <span>Two chains *G*<sub>1</sub> and *G*<sub>2</sub> are root consistent, if for every *v* contained in both chains, *v* is a root of *j*-th frame in *G*<sub>1</sub>, then *v* is a root of *j*-th frame in *G*<sub>2</sub>.</span></span>

By consistent chains, if *G*<sub>1</sub> ∼ *G*<sub>2</sub> and *v* belongs to both chains, then *G*<sub>1</sub>\[*v*\] = *G*<sub>2</sub>\[*v*\]. We can prove the proposition by induction. For *j* = 0, the first root set is the same in both *G*<sub>1</sub> and *G*<sub>2</sub>. Hence, it holds for *j* = 0. Suppose that the proposition holds for every *j* from 0 to *k*. We prove that it also holds for *j*= *k* + 1. Suppose that *v* is a root of frame *f*<sub>*k* + 1</sub> in *G*<sub>1</sub>. Then there exists a set *S* reaching 2/3 of members in *G*<sub>1</sub> of frame *f*<sub>*k*</sub> such that ∀*u* ∈ *S* (*u* → *v*). As *G*<sub>1</sub> ∼ *G*<sub>2</sub>, and *v* in *G*<sub>2</sub>, then ∀*u* ∈ *S* (*u* ∈ *G*<sub>2</sub>). Since the proposition holds for *j*=*k*, As *u* is a root of frame *f*<sub>*k*</sub> in *G*<sub>1</sub>, *u* is a root of frame *f*<sub>*k*</sub> in *G*<sub>2</sub>. Hence, the set *S* of 2/3 members *u* happens before *v* in *G*<sub>2</sub>. So *v* belongs to *f*<sub>*k* + 1</sub> in *G*<sub>2</sub>.

Thus, all nodes have the same consistent root sets, which are the root sets in *G*<sup>*C*</sup>. Frame numbers are consistent for all nodes.
<span>  ***(Flag table)*** <span>A flag table stores reachability from an event block to another root. The sum of all reachabilities, namely all values in flag table, indicates the number of reachabilities from an event block to other roots.</span></span>

<span>  ***(Consistent flag table)*** <span>For any top event *v* in both OPERA chains *G*<sub>1</sub> and *G*<sub>2</sub>, and *G*<sub>1</sub> ∼ *G*<sub>2</sub>, then the flag tables of *v* are consistent if they are the same in both chains.</span></span>

From the above, the root sets of *G*<sub>1</sub> and *G*<sub>2</sub> are consistent. If *v* contained in *G*<sub>1</sub>, and *v* is a root of *j*-th frame in *G*<sub>1</sub>, then *v* is a root of *j*-th frame in *G*<sub>*i*</sub>. Since *G*<sub>1</sub> ∼ *G*<sub>2</sub>, *G*<sub>1</sub>\[*v*\]=*G*<sub>2</sub>\[*v*\]. The reference event blocks of *v* are the same in both chains. Thus the flag tables of *v* of both chains are the same.
<span>  ***(Clotho)*** <span>A root *r*<sub>*k*</sub> in the frame *f*<sub>*a* + 3</sub> can nominate a root *r*<sub>*a*</sub> as Clotho if more than 2n/3 roots in the frame *f*<sub>*a* + 1</sub> dominate *r*<sub>*a*</sub> and *r*<sub>*k*</sub> dominates the roots in the frame *f*<sub>*a* + 1</sub>.</span></span>

Each node nominates a root into Clotho via the flag table. If all nodes have an OPERA chain with same shape, the values in flag table will be equal to each other in OPERA chain. Thus, all nodes nominate the same root into Clotho since the OPERA chain of all nodes has same shape.

<span>  ***(Atropos)*** <span>An Atropos is assigned consensus time through the Lachesis consensus algorithm and is utilized for determining the order between event blocks. Atropos blocks form a Main-chain, which allows time consensus ordering and responses to attacks.</span></span>

For any root set *R* in the frame *f*<sub>*i*</sub>, the time consensus algorithm checks whether more than 2n/3 roots in the frame *f*<sub>*i* − 1</sub> selects the same value. However, each node selects one of the values collected from the root set in the previous frame by the time consensus algorithm and Reselection process. Based on the Reselection process, the time consensus algorithm can reach agreement. However, there is a possibility that consensus time candidate does not reach agreement . To solve this problem, time consensus algorithm includes minimal selection frame per next *h* frame. In minimal value selection algorithm, each root selects minimum value among values collected from previous root set. Thus, the consensus time reaches consensus by time consensus algorithm.

<span>  ***(Main-chain (Blockchain))*** <span>For faster consensus, *Main-chain* is a special sub-graph of the OPERA chain (DAG).</span></span>

The Main chain — a core subgraph of OPERA chain, plays the important role of ordering the event blocks. The Main chain stores shortcuts to connect between the Atropos. After the topological ordering is computed over all event blocks through the Lachesis protocol, Atropos blocks are determined and form the Main chain. To improve path searching, we use a flag table — a local hash table structure as a cache that is used to quickly determine the closest root to a event block.

In the OPERA chain, an event block is called a *root* if the event block is linked to more than two-thirds of previous roots. A leaf vertex is also a root itself. With root event blocks, we can keep track of “vital” blocks that 2*n*/3 of the network agree on.

\[H\] <embed src="Mainchain.pdf" title="fig:" height="302" />

Figure \[fig:mainchain\] shows an example of the Main chain composed of Atropos event blocks. In particular, the Main chain consists of Atropos blocks that are derived from root blocks and so are agreed by 2*n*/3 of the network nodes. Thus, this guarantees that at least 2*n*/3 of nodes have come to consensus on this Main chain.

Each participant node has a copy of the Main chain and can search consensus position of its own event blocks. Each event block can compute its own consensus position by checking the nearest Atropos event block. Assigning and searching consensus position are introduced in the consensus time selection section.

The Main chain provides quick access to the previous transaction history to efficiently process new incoming event blocks. From the Main chain, information about unknown participants or attackers can be easily viewed. The Main chain can be used efficiently in transaction information management by providing quick access to new event blocks that have been agreed on by the majority of nodes. In short, the Main-chain gives the following advantages:

- All event blocks or nodes do not need to store all information. It is efficient for data management.

- Access to previous information is efficient and fast.

Based on these advantages, OPERA chain can respond strongly to efficient transaction treatment and attacks through its Main-chain.

Lachesis Protocol
=================

*loop*: A, B = *k*-node Selection algorithm() Request sync to node A and B Sync all known events by Lachesis protocol Event block creation (optional) Broadcast out the message Root selection Clotho selection Atropos time consensus *loop*: Request sync from a node Sync all known events by Lachesis protocol

Algorithm \[al:main\] shows the pseudo algorithm for the Lachesis core procedure. The algorithm consists of two parts and runs them in parallel.

- In part one, each node requests synchronization and creates event blocks. In line 3, a node runs the Node Selection Algorithm. The Node Selection Algorithm returns the *k* IDs of other nodes to communicate with. In line 4 and 5, the node synchronizes the OPERA chain (DAG) with the other nodes. Line 6 runs the Event block creation, at which step the node creates an event block and checks whether it is a root. The node then broadcasts the created event block to all other known nodes in line 7. The step in this line is optional. In line 8 and 9, Clotho selection and Atropos time consensus algorithms are invoked. The algorithms determines whether the specified root can be a Clotho, assign the consensus time, and then confirm the Atropos.

- The second part is to respond to synchronization requests. In line 10 and 11, the node receives a synchronization request and then sends its response about the OPERA chain.

Peer selection algorithm
------------------------

In order to create an event block, a node needs to select *k* other nodes. Lachesis protocols does not depend on how peer nodes are selected. One simple approach can use a random selection from the pool of *n* nodes. The other approach is to define some criteria or cost function to select other peers of a node.

Within distributed system, a node can select other nodes with low communication costs, low network latency, high bandwidth, high successful transaction throughputs.

Dynamic participants
--------------------

Our Lachesis protocol allows an arbitrary number of participants to dynamically join the system. The OPERA chain (DAG) can still operate with new participants. Computation on flag tables is set based and independent of which and how many participants have joined the system. Algorithms for selection of Roots, Clothos and Atroposes are flexible enough and not dependence on a fixed number of participants.

Peer synchronization
--------------------

We describe an algorithm that synchronizes events between the nodes.

Node *n*<sub>1</sub> selects random peer to synchronize with *n*<sub>1</sub> gets local known events (map\[int\]int) *n*<sub>1</sub> sends RPC request Sync request to peer *n*<sub>2</sub> receives RPC requestSync request *n*<sub>2</sub> does an EventDiff check on the known map (map\[int\]int)

*n*<sub>2</sub> returns unknown events, and map\[int\]int of known events to *n*<sub>1</sub>

The algorithm assumes that a node always needs the events in topological ordering (specifically in reference to the lamport timestamps), an alternative would be to use an inverse bloom lookup table (IBLT) for completely potential randomized events.
Alternatively, one can simply use a fixed incrementing index to keep track of the top event for each node.

Node Structure
--------------

This section gives an overview of the node structure in Lachesis.

Each node has a height vector, in-degree vector, flag table, frames, clotho check list, max-min value, main-chain (blockchain), and their own local view of the OPERA chain (DAG). The height vector is the number of event blocks created by the *i*<sup>*t**h*</sup> node. The in-degree vector refers to the number of edges from other event blocks created by other nodes to the top event block of this node. The top event block indicates the most recently created event block by this node. The flag table is a *n**x**k* matrix, where n is the number of nodes and k is the number of roots that an event block can reach. If an event block *e* created by *i*<sup>*t**h*</sup> node can reach *j*<sup>*t**h*</sup> root, then the flag table stores the hash value of the *j*<sup>*t**h*</sup> root. Each node maintains the flag table of each top event block.

Frames store the root set in each frame. Clotho check list has two types of check points; Clotho candidate (*C**C*) and Clotho (*C*). If a root in a frame is a *C**C*, a node check the *C**C* part and if a root becomes Clotho, a node check *C* part. Max-min value is timestamp that addresses for Atropos selection. The Main-chain is a data structure storing hash values of the Atropos blocks.

<embed src="Node_structure.pdf" />

Figure \[fig:node\] shows an example of the node structure component of a node *A*. In the figure, each value excluding self height in the height vector is 1 since the initial state is shared to all nodes. In the in-degree vector, node *A* stores the number of edges from other event blocks created by other nodes to the top event block. The in-degrees of node *A*, *B*, and *C* are 1. In flag table, node *A* knows other two root hashes since the top event block can reach those two roots. Node *A* also knows that other nodes know their own roots. In the example situation there is no clotho candidate and Clotho, and thus clotho check list is empty. The main-chain and max-min value are empty for the same reason as clotho check list.

Peer selection algorithm via Cost function
------------------------------------------

We define three versions of the Cost Function (*C*<sub>*F*</sub>). Version one is focused around updated information share and is discussed below. The other two versions are focused on root creation and consensus facilitation, these will be discussed in a following paper.

We define a Cost Function (*C*<sub>*F*</sub>) for preventing the creation of lazy nodes. A lazy node is a node that has a lower work portion in the OPERA chain (has created fewer event blocks). When a node creates an event block, the node selects other nodes with low value outputs from the cost function and refers to the top event blocks of the reference nodes. An equation (\[eq1\]) of *C*<sub>*F*</sub> is as follows,

*C*<sub>*F*</sub> = *I*/*H*

where *I* and *H* denote values of in-degree vector and height vector respectively. If the number of nodes with the lowest *C*<sub>*F*</sub> is more than *k*, one of the nodes is selected at random. The reason for selecting high *H* is that we can expect a high possibility to create a root because the high *H* indicates that the communication frequency of the node had more opportunities than others with low *H*. Otherwise, the nodes that have high *C*<sub>*F*</sub> (the case of *I* &gt; *H*) have generated fewer event blocks than the nodes that have low *C*<sub>*F*</sub>..

<embed src="costfunction_1.pdf" />

Figure \[fig:costfunction\_1\] shows an example of the node selection based on the cost function after the creation of leaf events by all nodes. In this example, there are five nodes and each node created leaf events. All nodes know other leaf events. Node *A* creates an event block *v*<sub>1</sub> and *A* calculates the cost functions. Step 2 in Figure \[fig:costfunction\_1\] shows the results of cost functions based on the height and in-degree vectors of node *A*. In the initial step, each value in the vectors are same because all nodes have only leaf events. Node *A* randomly selects *k* nodes and connects *v*<sub>1</sub> to the leaf events of selected nodes. In this example, we set *k*=3 and assume that node *A* selects node *B* and *C*.

<embed src="costfunction_2.pdf" />

Figure \[fig:costfunction\_2\] shows an example of the node selection after a few steps of the simulation in Figure \[fig:costfunction\_1\]. In Figure \[fig:costfunction\_2\], the recent event block is *v*<sub>5</sub> created by node *A*. Node *A* calculates the cost function and selects the other two nodes that have the lowest results of the cost function. In this example, node *B* has 0.5 as the result and other nodes have the same values. Because of this, node *A* first selects node *B* and randomly selects other nodes among nodes *C*, *D*, and *E*.

The height of node *D* in the example is 2 (leaf event and event block *v*<sub>4</sub>). On the other hand, the height of node *D* in node structure of *A* is 1. Node *A* is still not aware of the presence of the event block *v*<sub>4</sub>. It means that there is no path from the event blocks created by node *A* to the event block *v*<sub>4</sub>. Thus, node *A* has 1 as the height of node *D*.

**Input:** Height Vector *H*, In-degree Vector *I* **Output:** reference node *r**e**f* min\_cost ← *I**N**F* *s*<sub>*r**e**f*</sub> ← None *c*<sub>*f*</sub> ← $\\frac{I\_k}{H\_k}$ min\_cost ← *c*<sub>*f*</sub> *s*<sub>*r**e**f*</sub> ← <span>k</span> *s*<sub>*r**e**f*</sub> ← *s*<sub>*r**e**f*</sub> ∪ *k* *r**e**f* ← random select in *s*<sub>*r**e**f*</sub>

Algorithm \[al:ns\] shows the selecting algorithm for selecting reference nodes. The algorithm operates for each node to select a communication partner from other nodes. Line 4 and 5 set min\_cost and *S*<sub>*r**e**f*</sub> to initial state. Line 7 calculates the cost function *c*<sub>*f*</sub> for each node. In line 8, 9, and 10, we find the minimum value of the cost function and set min\_cost and *S*<sub>*r**e**f*</sub> to *c*<sub>*f*</sub> and the ID of each node respectively. Line 11 and 12 append the ID of each node to *S*<sub>*r**e**f*</sub> if min\_cost equals *c*<sub>*f*</sub>. Finally, line 13 selects randomly *k* node IDs from *S*<sub>*r**e**f*</sub> as communication partners. The time complexity of Algorithm 2 is *O*(*n*), where *n* is the number of nodes.

After the reference node is selected, each node communicates and shares information of all event blocks known by them. A node creates an event block by referring to the top event block of the reference node. The Lachesis protocol works and communicates asynchronously. This allows a node to create an event block asynchronously even when another node creates an event block. The communication between nodes does not allow simultaneous communication with the same node.

<embed src="node_selection.pdf" height="264" />

Figure \[fig:node\_selection\] shows an example of the node selection in Lachesis protocol. In this example, there are five nodes (*A*, *B*, *C*, *D*, and *E*) and each node generates the first event blocks, called leaf events. All nodes share other leaf events with each other. In the first step, node *A* generates new event block *a*<sub>1</sub>. Then node *A* calculates the cost function to connect other nodes. In this initial situation, all nodes have one event block called leaf event, thus the height vector and the in-degree vector in node *A* has same values. In other words, the heights of each node are 1 and in-degrees are 0. Node *A* randomly select the other two nodes and connects *a*<sub>1</sub> to the top two event blocks from the other two nodes. Step 2 shows the situation after connections. In this example, node *A* select node *B* and *C* to connect *a*<sub>1</sub> and the event block *a*<sub>1</sub> is connected to the top event blocks of node *B* and *C*. Node *A* only knows the situation of the step 2.

After that, in the example, node *B* generates a new event block *b*<sub>1</sub> and also calculates the cost function. *B* randomly select the other two nodes; *A*, and *D*, since *B* only has information of the leaf events. Node *B* requests to *A* and *D* to connect *b*<sub>1</sub>, then nodes *A* and *D* send information for their top event blocks to node *B* as response. The top event block of node *A* is *a*<sub>1</sub> and node *D* is the leaf event. The event block *b*<sub>1</sub> is connected to *a*<sub>1</sub> and leaf event from node *D*. Step 4 shows these connections.

Event block creation
--------------------

In the Lachesis protocol, every node can create an event block. Each event block refers to other *k* event blocks using their hash values. In the Lachesis protocol, a new event block refers to *k*-neighbor event blocks under the following conditions:

1.  Each of the *k* reference event blocks is the top event blocks of its own node.

2.  One reference should be made to a self-ref that references to an event block of the same node.

3.  The other *k*-1 reference refers to the other *k*-1 top event nodes on other nodes.

<embed src="Ex_eventblock.pdf" style="width:100.0%" />

Figure \[fig:ex\_ebc\] shows the example of an event block creation with a flag table. In this example the recent created event block is *b*<sub>1</sub> by node *B*. The figure shows the node structure of node *B*. We omit the other information such as height and in-degree vectors since we only focus on the change of the flag table with the event block creation in this example. The flag table of *b*<sub>1</sub> in Figure \[fig:ex\_ebc\] is updated with the information of the previous connected event blocks *a*<sub>1</sub>, *b*<sub>0</sub>, and *c*<sub>1</sub>. Thus, the set of the flag table is the results of OR operation among the three root sets for *a*1 (*a*<sub>0</sub>, *b*<sub>0</sub>, and *c*<sub>0</sub>), *b*<sub>0</sub> (*b*<sub>0</sub>), and *c*<sub>1</sub> (*b*<sub>0</sub>, *c*<sub>0</sub>, and *d*<sub>0</sub>).

<embed src="sync.pdf" height="226" />

Figure \[fig:communication process\], shows the communication process is divided into five steps for two nodes to create an event block. Simply, a node *A* requests to *B*. then, *B* responds to *A* directly.

Topological ordering of events using Lamport timestamps
-------------------------------------------------------

Every node has a physical clock and it needs physical time to create an event block. However, for consensus, Lachesis protocols relies on a logical clock for each node. For the purpose, we use *“Lamport timestamps”* to determine the time ordering between event blocks in a asynchronous distributed system.

<embed src="Lamport_timestamps.pdf" style="width:70.0%" />

The Lamport timestamps algorithm is as follows:

1.  Each node increments its count value before creating an event block.

2.  When sending a message include its count value, receiver should consider which sender’s message is received and increments its count value.

3.  If current counter is less than or equal to the received count value from another node, then the count value of the recipient is updated.

4.  If current counter is greater than the received count value from another node, then the current count value is updated.

We use the Lamport’s algorithm to enforce a topological ordering of event blocks and use it in the Atropos selection algorithm.

Since an event block is created based on logical time, the sequence between each event blocks is immediately determined. Because the Lamport timestamps algorithm gives a partial order of all events, the whole time ordering process can be used for Byzantine fault tolerance.

Domination Relation
-------------------

Here, we introduce a new idea that extends the concept of domination.
For a vertex *v* in a DAG *G*, let *G*\[*v*\]=(*V*<sub>*v*</sub>, *E*<sub>*v*</sub>) denote an induced-subgraph of *G* such that *V*<sub>*v*</sub> consists of all ancestors of *v* including *v*, and *E*<sub>*v*</sub> is the induced edges of *V*<sub>*v*</sub> in *G*.
For a set *S* of vertices, an event *v*<sub>*d*</sub> $\\frac{2}{3}$-dominates *S* if there are more than 2/3 of vertices *v*<sub>*x*</sub> in *S* such that *v*<sub>*d*</sub> dominates *v*<sub>*x*</sub>. Recall that *R*<sub>1</sub> is the set of all leaf vertices in *G*. The $\\frac{2}{3}$-dom set *D*<sub>0</sub> is the same as the set *R*<sub>1</sub>.The $\\frac{2}{3}$-dom set *D*<sub>*i*</sub> is defined as follows:
A vertex *v*<sub>*d*</sub> belongs to a $\\frac{2}{3}$-dom set within the graph *G*\[*v*<sub>*d*</sub>\], if *v*<sub>*d*</sub> $\\frac{2}{3}$-dominates *R*<sub>1</sub>. The $\\frac{2}{3}$-dom set *D*<sub>*k*</sub> consists of all roots *d*<sub>*i*</sub> such that *d*<sub>*i*</sub> ∉ *D*<sub>*i*</sub>, ∀ *i* = 1..(*k*-1), and *d*<sub>*i*</sub> $\\frac{2}{3}$-dominates *D*<sub>*i* − 1</sub>.
The $\\frac{2}{3}$-dom set *D*<sub>*i*</sub> is the same with the root set *R*<sub>*i*</sub>, for all nodes.

Examples of domination relation in DAGs
---------------------------------------

This section gives several examples of DAGs and the domination relation between their event blocks.

(a)<img src="domtrees.png" title="fig:" alt="Examples of OPERA chain and dominator tree" />
(b)<img src="domtrees_add1event.png" title="fig:" alt="Examples of OPERA chain and dominator tree" />

Figure \[fig:domtrees1\] shows an examples of a DAG and dominator trees.

<img src="domset.png" alt="An example of OPERA chain and its 2/3 domination graph. The \frac{2}{3}-dom sets are shown in grey." />

Figure \[fig:domset1\] depicts an example of a DAG and 2/3 dom sets.

(a) <img src="deptrees.png" title="fig:" alt="An example of dependency graphs on individual nodes. From (a)-(c) there is one new event block appended. There is no fork, the simplified dependency graphs become trees." />
(b) <img src="deptrees_mod_add1event.png" title="fig:" alt="An example of dependency graphs on individual nodes. From (a)-(c) there is one new event block appended. There is no fork, the simplified dependency graphs become trees." />
(c) <img src="deptrees_mod_add2event.png" title="fig:" alt="An example of dependency graphs on individual nodes. From (a)-(c) there is one new event block appended. There is no fork, the simplified dependency graphs become trees." />

Figure \[fig:deptreesmod1\] shows an example an dependency graphs. On each row, the left most figure shows the latest OPERA chain. The left figures on each row depict the dependency graphs of each node, which are in their compact form. When no fork presents, each of the compact dependency graphs is a tree.

(a)<img src="deptrees_fork.png" title="fig:" alt="An example of a pair of fork events in an OPERA chain. The fork events are shown in red and green. The OPERA chains from (a) to (d) are different by adding one single event at a time." />
(b)<img src="deptrees_fork_add1event.png" title="fig:" alt="An example of a pair of fork events in an OPERA chain. The fork events are shown in red and green. The OPERA chains from (a) to (d) are different by adding one single event at a time." />
(c)<img src="deptrees_fork_add2event.png" title="fig:" alt="An example of a pair of fork events in an OPERA chain. The fork events are shown in red and green. The OPERA chains from (a) to (d) are different by adding one single event at a time." />

Figure \[fig:deptreesfork1\] shows an example of a pair of fork events. Each row shows an OPERA chain (left most) and the compact dependency graphs on each node (right). The fork events are shown in red and green vertices

Root Selection
--------------

All nodes can create event blocks and an event block can be a root when satisfying specific conditions. Not all event blocks can be roots. First, the first created event blocks are themselves roots. These leaf event blocks form the first root set *R*<sub>*S*<sub>1</sub></sub> of the first frame *f*<sub>1</sub>. If there are total *n* nodes and these nodes create the event blocks, then the cardinality of the first root set |*R*<sub>*S*<sub>1</sub></sub>| is *n*. Second, if an event block *e* can reach at least 2n/3 roots, then *e* is called a root. This event *e* does not belong to *R*<sub>*S*1</sub>, but the next root set *R*<sub>*S*<sub>2</sub></sub> of the next frame *f*<sub>2</sub>. Thus, excluding the first root set, the range of cardinality of root set *R*<sub>*S*<sub>*k*</sub></sub> is 2*n*/3 &lt; |*R*<sub>*S*<sub>*k*</sub></sub>|≤*n*. The event blocks including *R*<sub>*S*<sub>*k*</sub></sub> before *R*<sub>*S*<sub>*k* + 1</sub></sub> is in the frame *f*<sub>*k*</sub>. The roots in *R*<sub>*S*<sub>*k* + 1</sub></sub> does not belong to the frame *f*<sub>*k*</sub>. Those are included in the frame *f*<sub>*k* + 1</sub> when a root belonging to *R*<sub>*S*<sub>*k* + 2</sub></sub> occurs.

We introduce the use of a flag table to quickly determine whether a new event block becomes a root. Each node maintains a flag table of the top event block. Every event block that is newly created is assigned *k* hashes for its *k* referenced event blocks. We apply an *O**R* operation on each set in the flag table of the referenced event blocks.

\[H\] <embed src="Root_selection.pdf" title="fig:" height="226" />

Figure \[fig:ex\_ft\] shows an example of how to use flag tables to determine a root. In this example, *b*<sub>1</sub> is the most recently created event block. We apply an *O**R* operation on each set of the flag tables for *b*<sub>1</sub>’s *k* referenced event blocks. The result is the flag table of *b*<sub>1</sub>. If the cardinality of the root set in *b*<sub>1</sub>’s flag table is more than 2*n*/3, *b*<sub>1</sub> is a root. In this example, the cardinality of the root set in *b*<sub>1</sub> is 4, which is greater than 2*n*/3 (*n*=5). Thus, *b*<sub>1</sub> becomes root. In this example, *b*<sub>1</sub> is added to frame *f*<sub>2</sub> since *b*<sub>1</sub> becomes new root.

The root selection algorithm is as follows:

1.  The first event blocks are considered as roots.

2.  When a new event block is added in the OPERA chain (DAG), we check whether the event block is a root by applying an *O**R* operation on each set of the flag tables connected to the new event block. If the cardinality of the root set in the flag table for the new event block is more than 2n/3, the new event block becomes a root.

3.  When a new root appears on the OPERA chain, nodes update their frames. If one of the new event blocks becomes a root, all nodes that share the new event block add the hash value of the event block to their frames.

4.  The new root set is created if the cardinality of the previous root set *R*<sub>*S*<sub>*p*</sub></sub> is more than 2n/3 and the new event block can reach 2*n*/3 roots in *R*<sub>*S*<sub>*p*</sub></sub>.

5.  When the new root set *R*<sub>*S*<sub>*k* + 1</sub></sub> is created, the event blocks from the previous root set *R*<sub>*S*<sub>*k*</sub></sub> to before *R*<sub>*S*<sub>*k* + 1</sub></sub> belong to the frame *f*<sub>*k*</sub>.

Clotho Selection
----------------

A Clotho is a root that satisfies the Clotho creation conditions. Clotho creation conditions are that more than 2n/3 nodes know the root and a root knows this information.

In order for a root *r* in frame *f*<sub>*i*</sub> to become a Clotho, *r* must be reached by more than n/3 roots in the frame *f*<sub>*i* + 1</sub>. Based on the definition of the root, each root reaches more than 2n/3 roots in previous frames. If more than n/3 roots in the frame *f*<sub>*i* + 1</sub> can reach *r*, then *r* is spread to all roots in the frame *f*<sub>*i* + 2</sub>. It means that all nodes know the existence of *r*. If we have any root in the frame *f*<sub>*i* + 3</sub>, a root knows that *r* is spread to more than 2n/3 nodes. It satisfies Clotho creation conditions.

\[H\] <embed src="frame4.pdf" title="fig:" style="width:50.0%" />

In the example in Figure \[fig:frame4\], n is 5 and each circle indicates a root in a frame. Each arrow means one root can reach (happened-before) to the previous root. Each root has 4 or 5 arrows (out-degree) since n is 5 (more than 2n/3 ≥ 4). *b*<sub>1</sub> and *c*<sub>2</sub> in frame *f*<sub>2</sub> are roots that can reach *c*<sub>0</sub> in frame *f*<sub>1</sub>. *d*<sub>1</sub> and *e*<sub>1</sub> also can reach *c*<sub>0</sub>, but we only marked *b*<sub>1</sub> and *c*<sub>2</sub> (when n is 5, more than n/3 ≥ 2) since we show at least more than n/3 conditions in this example. And it was marked with a blue bold arrow (Namely, the roots that can reach root *c*<sub>0</sub> have the blue bold arrow). In this situation, an event block must be able to reach *b*<sub>1</sub> or *c*<sub>2</sub> in order to become a root in frame *f*<sub>3</sub> (In our example, n=5, more than n/3 ≥ 2, and more than 2n/3 ≥ 4. Thus, to be a root, either must be reached). All roots in frame *f*<sub>3</sub> reach *c*<sub>0</sub> in frame *f*<sub>1</sub>.

To be a root in frame *f*<sub>4</sub>, an event block must reach more than 2n/3 roots in frame *f*<sub>3</sub> that can reach *c*<sub>0</sub>. Therefore, if any of the root in frame *f*<sub>4</sub> exists, the root must have happened-before more than 2n/3 roots in frame *f*<sub>3</sub>. Thus, the root of *f*<sub>4</sub> knows that *c*<sub>0</sub> is spread over more than 2n/3 of the entire nodes. Thus, we can select *c*<sub>0</sub> as Clotho.

<embed src="Clotho_fig1.pdf" style="width:80.0%" />

Figure \[fig:Clotho\] shows an example of a Clotho. In this example, all roots in the frame *f*<sub>1</sub> have happened-before more than n/3 roots in the frame *f*<sub>2</sub>. We can select all roots in the frame *f*<sub>1</sub> as Clotho since the recent frame is *f*<sub>4</sub>.

\[H\]

**Input**: a root *r* *c*.*i**s*\_*c**l**o**t**h**o* ← *n**i**l* *c*.*y**e**s* ← 0 c.yes ← c.yes + 1 *c*.*i**s*\_*c**l**o**t**h**o* ← *y**e**s*

Algorithm \[al:acs\] shows the pseudo code for Clotho selection. The algorithm takes a root *r* as input. Line 4 and 5 set *c*.*i**s*\_*c**l**o**t**h**o* and *c*.*y**e**s* to *n**i**l* and 0 respectively. Line 6-8 checks whether any root *c*′ in *f**r**a**m**e*(*i* − 3, *r*) has happened-before with the 2n/3 condition *c* where *i* is the current frame. In line 9-10, if the number of roots in *f**r**a**m**e*(*i* − 2, *r*) which happened-before *c* is more than 2*n*/3, the root *c* is set as a Clotho. The time complexity of Algorithm 3 is *O*(*n*<sup>2</sup>), where *n* is the number of nodes.

<embed src="Clotho_fig2.pdf" />

Figure \[fig:ClothoNodeA\] shows the state of node *A* when a Clotho is selected. In this example, node *A* knows all roots in the frame *f*<sub>1</sub> become Clotho’s. Node *A* prunes unnecessary information on its own structure. In this case, node *A* prunes the root set in the frame *f*<sub>1</sub> since all roots in the frame *f*<sub>1</sub> become Clotho and the Clotho Check list stores the Clotho information.

Atropos Selection
-----------------

Atropos selection algorithm is the process in which the candidate time generated from Clotho selection is shared with other nodes, and each root re-selects candidate time repeatedly until all nodes have same candidate time for a Clotho.

After a Clotho is nominated, each node then computes a candidate time of the Clotho. If there are more than two-thirds of the nodes that compute the same value for candidate time, that time value is recorded. Otherwise, each node reselects candidate time. By the reselection process, each node reaches time consensus for candidate time of Clotho as the OPERA chain (DAG) grows. The candidate time reaching the consensus is called Atropos consensus time. After Atropos consensus time is computed, the Clotho is nominated to Atropos and each node stores the hash value of Atropos and Atropos consensus time in Main-Chain (blockchain). The Main-chain is used for time order between event blocks. The proof of Atropos consensus time selection is shown in the section \[se:proof\].

<embed src="Atropos_fig1.pdf" height="302" />

Figure \[fig:Atropos\] shows the example of Atropos selection. In Figure \[fig:Clotho\], all roots in the frame *f*<sub>1</sub> are selected as Clotho through the existence of roots in the frame *f*<sub>4</sub>. Each root in the frame *f*<sub>5</sub> computes candidate time using timestamps of reachable roots in the frame *f*<sub>4</sub>. Each root in the frame *f*<sub>5</sub> stores the candidate time to min-max value space. The root *r*<sub>6</sub> in the frame *f*<sub>6</sub> can reach more than 2n/3 roots in *f*<sub>5</sub> and *r*<sub>6</sub> can know the candidate time of the reachable roots that *f*<sub>5</sub> takes. If *r*<sub>6</sub> knows the same candidate time than more than 2n/3, we select the candidate time as Atropos consensus time. Then all Clotho in the frame *f*<sub>1</sub> become Atropos.

<embed src="Atropos_fig2.pdf" />

Figure \[fig:Atropos\_Node\] shows the state of node *B* when Atropos is selected. In this example, node *B* knows all roots in the frame *f*<sub>1</sub> become Atropos. Then node *B* prunes information of the frame *f*<sub>1</sub> in clotho check list since all roots in the frame *f*<sub>1</sub> become Atropos and main chain stores Atropos information.

\[H\]

**Input**: *c*.*C**l**o**t**h**o* in frame *f*<sub>*i*</sub> *c*.*c**o**n**s**e**n**s**u**s*\_*t**i**m**e* ← *n**i**l* *m* ← the index of the last frame *f*<sub>*m*</sub> *R* ← be the Root set *R*<sub>*S*<sub>*i* + *d*</sub></sub> in frame *f*<sub>*i* + *d*</sub> *r*.*t**i**m**e*(*c*) ← *r*.*l**a**m**p**o**r**t*\_*t**i**m**e* s ← the set of Root in *f*<sub>*j* − 1</sub> that *r* can be happened-before with 2n/3 condition t ← RESELECTION(s, *c*) k ← the number of root having *t* in *s* *c*.*c**o**n**s**e**n**s**u**s*\_*t**i**m**e* ← *t* *r*.*t**i**m**e*(*c*) ← *t* *r*.*t**i**m**e*(*c*) ← *t* *r*.*t**i**m**e*(*c*) ← the minimum value in *s*

\[H\]

<span> ALG@cmd@@L @Function @currentfunction<span>Reselection</span></span> <span> bsphack @writeauxout<span> </span> esphack </span> **Input**: Root set *R*, and Clotho *c* **Output**: candidate time *t* *τ* ← set of all *t*<sub>*i*</sub> = *r*.*t**i**m**e*(*c*) for all *r* in *R* *D* ← set of tuples (*t*<sub>*i*</sub>, *c*<sub>*i*</sub>) computed from *τ*, where *c*<sub>*i*</sub> = *c**o**u**n**t*(*t*<sub>*i*</sub>) *m**a**x*\_*c**o**u**n**t* ← *m**a**x*(*c*<sub>*i*</sub>) *t* ← *i**n**f**i**n**i**t**e* *t* ← *t*<sub>*i*</sub> **return** *t*

Algorithm \[al:atc\] and \[al:resel\] show pseudo code of Atropos consensus time selection and Consensus time reselection. In Algorithm \[al:atc\], at line 6, *d* saves the deference of relationship between root set of *c* and *w*. Thus, line 8 means that *w* is one of the elements in root set of the frame *f*<sub>*i* + 3</sub>, where the frame *f*<sub>*i*</sub> includes *c*. Line 10, each root in the frame *f*<sub>*j*</sub> selects own Lamport timestamp as candidate time of *c* when they confirm root *c* as Cltoho. In line 12, 13, and 14, *s*, *t*, and *k* save the set of root that *w* can be happened-before with 2n/3 condition *c*, the result of *R**E**S**E**L**E**C**T**I**O**N* function, and the number of root in *s* having *t*. Line 15 is checking whether there is a difference as much as *h* between *i* and *j* where *h* is a constant value for minimum selection frame. Line 16-20 is checking whether more than two-thirds of root in the frame *f*<sub>*j* − 1</sub> nominate the same candidate time. If two-thirds of root in the frame *f*<sub>*j* − 1</sub> nominate the same candidate time, the root *c* is assigned consensus time as *t*. Line 22 is minimum selection frame. In minimum selection frame, minimum value of candidate time is selected to reach byzantine agreement. Algorithm \[al:resel\] operates in the middle of Algorithm \[al:atc\]. In Algorithm \[al:resel\], input is a root set *W* and output is a reselected candidate time. Line 4-5 computes the frequencies of each candidate time from all the roots in *W*. In line 6-11, a candidate time which is smallest time that is the most nomitated. The time complexity of Algorithm \[al:resel\] is *O*(*n*) where *n* is the number of nodes. Since Algorithm \[al:atc\] includes Algorithm \[al:resel\], the time complexity of Algorithm \[al:atc\] is *O*(*n*<sup>2</sup>) where *n* is the number of nodes.

In the Atropos Consensus Time Selection algorithm, nodes reach consensus agreement about candidate time of a Clotho without additional communication (i.e., exchanging candidate time) with each other. Each node communicates with each other through the Lachesis protocol, the OPERA chain of all nodes grows up into same shape. This allows each node to know the candidate time of other nodes based on its OPERA chain and reach a consensus agreement. The proof that the agreement based on OPERA chain become agreement in action is shown in the section \[se:proof\].

Atropos can be determined by the consensus time of each Clotho. It is an event block that is determined by finality and is non-modifiable. Furthermore, all event blocks can be reached from Atropos guarantee finality.

Lachesis Consensus
------------------

<img src="pBFTtoPath" alt="Consensus Method in a DAG (combines chain with consensus process of pBFT)" height="264" />

Figure \[fig:pBFTtoPath\] illustrates how consensus is reached through the domination relation in the OPERA chain. In the figure, leaf set, denoted by *R*<sub>*s*0</sub>, consists of the first event blocks created by individual participant nodes. *V* is the set of event blocks that do not belong neither in *R*<sub>*s*0</sub> nor in any root set *R*<sub>*s**i*</sub>. Given a vertex *v* in *V* ∪ *R*<sub>*s**i*</sub>, there exists a path from *v* that can reach a leaf vertex *u* in *R*<sub>*s*0</sub>. Let *r*<sub>1</sub> and *r*<sub>2</sub> be root event blocks in root set *R*<sub>*s*1</sub> and *R*<sub>*s*2</sub>, respectively. *r*<sub>1</sub> is the block where a quorum or more blocks exist on a path that reaches a leaf event block. Every path from *r*<sub>1</sub> to a leaf vertex will contain a vertex in *V*<sub>1</sub>. Thus, if there exists a vertex *r* in *V*<sub>1</sub> such that *r* is created by more than a quorum of participants, then *r* is already included in *R*<sub>*s*1</sub>. Likewise, *r*<sub>2</sub> is a block that can be reached for *R*<sub>*s*1</sub> including *r*<sub>1</sub> through blocks made by a quorum of participants. For all leaf event blocks that could be reached by *r*<sub>1</sub>, they are connected with more than quorum participants through the presence of *r*<sub>1</sub>. The existence of the root *r*<sub>2</sub> shows that information of *r*<sub>1</sub> is connected with more than a quorum. This kind of a path search allows the chain to reach consensus in a similar manner as the pBFT consensus processes. It is essential to keep track of the blocks satisfying the pBFT consensus process for quicker path search; our OPERA chain and Main-chain keep track of these blocks.

The sequential order of each event block is an important aspect for Byzantine fault tolerance. In order to determine the pre-and-post sequence between all event blocks, we use Atropos consensus time, Lamport timestamp algorithm and the hash value of the event block.

<embed src="topological_ordering_rule.pdf" style="width:100.0%" />

First, when each node creates event blocks, they have a logical timestamp based on Lamport timestamp. This means that they have a partial ordering between the relevant event blocks. Each Clotho has consensus time to the Atropos. This consensus time is computed based on the logical time nominated from other nodes at the time of the 2n/3 agreement.

Each event block is based on the following three rules to reach an agreement:

1.  If there are more than one Atropos with different times on the same frame, the event block with smaller consensus time has higher priority.

2.  If there are more than one Atropos having any of the same consensus time on the same frame, determine the order based on the own logical time from Lamport timestamp.

3.  When there are more than one Atropos having the same consensus time, if the local logical time is same, a smaller hash value is given priority through hash function.

<embed src="topological_ordering.pdf" style="width:90.0%" />

Figure \[fig:sequence of operachain\] shows the part of OPERA chain in which the final consensus order is determined based on these 3 rules. The number represented by each event block is a logical time based on Lamport timestamp. Final topological consensus order containing the event blocks are based on agreement from the apropos. Based on each Atropos, they will have different colors depending on their range.

Detecting Forks
---------------

<span>  ***(Fork)*** <span>A pair of events (*v*<sub>*x*</sub>, *v*<sub>*y*</sub>) is a fork if *v*<sub>*x*</sub> and *v*<sub>*y*</sub> have the same creator, but neither is a self-ancestor of the other. Denoted by *v*<sub>*x*</sub> ⋔ *v*<sub>*y*</sub>.</span></span>

For example, let *v*<sub>*z*</sub> be an event in node *n*<sub>1</sub> and two child events *v*<sub>*x*</sub> and *v*<sub>*y*</sub> of *v*<sub>*z*</sub>. if *v*<sub>*x*</sub>↪<sup>*s*</sup>*v*<sub>*z*</sub>, *v*<sub>*y*</sub>↪<sup>*s*</sup>*v*<sub>*z*</sub>, $v\_x \\not {\\hookrightarrow^{s}}v\_y$, $v\_y \\not {\\hookrightarrow^{s}}v\_z$, then (*v*<sub>*x*</sub>, *v*<sub>*y*</sub>) is a fork. The fork relation is symmetric; that is *v*<sub>*x*</sub> ⋔ *v*<sub>*y*</sub> iff *v*<sub>*y*</sub> ⋔ *v*<sub>*x*</sub>.

By definition, (*v*<sub>*x*</sub>, *v*<sub>*y*</sub>) is a fork if *c**r*(*v*<sub>*x*</sub>)=*c**r*(*v*<sub>*y*</sub>), $v\_x \\not {\\hookrightarrow^{a}}v\_y$ and $v\_y \\not {\\hookrightarrow^{a}}v\_x$. Using Happened-Before, the second part means $v\_x \\not \\rightarrow v\_y$ and $v\_y \\not \\rightarrow v\_x$. By definition of concurrent, we get *v*<sub>*x*</sub> ∥ *v*<sub>*y*</sub>.

If there is a fork *v*<sub>*x*</sub> ⋔ *v*<sub>*y*</sub>, then *v*<sub>*x*</sub> and *v*<sub>*y*</sub> cannot both be roots on honest nodes.

Here, we show a proof by contradiction. Any honest node cannot accept a fork so *v*<sub>*x*</sub> and *v*<sub>*y*</sub> cannot be roots on the same honest node. Now we prove a more general case. Suppose that both *v*<sub>*x*</sub> is a root of *n*<sub>*x*</sub> and *v*<sub>*y*</sub> is root of *n*<sub>*y*</sub>, where *n*<sub>*x*</sub> and *n*<sub>*y*</sub> are honest nodes. Since *v*<sub>*x*</sub> is a root, it reached events created by more than 2/3 of member nodes. Similarly, *v*<sub>*y*</sub> is a root, it reached events created by more than 2/3 of member nodes. Thus, there must be an overlap of more than *n*/3 members of those events in both sets. Since we assume less than *n*/3 members are not honest, so there must be at least one honest member in the overlap set. Let *n*<sub>*m*</sub> be such an honest member. Because *n*<sub>*m*</sub> is honest, *n*<sub>*m*</sub> does not allow the fork.

Conclusion
==========

We further optimize the OPERA chain and Main-chain for faster consensus. By using Lamport timestamps and domination relation, the topological ordering of event blocks in OPERA chain and Main chain is more intuitive and reliable in distributed system.

We have presented a formal semantics for Lachesis protocol in Section \[se:lca\]. Our formal proof of pBFT for our Lachesis protocol is given in Section \[se:proof\]. Our work is the first that studies such concurrent common knowledge sematics  and dominator relationships in DAG-based protocols.

Appendix
========

Preliminaries
-------------

The history of a Lachesis protocol can be represented by a directed acyclic graph *G* = (*V*, *E*), where *V* is a set of vertices and *E* is a set of edges. Each vertex in a row (node) represents an event. Time flows left-to-right of the graph, so left vertices represent earlier events in history. A path *p* in *G* is a sequence of vertices (*v*<sub>1</sub>, *v*<sub>2</sub>, …, *v*<sub>*k*</sub>) by following the edges in *E*. Let *v*<sub>*c*</sub> be a vertex in *G*. A vertex *v*<sub>*p*</sub> is the *parent* of *v*<sub>*c*</sub> if there is an edge from *v*<sub>*p*</sub> to *v*<sub>*c*</sub>. A vertex *v*<sub>*a*</sub> is an *ancestor* of *v*<sub>*c*</sub> if there is a path from *v*<sub>*a*</sub> to *v*<sub>*c*</sub>.

Each machine that participates in the Lachesis protocol is called a node.

Let *n* denote the total number of nodes.

Each node can create event blocks, send (receive) messages to (from) other nodes.

An event block is a vertex of the OPERA chain.

Suppose a node *n*<sub>*i*</sub> creates an event *v*<sub>*c*</sub> after an event *v*<sub>*s*</sub> in *n*<sub>*i*</sub>. Each event block has exactly *k* references. One of the references is self-reference, and the other *k*-1 references point to the top events of *n*<sub>*i*</sub>’s *k*-1 peer nodes.

A node *n*<sub>*i*</sub> has *k* peer nodes.

An event *v* is a top event of a node *n*<sub>*i*</sub> if there is no other event in *n*<sub>*i*</sub> referencing *v*.

An event *v*<sub>*s*</sub> is called “self-ref" of event *v*<sub>*c*</sub>, if the self-ref hash of *v*<sub>*c*</sub> points to the event *v*<sub>*s*</sub>. Denoted by *v*<sub>*c*</sub>↪<sup>*s*</sup>*v*<sub>*s*</sub>.

An event *v*<sub>*r*</sub> is called “ref" of event *v*<sub>*c*</sub> if the reference hash of *v*<sub>*c*</sub> points to the event *v*<sub>*r*</sub>. Denoted by *v*<sub>*c*</sub>↪<sup>*r*</sup>*v*<sub>*r*</sub>.

For simplicity, we can use ↪ to denote a reference relationship (either ↪<sup>*r*</sup> or ↪<sup>*s*</sup>).

An event block *v*<sub>*a*</sub> is self-ancestor of an event block *v*<sub>*c*</sub> if there is a sequence of events such that *v*<sub>*c*</sub>↪<sup>*s*</sup>*v*<sub>1</sub>↪<sup>*s*</sup>…↪<sup>*s*</sup>*v*<sub>*m*</sub>↪<sup>*s*</sup>*v*<sub>*a*</sub>. Denoted by *v*<sub>*c*</sub>↪<sup>*s**a*</sup>*v*<sub>*a*</sub>.

An event block *v*<sub>*a*</sub> is an ancestor of an event block *v*<sub>*c*</sub> if there is a sequence of events such that *v*<sub>*c*</sub> ↪ *v*<sub>1</sub> ↪ … ↪ *v*<sub>*m*</sub> ↪ *v*<sub>*a*</sub>. Denoted by *v*<sub>*c*</sub>↪<sup>*a*</sup>*v*<sub>*a*</sub>.

For simplicity, we simply use *v*<sub>*c*</sub>↪<sup>*a*</sup>*v*<sub>*s*</sub> to refer both ancestor and self-ancestor relationship, unless we need to distinguish the two cases.

OPERA chain is a DAG graph *G* = (*V*, *E*) consisting of *V* vertices and *E* edges. Each vertex *v*<sub>*i*</sub> ∈ *V* is an event block. An edge (*v*<sub>*i*</sub>, *v*<sub>*j*</sub>)∈*E* refers to a hashing reference from *v*<sub>*i*</sub> to *v*<sub>*j*</sub>; that is, *v*<sub>*i*</sub> ↪ *v*<sub>*j*</sub>.

### Domination relation

Then we define the domination relation for event blocks. To begin with, we first introduce pseudo vertices, *top* and *bot*, of the DAG OPERA chain *G*.

A pseudo vertex, called top, is the parent of all top event blocks. Denoted by ⊤.

A pseudo vertex, called bottom, is the child of all leaf event blocks. Denoted by ⊥.

With the pseudo vertices, we have ⊥ happened before all event blocks. Also all event blocks happened before ⊤. That is, for all event *v*<sub>*i*</sub>, ⊥ → *v*<sub>*i*</sub> and *v*<sub>*i*</sub> → ⊤.

An event *v*<sub>*d*</sub> dominates an event *v*<sub>*x*</sub> if every path from ⊤ to *v*<sub>*x*</sub> must go through *v*<sub>*d*</sub>. Denoted by *v*<sub>*d*</sub> ≫ *v*<sub>*x*</sub>.

An event *v*<sub>*d*</sub> strictly dominates an event *v*<sub>*x*</sub> if *v*<sub>*d*</sub> ≫ *v*<sub>*x*</sub> and *v*<sub>*d*</sub> does not equal *v*<sub>*x*</sub>. Denoted by *v*<sub>*d*</sub>≫<sup>*s*</sup>*v*<sub>*x*</sub>.

A vertex *v*<sub>*d*</sub> is said “domfront” a vertex *v*<sub>*x*</sub> if *v*<sub>*d*</sub> dominates an immediate predecessor of *v*<sub>*x*</sub>, but *v*<sub>*d*</sub> does not strictly dominate *v*<sub>*x*</sub>. Denoted by *v*<sub>*d*</sub>≫<sup>*f*</sup>*v*<sub>*x*</sub>.

The dominance frontier of a vertex *v*<sub>*d*</sub> is the set of all nodes *v*<sub>*x*</sub> such that *v*<sub>*d*</sub>≫<sup>*f*</sup>*v*<sub>*x*</sub>. Denoted by *D**F*(*v*<sub>*d*</sub>).

From the above definitions of domfront and dominance frontier, the following holds. If *v*<sub>*d*</sub>≫<sup>*f*</sup>*v*<sub>*x*</sub>, then *v*<sub>*x*</sub> ∈ *D**F*(*v*<sub>*d*</sub>).

Here, we introduce a new idea that extends the concept domination.

For a vertex *v* in a DAG *G*, let *G*\[*v*\]=(*V*<sub>*v*</sub>, *E*<sub>*v*</sub>) denote an induced-subgraph of *G* such that *V*<sub>*v*</sub> consists of all ancestors of *v* including *v*, and *E*<sub>*v*</sub> is the induced edges of *V*<sub>*v*</sub> in *G*.

For a set *S* of vertices, an event *v*<sub>*d*</sub> $\\frac{2}{3}$-dominates *S* if there are more than 2/3 of vertices *v*<sub>*x*</sub> in *S* such that *v*<sub>*d*</sub> dominates *v*<sub>*x*</sub>. Recall that *R*<sub>1</sub> is the set of all leaf vertices in *G*. The $\\frac{2}{3}$-dom set *D*<sub>0</sub> is the same as the set *R*<sub>1</sub>.The $\\frac{2}{3}$-dom set *D*<sub>*i*</sub> is defined as follows:

A vertex *v*<sub>*d*</sub> belongs to a $\\frac{2}{3}$-dom set within the graph *G*\[*v*<sub>*d*</sub>\], if *v*<sub>*d*</sub> $\\frac{2}{3}$-dominates *R*<sub>1</sub>. The $\\frac{2}{3}$-dom set *D*<sub>*k*</sub> consists of all roots *d*<sub>*i*</sub> such that *d*<sub>*i*</sub> ∉ *D*<sub>*i*</sub>, ∀ *i* = 1..(*k*-1), and *d*<sub>*i*</sub> $\\frac{2}{3}$-dominates *D*<sub>*i* − 1</sub>.

The $\\frac{2}{3}$-dom set *D*<sub>*i*</sub> is the same with the root set *R*<sub>*i*</sub>, for all nodes.

Proof of Lachesis Consensus Algorithm
-------------------------------------

This section presents a proof of liveness and safety of our Lachesis protocols. We aim to show that our consensus is Byzantine fault tolerant with a presumption that more than two-thirds of participants are reliable nodes. We first provide some definitions, lemmas and theorems. Then we validate the Byzantine fault tolerance.

### Proof of Byzantine Fault Tolerance for Lachesis Consensus Algorithm

An event block *v*<sub>*x*</sub> is said Happened-Immediate-Before an event block *v*<sub>*y*</sub> if *v*<sub>*x*</sub> is a (self-) ref of *v*<sub>*y*</sub>. Denoted by *v*<sub>*x*</sub> ↦ *v*<sub>*y*</sub>.

An event block *v*<sub>*x*</sub> is said Happened-Before an event block *v*<sub>*y*</sub> if *v*<sub>*x*</sub> is a (self-) ancestor of *v*<sub>*y*</sub>. Denoted by *v*<sub>*x*</sub> → *v*<sub>*y*</sub>.

The happens-before relation is the transitive closure of happens-immediately-before. Thus, an event *v*<sub>*x*</sub> happened before an event *v*<sub>*y*</sub> if one of the followings happens: (a) *v*<sub>*y*</sub>↪<sup>*s*</sup>*v*<sub>*x*</sub>, (b) *v*<sub>*y*</sub>↪<sup>*r*</sup>*v*<sub>*x*</sub>, or (c) *v*<sub>*y*</sub>↪<sup>*a*</sup>*v*<sub>*x*</sub>. We come up with the following proposition:

*v*<sub>*x*</sub> ↦ *v*<sub>*y*</sub> iff *v*<sub>*y*</sub> ↪ *v*<sub>*x*</sub> iff edge (*v*<sub>*y*</sub>, *v*<sub>*x*</sub>) ∈*E* of OPERA chain.

*v*<sub>*x*</sub> → *v*<sub>*y*</sub> iff *v*<sub>*y*</sub>↪<sup>*a*</sup>*v*<sub>*x*</sub>.

Two event blocks *v*<sub>*x*</sub> and *v*<sub>*y*</sub> are said concurrent if neither of them happened before the other. Denoted by *v*<sub>*x*</sub> ∥ *v*<sub>*y*</sub>.

Given two vertices *v*<sub>*x*</sub> and *v*<sub>*y*</sub> both contained in two OPERA chains *G*<sub>1</sub> and *G*<sub>2</sub> on two nodes. We have the following: (1) *v*<sub>*x*</sub> → *v*<sub>*y*</sub> in *G*<sub>1</sub> iff *v*<sub>*x*</sub> → *v*<sub>*y*</sub> in *G*<sub>2</sub>; (2) *v*<sub>*x*</sub> ∥ *v*<sub>*y*</sub> in *G*<sub>1</sub> iff *v*<sub>*x*</sub> ∥ *v*<sub>*y*</sub> in *G*<sub>2</sub>.

Below is some main definitions in Lachesis protocol.

The first created event block of a node is called a leaf event block.

\[def:root\] The leaf event block of a node is a root. When an event block *v* can reach more than 2*n*/3 of the roots in the previous frames, *v* becomes a root.

The set of all first event blocks (leaf events) of all nodes form the first root set *R*<sub>1</sub> (|*R*<sub>1</sub>| = *n*). The root set *R*<sub>*k*</sub> consists of all roots *r*<sub>*i*</sub> such that *r*<sub>*i*</sub> ∉ *R*<sub>*i*</sub>, ∀ *i* = 1..(*k*-1) and *r*<sub>*i*</sub> can reach more than 2n/3 other roots in the current frame, *i* = 1..(*k*-1).

Frame *f*<sub>*i*</sub> is a natural number that separates Root sets.

The root set at frame *f*<sub>*i*</sub> is denoted by *R*<sub>*i*</sub>.

\[dfn:conchains\] OPERA chains *G*<sub>1</sub> and *G*<sub>2</sub> are consistent iff for any event *v* contained in both chains, *G*<sub>1</sub>\[*v*\]=*G*<sub>2</sub>\[*v*\]. Denoted by *G*<sub>1</sub> ∼ *G*<sub>2</sub>.

When two consistent chains contain the same event *v*, both chains contain the same set of ancestors for *v*, with the same reference and self-ref edges between those ancestors:

\[thm:conchains\] All nodes have consistent OPERA chains.

If two nodes have OPERA chains containing event *v*, then they have the same *k* hashes contained within *v*. A node will not accept an event during a sync unless that node already has *k* references for that event, so both OPERA chains must contain *k* references for *v*. The cryptographic hashes are assumed to be secure, therefore the references must be the same. By induction, all ancestors of *v* must be the same. Therefore, the two OPERA chains are consistent.

If a node *n*<sub>*x*</sub> creates an event block *v*, then the creator of *v*, denoted by *c**r*(*v*), is *n*<sub>*x*</sub>.

The pair of events (*v*<sub>*x*</sub>, *v*<sub>*y*</sub>) is a fork if *v*<sub>*x*</sub> and *v*<sub>*y*</sub> have the same creator, but neither is a self-ancestor of the other. Denoted by *v*<sub>*x*</sub> ⋔ *v*<sub>*y*</sub>.

For example, let *v*<sub>*z*</sub> be an event in node *n*<sub>1</sub> and two child events *v*<sub>*x*</sub> and *v*<sub>*y*</sub> of *v*<sub>*z*</sub>. if *v*<sub>*x*</sub>↪<sup>*s*</sup>*v*<sub>*z*</sub>, *v*<sub>*y*</sub>↪<sup>*s*</sup>*v*<sub>*z*</sub>, $v\_x \\not {\\hookrightarrow^{s}}v\_y$, $v\_y \\not {\\hookrightarrow^{s}}v\_z$, then (*v*<sub>*x*</sub>, *v*<sub>*y*</sub>) is a fork. The fork relation is symmetric; that is *v*<sub>*x*</sub> ⋔ *v*<sub>*y*</sub> iff *v*<sub>*y*</sub> ⋔ *v*<sub>*x*</sub>.

*v*<sub>*x*</sub> ⋔ *v*<sub>*y*</sub> iff *c**r*(*v*<sub>*x*</sub>)=*c**r*(*v*<sub>*y*</sub>) and *v*<sub>*x*</sub> ∥ *v*<sub>*y*</sub>.

By definition, (*v*<sub>*x*</sub>, *v*<sub>*y*</sub>) is a fork if *c**r*(*v*<sub>*x*</sub>)=*c**r*(*v*<sub>*y*</sub>), $v\_x \\not {\\hookrightarrow^{a}}v\_y$ and $v\_y \\not {\\hookrightarrow^{a}}v\_x$. Using Happened-Before, the second part means $v\_x \\not \\rightarrow v\_y$ and $v\_y \\not \\rightarrow v\_x$. By definition of concurrent, we get *v*<sub>*x*</sub> ∥ *v*<sub>*y*</sub>.

(fork detection). If there is a fork *v*<sub>*x*</sub> ⋔ *v*<sub>*y*</sub>, then *v*<sub>*x*</sub> and *v*<sub>*y*</sub> cannot both be roots on honest nodes.

Here, we show a proof by contradiction. Any honest node cannot accept a fork so *v*<sub>*x*</sub> and *v*<sub>*y*</sub> cannot be roots on the same honest node. Now we prove a more general case. Suppose that both *v*<sub>*x*</sub> is a root of *n*<sub>*x*</sub> and *v*<sub>*y*</sub> is root of *n*<sub>*y*</sub>, where *n*<sub>*x*</sub> and *n*<sub>*y*</sub> are honest nodes. Since *v*<sub>*x*</sub> is a root, it reached events created by more than 2/3 of member nodes. Similary, *v*<sub>*y*</sub> is a root, it reached events created by more than 2/3 of member nodes. Thus, there must be an overlap of more than *n*/3 members of those events in both sets. Since we assume less than *n*/3 members are not honest, so there must be at least one honest member in the overlap set. Let *n*<sub>*m*</sub> be such an honest member. Because *n*<sub>*m*</sub> is honest, *n*<sub>*m*</sub> does not allow the fork. This contradicts the assumption. Thus, the lemma is proved.

Each node *n*<sub>*i*</sub> has an OPERA chain *G*<sub>*i*</sub>. We define a consistent chain from a sequence of OPERA chain *G*<sub>*i*</sub>.

A global consistent chain *G*<sup>*C*</sup> is a chain if *G*<sup>*C*</sup> ∼ *G*<sub>*i*</sub> for all *G*<sub>*i*</sub>.

We denote *G* ⊑ *G*′ to stand for *G* is a subgraph of *G*′.

∀*G*<sub>*i*</sub> (*G*<sup>*C*</sup> ⊑ *G*<sub>*i*</sub>).

∀*v* ∈ *G*<sup>*C*</sup> ∀*G*<sub>*i*</sub> (*G*<sup>*C*</sup>\[*v*\]⊑*G*<sub>*i*</sub>\[*v*\]).

(∀*v*<sub>*c*</sub> ∈ *G*<sup>*C*</sup>) (∀*v*<sub>*p*</sub> ∈ *G*<sub>*i*</sub>) ((*v*<sub>*p*</sub> → *v*<sub>*c*</sub>)⇒*v*<sub>*p*</sub> ∈ *G*<sup>*C*</sup>).

Now we state the following important propositions.

Two chains *G*<sub>1</sub> and *G*<sub>2</sub> are root consistent, if for every *v* contained in both chains, and *v* is a root of *j*-th frame in *G*<sub>1</sub>, then *v* is a root of *j*-th frame in *G*<sub>2</sub>.

If *G*<sub>1</sub> ∼ *G*<sub>2</sub>, then *G*<sub>1</sub> and *G*<sub>2</sub> are root consistent.

By consistent chains, if *G*<sub>1</sub> ∼ *G*<sub>2</sub> and *v* belongs to both chains, then *G*<sub>1</sub>\[*v*\] = *G*<sub>2</sub>\[*v*\]. We can prove the proposition by induction. For *j* = 0, the first root set is the same in both *G*<sub>1</sub> and *G*<sub>2</sub>. Hence, it holds for *j* = 0. Suppose that the proposition holds for every *j* from 0 to *k*. We prove that it also holds for *j*= *k* + 1. Suppose that *v* is a root of frame *f*<sub>*k* + 1</sub> in *G*<sub>1</sub>. Then there exists a set *S* reaching 2/3 of members in *G*<sub>1</sub> of frame *f*<sub>*k*</sub> such that ∀*u* ∈ *S* (*u* → *v*). As *G*<sub>1</sub> ∼ *G*<sub>2</sub>, and *v* in *G*<sub>2</sub>, then ∀*u* ∈ *S* (*u* ∈ *G*<sub>2</sub>). Since the proposition holds for *j*=*k*, As *u* is a root of frame *f*<sub>*k*</sub> in *G*<sub>1</sub>, *u* is a root of frame *f*<sub>*k*</sub> in *G*<sub>2</sub>. Hence, the set *S* of 2/3 members *u* happens before *v* in *G*<sub>2</sub>. So *v* belongs to *f*<sub>*k* + 1</sub> in *G*<sub>2</sub>. The proposition is proved.

From the above proposition, one can deduce the following:

*G*<sup>*C*</sup> is root consistent with *G*<sub>*i*</sub> for all nodes.

Thus, all nodes have the same consistent root sets, which are the root sets in *G*<sup>*C*</sup>. Frame numbers are consistent for all nodes.

For any top event *v* in both OPERA chains *G*<sub>1</sub> and *G*<sub>2</sub>, and *G*<sub>1</sub> ∼ *G*<sub>2</sub>, then the flag tables of *v* are consistent iff they are the same in both chains.

From the above lemmas, the root sets of *G*<sub>1</sub> and *G*<sub>2</sub> are consistent. If *v* contained in *G*<sub>1</sub>, and *v* is a root of *j*-th frame in *G*<sup>1</sup>, then *v* is a root of *j*-th frame in *G*<sub>*i*</sub>. Since *G*<sub>1</sub> ∼ *G*<sub>2</sub>, *G*<sub>1</sub>\[*v*\]=*G*<sub>2</sub>\[*v*\]. The reference event blocks of *v* are the same in both chains. Thus the flag tables of *v* of both chains are the same.

Thus, all nodes have consistent flag tables.

A root *r*<sub>*k*</sub> in the frame *f*<sub>*a* + 3</sub> can nominate a root *r*<sub>*a*</sub> as Clotho if more than 2n/3 roots in the frame *f*<sub>*a* + 1</sub> Happened-Before *r*<sub>*a*</sub> and *r*<sub>*k*</sub> Happened-Before the roots in the frame *f*<sub>*a* + 1</sub>.

\[lem:root\] For any root set *R*, all nodes nominate same root into Clotho.

Based on Theorem \[thm:same\], each node nominates a root into Clotho via the flag table. If all nodes have an OPERA chain with same shape, the values in flag table should be equal to each other in OPERA chain. Thus, all nodes nominate the same root into Clotho since the OPERA chain of all nodes has same shape.

\[lem:resel\] In the Reselection algorithm, for any Clotho, a root in OPERA chain selects the same consensus time candidate.

Based on Theorem \[thm:same\], if all nodes have an OPERA chain with the same partial shape, a root in OPERA chain selects the same consensus time candidate by the Reselection algorithm.

\[lem:fork\] If the pair of event blocks (x,y) is fork and a root has Happened-before the fork, this fork is detected in the Clotho selection process.

We show a proof by contradiction. Assume that no node can detect the fork in the Clotho selection process.

Assume that there is a root *r*<sub>*i*</sub> that becomes Clotho in *f*<sub>*i*</sub>, which was selected as Clotho by n/3 of the roots in *f*<sub>*i* + 1</sub>. More than 2n/3 roots in *f*<sub>*i* + 1</sub> should have happened-before by a root in *f*<sub>*i* + 2</sub>. If a pair (*v*<sub>*x*</sub>, *v*<sub>*y*</sub>) is fork, There are two cases: (1) assume that *r*<sub>*i*</sub> is one of *v*<sub>*x*</sub> and *v*<sub>*y*</sub>, (2) assume that *r*<sub>*i*</sub> can reach both *v*<sub>*x*</sub> and *v*<sub>*y*</sub>.

Our proof for both two cases is as follows.

Let *k* denote that the number of roots in *f*<sub>*i* + 1</sub> that can reach *r*<sub>*i*</sub> in *f*<sub>*i*</sub> (∴ n/3 &lt; *k*).

In order to select root in *f*<sub>*i* + 2</sub>, the root in *f*<sub>*i* + 2</sub> should reach more than 2n/3 of roots in *f*<sub>*i* + 1</sub> by Definition \[def:root\]. At the moment, assume that *l* is the number of roots in *f*<sub>*i* + 1</sub> that can be reached by the root in *f*<sub>*i* + 2</sub> (∴ 2n/3 &lt; *l*).

At this time, n &lt; k + l (∵ n/3 + 2n/3 &lt; *k* + *l*), there are (n - *k* + *l*) roots in frame *f*<sub>*i* + 1</sub> that should reach *r*<sub>*i*</sub> in *f*<sub>*i*</sub> and all roots in *f*<sub>*i* + 2</sub> should reach at least n – *k* + *l* of roots in *f*<sub>*i* + 1</sub>. It means that all roots in *f*<sub>*i* + 2</sub> know the existence of *r*<sub>*i*</sub>. Therefore, the node that generated all the roots of *f*<sub>*i* + 2</sub> detect the fork in the Clotho selection of *r*<sub>*i*</sub>, which contradicts the assumption.

It can be covered two cases. If *r*<sub>*i*</sub> is part of the fork, we can detect in *f*<sub>*i* + 2</sub>. If there is fork (*v*<sub>*x*</sub>, *v*<sub>*y*</sub>) that can be reached by *r*<sub>*i*</sub>, it also can be detected in *f*<sub>*i* + 2</sub> since we can detect the fork in the Clotho selection of *r*<sub>*i*</sub> and it indicates that all event blocks that can be reached by *r*<sub>*i*</sub> are detected by the roots in *f*<sub>*i* + 2</sub>.

For a root *v* happened-before a fork in OPERA chain, *v* must see the fork before becoming Clotho.

Suppose that a node creates two event blocks (*v*<sub>*x*</sub>, *v*<sub>*y*</sub>), which forms a fork. To create two Clothos that can reach both events, the event blocks should reach by more than 2n/3 nodes. Therefore, the OPERA chain can structurally detect the fork before roots become Clotho.

\[thm:same\] All nodes grows up into same consistent OPERA chain *G*<sup>*C*</sup>, which contains no fork.

Suppose that there are two event blocks *v*<sub>*x*</sub> and *v*<sub>*y*</sub> contained in both *G*<sub>1</sub> and *G*<sub>2</sub>, and their path between *v*<sub>*x*</sub> and *v*<sub>*y*</sub> in *G*<sub>1</sub> is not equal to that in *G*<sub>2</sub>. We can consider that the path difference between the nodes is a kind of fork attack. Based on Lemma \[lem:fork\], if an attacker forks an event block, each chain of *G*<sub>*i*</sub> and *G*<sub>2</sub> can detect and remove the fork before the Clotho is generated. Thus, any two nodes have consistent OPERA chain.

If the consensus time of Clotho is validated, the Clotho become an Atropos.

\[thm:ct\] Lachesis consensus algorithm guarantees to reach agreement for the consensus time.

For any root set *R* in the frame *f*<sub>*i*</sub>, time consensus algorithm checks whether more than 2n/3 roots in the frame *f*<sub>*i* − 1</sub> selects the same value. However, each node selects one of the values collected from the root set in the previous frame by the time consensus algorithm and Reselection process. Based on the Reselection process, the time consensus algorithm can reach agreement. However, there is a possibility that consensus time candidate does not reach agreement . To solve this problem, time consensus algorithm includes minimal selection frame per next *h* frame. In minimal value selection algorithm, each root selects minimum value among values collected from previous root set. Thus, the consensus time reaches consensus by time consensus algorithm.

\[thm:bft\] If the number of reliable nodes is more than 2*n*/3, event blocks created by reliable nodes must be assigned to consensus order.

In OPERA chain, since reliable nodes try to create event blocks by communicating with every other nodes continuously, reliable nodes will share the event block *x* with each other. If a root *y* in the frame *f*<sub>*i*</sub> Happened-Before event block *x* and more than n/3 roots in the frame *f*<sub>*i* + 1</sub> Happened-Before the root *y*, the root *y* will be nominated as Clotho and Atropos. Thus, event block *x* and root *y* will be assigned consensus time *t*.

For an event block, assigning consensus time means that the validated event block is shared by more than 2n/3 nodes. Therefore, malicious node cannot try to attack after the event blocks are assigned consensus time. When the event block *x* has consensus time *t*, it cannot occur to discover new event blocks with earlier consensus time than *t*. There are two conditions to be assigned consensus time earlier than *t* for new event blocks. First, a root *r* in the frame *f*<sub>*i*</sub> should be able to share new event blocks. Second, the more than 2n/3 roots in the frame *f*<sub>*i* + 1</sub> should be able to share *r*. Even if the first condition is satisfied by malicious nodes (e.g., parasite chain), the second condition cannot be satisfied since at least 2n/3 roots in the frame *f*<sub>*i* + 1</sub> are already created and cannot be changed. Therefore, after an event block is validated, new event blocks should not be participate earlier consensus time to OPERA chain.

Semantics of Lachesis protocol
------------------------------

This section gives the formal semantics of Lachesis consensus protocol. We use CCK model of an asynchronous system as the base of the semantics of our Lachesis protocol. Events are ordered based on Lamport’s happens-before relation. In particular, we use Lamport’s theory to describe global states of an asynchronous system.

We present notations and concepts, which are important for Lachesis protocol. In several places, we adapt the notations and concepts of CCK paper to suit our Lachesis protocol.

An asynchronous system consists of the following:

A process *p*<sub>*i*</sub> represents a machine or a node. The process identifier of *p*<sub>*i*</sub> is *i*. A set *P* = {1,...,*n*} denotes the set of process identifiers.

A process *i* can send messages to process *j* if there is a channel (*i*,*j*). Let *C* ⊆ {(*i*,*j*) s.t. *i*, *j* ∈ *P*} denote the set of channels.

A local state of a process *i* is denoted by *s*<sub>*j*</sub><sup>*i*</sup>.

A local state consists of a sequence of event blocks *s*<sub>*j*</sub><sup>*i*</sup> = *v*<sub>0</sub><sup>*i*</sup>, *v*<sub>1</sub><sup>*i*</sup>, …, *v*<sub>*j*</sub><sup>*i*</sup>.

In a DAG-based protocol, each *v*<sub>*j*</sub><sup>*i*</sup> event block is valid only the reference blocks exist exist before it. From a local state *s*<sub>*j*</sub><sup>*i*</sup>, one can reconstruct a unique DAG. That is, the mapping from a local state *s*<sub>*j*</sub><sup>*i*</sup> into a DAG is *injective* or one-to-one. Thus, for Lachesis, we can simply denote the *j*-th local state of a process *i* by the OPERA chain *g*<sub>*j*</sub><sup>*i*</sup> (often we simply use *G*<sub>*i*</sub> to denote the current local state of a process *i*).

An action is a function from one local state to another local state.

Generally speaking, an action can be either: a *s**e**n**d*(*m*) action where *m* is a message, a *r**e**c**e**i**v**e*(*m*) action, and an internal action. A message *m* is a triple ⟨*i*, *j*, *B*⟩ where *i* ∈ *P* is the sender of the message, *j* ∈ *P* is the message recipient, and *B* is the body of the message. Let *M* denote the set of messages. In Lachesis protocol, *B* consists of the content of an event block *v*. Semantics-wise, in Lachesis, there are two actions that can change a process’s local state: creating a new event and receiving an event from another process.

An event is a tuple ⟨*s*, *α*, *s*′⟩ consisting of a state, an action, and a state.

Sometimes, the event can be represented by the end state *s*′. The *j*-th event in history *h*<sub>*i*</sub> of process *i* is ⟨*s*<sub>*j* − 1</sub><sup>*i*</sup>, *α*, *s*<sub>*j*</sub><sup>*i*</sup>⟩, denoted by *v*<sub>*j*</sub><sup>*i*</sup>.

A local history *h*<sub>*i*</sub> of process *i* is a (possibly infinite) sequence of alternating local states — beginning with a distinguished initial state. A set *H*<sub>*i*</sub> of possible local histories for each process *i* in *P*.

The state of a process can be obtained from its initial state and the sequence of actions or events that have occurred up to the current state. In Lachesis protocol, we use append-only sematics. The local history may be equivalently described as either of the following:
*h*<sub>*i*</sub> = *s*<sub>0</sub><sup>*i*</sup>, *α*<sub>1</sub><sup>*i*</sup>, *α*<sub>2</sub><sup>*i*</sup>, *α*<sub>3</sub><sup>*i*</sup>…
*h*<sub>*i*</sub> = *s*<sub>0</sub><sup>*i*</sup>, *v*<sub>1</sub><sup>*i*</sup>, *v*<sub>2</sub><sup>*i*</sup>, *v*<sub>3</sub><sup>*i*</sup>…
*h*<sub>*i*</sub> = *s*<sub>0</sub><sup>*i*</sup>, *s*<sub>1</sub><sup>*i*</sup>, *s*<sub>2</sub><sup>*i*</sup>, *s*<sub>3</sub><sup>*i*</sup>, …

In Lachesis, a local history is equivalently expressed as:
*h*<sub>*i*</sub> = *g*<sub>0</sub><sup>*i*</sup>, *g*<sub>1</sub><sup>*i*</sup>, *g*<sub>2</sub><sup>*i*</sup>, *g*<sub>3</sub><sup>*i*</sup>, …
 where *g*<sub>*j*</sub><sup>*i*</sup> is the *j*-th local OPERA chain (local state) of the process *i*.

Each asynchronous run is a vector of local histories. Denoted by *σ* = ⟨*h*<sub>1</sub>, *h*<sub>2</sub>, *h*<sub>3</sub>, ...*h*<sub>*N*</sub>⟩.

Let *Σ* denote the set of asynchronous runs.

We can now use Lamport’s theory to talk about global states of an asynchronous system. A global state of run *σ* is an *n*-vector of prefixes of local histories of *σ*, one prefix per process. The happens-before relation can be used to define a consistent global state, often termed a consistent cut, as follows.

A consistent cut of a run *σ* is any global state such that if *v*<sub>*x*</sub><sup>*i*</sup> → *v*<sub>*y*</sub><sup>*j*</sup> and *v*<sub>*y*</sub><sup>*j*</sup> is in the global state, then *v*<sub>*x*</sub><sup>*i*</sup> is also in the global state. Denoted by **c**(*σ*).

By Theorem \[thm:conchains\], all nodes have consistent local OPERA chains. The concept of consistent cut formalizes such a global state of a run. A consistent cut consists of all consistent OPERA chains. A received event block exists in the global state implies the existence of the original event block. Note that a consistent cut is simply a vector of local states; we will use the notation **c**(*σ*)\[*i*\] to indicate the local state of *i* in cut **c** of run *σ*.

The formal semantics of an asynchronous system is given via the satisfaction relation ⊢. Intuitively **c**(*σ*)⊢*ϕ*, “**c**(*σ*) satisfies *ϕ*,” if fact *ϕ* is true in cut **c** of run *σ*. We assume that we are given a function *π* that assigns a truth value to each primitive proposition *p*. The truth of a primitive proposition *p* in **c**(*σ*) is determined by *π* and **c**. This defines **c**(*σ*)⊢*p*.

Two cuts **c**(*σ*) and **c****′**(*σ*′) are equivalent with respect to *i* if:
**c**(*σ*)∼<sub>*i*</sub>**c****′**(*σ*′) ⇔ **c**(*σ*)\[*i*\]=**c****′**(*σ*′)\[*i*\]

We introduce two families of modal operators, denoted by *K*<sub>*i*</sub> and *P*<sub>*i*</sub>, respectively. Each family indexed by process identifiers. Given a fact *ϕ*, the modal operators are defined as follows:

*K*<sub>*i*</sub>(*ϕ*) represents the statement “*ϕ* is true in all possible consistent global states that include *i*’s local state”.
**c**(*σ*)⊢*K*<sub>*i*</sub>(*ϕ*)⇔∀**c****′**(*σ*′)(**c****′**(*σ*′)∼<sub>*i*</sub>**c**(*σ*)  ⇒  **c****′**(*σ*′) ⊢ *ϕ*)

*P*<sub>*i*</sub>(*ϕ*) represents the statement “there is some consistent global state in this run that includes *i*’s local state, in which *ϕ* is true.”
**c**(*σ*)⊢*P*<sub>*i*</sub>(*ϕ*)⇔∃**c****′**(*σ*)(**c****′**(*σ*)∼<sub>*i*</sub>**c**(*σ*)  ∧  **c****′**(*σ*)⊢*ϕ*)

The next modal operator is written *M*<sup>*C*</sup> and stands for “majority concurrently knows.” This is adapted from the “everyone concurrently knows” in CCK paper . The definition of *M*<sup>*C*</sup>(*ϕ*) is as follows.

*M*<sup>*C*</sup>(*ϕ*)=<sub>*d**e**f*</sub>⋀<sub>*i* ∈ *S*</sub>*K*<sub>*i*</sub>*P*<sub>*i*</sub>(*ϕ*),
 where *S* ⊆ *P* and |*S*|&gt;2*n*/3.

In the presence of one-third of faulty nodes, the original operator “everyone concurrently knows” is sometimes not feasible. Our modal operator *M*<sup>*C*</sup>(*ϕ*) fits precisely the semantics for BFT systems, in which unreliable processes may exist.

The last modal operator is concurrent common knowledge (CCK), denoted by *C*<sup>*C*</sup>.

*C*<sup>*C*</sup>(*ϕ*) is defined as a fixed point of *M*<sup>*C*</sup>(*ϕ* ∧ *X*)

CCK defines a state of process knowledge that implies that all processes are in that same state of knowledge, with respect to *ϕ*, along some cut of the run. In other words, we want a state of knowledge *X* satisfying: *X* = *M*<sup>*C*</sup>(*ϕ* ∧ *X*). *C*<sup>*C*</sup> will be defined semantically as the weakest such fixed point, namely as the greatest fixed-point of *M*<sup>*C*</sup>(*ϕ* ∧ *X*). It therefore satisfies:
*C*<sup>*C*</sup>(*ϕ*)⇔*M*<sup>*C*</sup>(*ϕ* ∧ *C*<sup>*C*</sup>(*ϕ*))

Thus, *P*<sub>*i*</sub>(*ϕ*) states that there is some cut in the same asynchronous run *σ* including *i*’s local state, such that *ϕ* is true in that cut.

Note that *ϕ* implies *P*<sub>*i*</sub>(*ϕ*). But it is not the case, in general, that *P*<sub>*i*</sub>(*ϕ*) implies *ϕ* or even that *M*<sup>*C*</sup>(*ϕ*) implies *ϕ*. The truth of *M*<sup>*C*</sup>(*ϕ*) is determined with respect to some cut **c**(*σ*). A process cannot distinguish which cut, of the perhaps many cuts that are in the run and consistent with its local state, satisfies *ϕ*; it can only know the existence of such a cut.

Fact *ϕ* is valid in system *Σ*, denoted by *Σ* ⊢ *ϕ*, if *ϕ* is true in all cuts of all runs of *Σ*.
*Σ* ⊢ *ϕ* ⇔ (∀*σ* ∈ *Σ*)(∀**c**)(**c**(*a*)⊢*ϕ*)

Fact *ϕ* is valid, denoted ⊢*ϕ*, if *ϕ* is valid in all systems, i.e. (∀*Σ*)(*Σ* ⊢ *ϕ*).

A fact *ϕ* is local to process *i* in system *Σ* if *Σ* ⊢ (*ϕ* ⇒ *K*<sub>*i*</sub>*ϕ*)

If *ϕ* is local to process *i* in system *Σ*, then *Σ* ⊢ (*P*<sub>*i*</sub>(*ϕ*)⇒*ϕ*).

If fact *ϕ* is local to 2/3 of the processes in a system *Σ*, then *Σ* ⊢ (*M*<sup>*C*</sup>(*ϕ*)⇒*ϕ*) and furthermore *Σ* ⊢ (*C*<sup>*C*</sup>(*ϕ*)⇒*ϕ*).

A fact *ϕ* is attained in run *σ* if ∃**c**(*σ*)(**c**(*σ*)⊢*ϕ*).

Often, we refer to “knowing” a fact *ϕ* in a state rather than in a consistent cut, since knowledge is dependent only on the local state of a process. Formally, *i* knows *ϕ* in state *s* is shorthand for
∀**c**(*σ*)(**c**(*σ*)\[*i*\]=*s* ⇒ **c**(*σ*)⊢*ϕ*)

For example, if a process in Lachesis protocol knows a fork exists (i.e., *ϕ* is the exsistenc of fork) in its local state *s* (i.e., *g*<sub>*j*</sub><sup>*i*</sup>), then a consistent cut contains the state *s* will know the existence of that fork.

Process *i* learns *ϕ* in state *s*<sub>*j*</sub><sup>*i*</sup> of run *σ* if *i* knows *ϕ* in *s*<sub>*j*</sub><sup>*i*</sup> and, for all previous states *s*<sub>*k*</sub><sup>*i*</sup> in run *σ*, *k* &lt; *j*, *i* does not know *ϕ*.

The following theorem says that if *C*<sub>*C*</sub>(*ϕ* is attained in a run then all processes *i* learn *P*<sub>*i*</sub>*C*<sup>*C*</sup>(*ϕ*) along a single consistent cut.

If *C*<sup>*C*</sup>(*ϕ*) is attained in a run *σ*, then the set of states in which all processes learn *P*<sub>*i*</sub>*C*<sup>*C*</sup>(*ϕ*) forms a consistent cut in *σ*.

We have presented a formal semantics of Lachesis protocol based on the concepts and notations of concurrent common knowledge . For a proof of the above theorems and lemmas in this Section, we can use similar proofs as described in the original CCK paper.

With the formal semantics of Lachesis, the theorems and lemmas described in Section \[se:proof\] can be expressed in term of CCK. For example, one can study a fact *ϕ* (or some primitive proposition *p*) in the following forms: ‘is there any existence of fork?’. One can make Lachesis-specific questions like ’is event block *v* a root?’, ’is *v* a clotho?’, or ’is *v* a atropos?’. This is a remarkable result, since we are the first that define such a formal semantics for DAG-based protocol.

Reference
=========
