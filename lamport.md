## Lamports Algorithm
Lamport's Logical Clocks
People use physical time to order events. For example, we say that an event at 8:15 AM occurs before an event at 8:16 AM. In distributed systems, physical clocks are not always precise, so we can't rely on physical time to order events. Instead, we can use logical clocks to create a partial or total ordering of events. This article explores the concept of and an implementation of the logical clocks invented by Leslie Lamport in his seminal paper Time, Clocks, and the Ordering of Events in a Distributed System.
Math Refresher

Before we dive into time, clocks, or ordering, let's quickly review a bit of the underlying math.
Sets

A set is an unordered collection of distinct things. For example, we can bundle the integers 3, 11, and 0 into a set denoted {3,11,0}
. Or, we could throw the integer 42 into a set with only one element: {42}. Or, we could have the empty set {}. We say an element a is a member of A if A contains a, and we denote this a∈A

.

A subset A
of a set B is a set whose elements are all members of B. We denote that A is a subset of B as A⊆B. For example: {1,2}⊆{1,2,3}, {}⊆{1,2,3}, and {1,2,3}⊆{1,2,3}. A set A is a strict subset of B, denoted A⊂B, if A⊆B and A≠B

.

The Cartesian product of two sets A
and B, denoted A×B, is the set of ordered pairs (a,b) for every a∈A and b∈B. For example, {1,2}×{a,b,c}={(1,a),(1,b),(1,c),(2,a),(2,b),(2,c)}

.
Relations

A binary relation R
on a set A is a subset of A×A. Similarly, a binary relation on a set A and B is a subset of A×B. For example, here's a couple relations on {1,2} and {a,b,c}: {(1,a),(2,c)set}, {(1,a),(2,b),(3,c)}, {}, and {(1,a),(1,b),(1,c)}. We can denote (a,b)∈R as aRb. Consider for example the familiar less-than relation, <, on the set of natural numbers. Two naturals i, and j are in the relation < if i is less than j. We denote this i<j. Concretely, < is the infinite set {(0,1),(0,2),(0,3),…,(1,2),(1,3),…}

.
Partial and Total Orderings

An irreflexive partial ordering on a set A
is a relation on A

that satisfies three properties.

    irreflexivity: a≮a

antisymmetry: if a<b
then b≮a
transitivity: if a<b
and b<c then a<c

For example, the strict subset relation we saw earlier is an example of an irreflexive partial ordering. Here are some examples of the previous three properties being satisfied.

    irreflexivity: {1,2,3}⊄{1,2,3}

antisymmetry: {1,2}⊂{1,2,3}
, so {1,2,3}⊄{1,2}
transitivity: {1}⊂{1,2}
and {1,2}⊂{1,2,3}, so {1}⊂{1,2,3}

It's important to note that for some sets a
and b, a⊄b and b⊄a. For example, {1,2}⊄{2,3} and {2,3}⊄{1,2}

. This is quite different than total orderings.

An irreflexive total ordering is a irreflexive partial ordering that satisifes another condition.

    totality: if a≠b

then a<b or b<a

    .

For example, the "less-than" relation on natural numbers is an irreflexive total ordering. For all naturals i
and j where i≠j, i<j or j<i

.
A Partial Ordering

Physical time forms a natural "happened-before" irreflexive partial ordering of events. If we consider two events a
and b, we can use physical time to know whether a happened before b, b happened before a

, or the two are unrelated (i.e. they happened at the same time). Since physical clocks are imprecise, we can't use physical time in a distributed system, but we'd still like an irreflexive partial ordering of events.

We'll define such an ordering, but first, let's formalize our system a bit. Our distributed system consists of a set of processes that each execute their own set of events. There are three events each process can execute:

    A local event
    Sending a message to another process
    Receiving a message from another process

The union of every process' events is the set of events we wish to order. We can visualize such a system using a space-time graph, such as the one in Figure 1 below. Each vertical line represents a process. Time flows forward as we traverse the graph upwards through the set of events represented as annotated points. If one process sends a message and another receives a message, the two are events are connected by a colored line.
Figure 1

The system in Figure 1 has three processes: A
, B, and C. Process A

has four events.

    A0

: Process A sends a message to process B
A1
: Process A receives a message from process B
A2
: Process A
executes a local event
A3
: Process A receives a message from process B

Now we can define our "happened-before" irreflexive partial ordering, denoted →

, as the smallest relation on events that satisfies the following three properties.

    If a

and b are events in the same process and a happens before b then a→b. For example, A0→A1
.
If a process sends a message as event a
and another process receives the message as event b, then a→b. For example, A0→B2
.
If a→b
and b→c, then a→c For example, A0→B2 and B2→B4 and B4→C3, so A0→C3

    .

Graphically, a→b
if we can trace a path forward through time from a to b. For example, we can show that A0→C3 by tracing from A0 across the blue line to B2 up to B4 and across the purple line to C3. This interpretation also makes it clear that some events such as A2 and B3 are not related because we can't trace a path forward through time from A2 to B3 or from B3 to A2. In terms of physical time, A2 might happen before B3, but our →

relation is defined independent of physical time.
Logical Clocks

Now let's introduce clocks to help us order events according to our →

ordering. Unlike physical clocks which are physical entities that assign physical times to events, our clocks are simply a conceptualization of a mathematical function that assigns numbers to events. These numbers act as timestamps that help us order events. Since our clocks logically order events, rather than physically order them, we call them logical clocks.

More formally, each process Pi
has a clock Ci which is a function from events to the integers. The timestamp of an event e in Pi is Ci(e). The system clock, C, is also a function from events to the integers where C(e)=Ci(e) when e is an event in Pi. In Figure 1, timestamps are associated with horizontal dashed lines beginning at 0 and increasing by 1 forwards through time. For example, events B0 through B7 have timestamps 0 through 7

.

We evaluate our logical clocks with the following correctness criterion known as the Clock Condition.
∀a,b.a→b⟹C(a)<C(b)

For example, A0→B2
, so in order for a clock C to satisfy the Clock Condition, it must be that C(A0)<C(B2). Also note that it is not the case that ∀a,b.C(a)<C(b)⟹a→b. Consider A2 and B3. C(A2)<C(B3), yet A2 and B3 are not related according to our →

ordering.

Given a system, we can construct a logical clock using a simple algorithm. Each process Pi
maintains a mutable timestamp Ci. When an event e occurs, we let Ci(e) be Ci when e occurs. Each process Pi increments Ci after every event in Pi. Also, when a process Pi sends a message to Pj, it includes its timestamp Ci with the message. When Pj receives the message, it sets its timestamp Cj to be greater than the received Ci

. A concrete implementation of this algorithm is discussed below.
A Total Ordering

Our algorithm generates a clock that is consistent with our →
ordering according to the Clock Condition, but what if we want to totally order the events of a system? It's surprisingly simple! First, choose an arbitrary irreflexive total ordering of the processes, denoted ⇒. For example, ≺ may be {(A,B),(A,C),(B,C)}. Now, we can define a total ordering of events using C and our ≺

ordering as the smallest relation satisfying the following properties.

    If C(a)<C(b)

, then a⇒b
If C(a)=C(b)
and a≺b, then a⇒b

Now, we can order previously unrelated events. For example, A1⇒B2
and A2⇒B3

.
Implementation

I implemented Lamport's logical clocks in Python! The implementation provides a function lamport.wind which takes in a list of functions each of which represents a process. The functions take in a Clock_i instance that supports three methods: send(n), recv(n), and local(). wind returns a function which you can invoke to plot the space-time graphs we've been using to visualize the logical clocks. Here's a couple of examples!

import lamport

def f(clock):
    clock.send(1)
    clock.recv(1)

def g(clock):
    clock.recv(0)
    clock.send(0)

lamport.wind([f, g])()

import lamport

def f(clock):
    clock.send(1)
    clock.send(2)
    clock.recv(1)
    clock.recv(2)

def g(clock):
    clock.send(0)
    clock.send(2)
    clock.recv(0)
    clock.recv(2)

def h(clock):
    clock.send(0)
    clock.send(1)
    clock.recv(0)
    clock.recv(1)

lamport.wind([f, g, h])()

import lamport

def f(clock):
    clock.send(1)
    clock.recv(1)
    clock.local()
    clock.recv(1)

def g(clock):
    clock.send(0)
    clock.send(2)
    clock.recv(0)
    clock.local()
    clock.send(2)
    clock.send(0)
    clock.local()
    clock.recv(2)

def h(clock):
    clock.local()
    clock.send(1)
    clock.recv(1)
    clock.recv(1)

lamport.wind([f, g, h], "clock.png")()

