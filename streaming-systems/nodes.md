# Streaming Systems

## Chapter 1 - Streaming 101

* Terminology: What is Streaming?
    * A *Streaming System* is a processing engine designed with infinite
    datasets in mind.
    * Data Cardinality
        * *Bounded Data* - Dataset that is finite in size.
        * *Unbounded Data* - Data that can be infinite in size.
    * Data Constitution
        * *Table* - A holistic view of a dataset at a specific point in time.
        * *Stream* - An element-by-element view of the evolution of a dataset over
        time.
    * Streaming systems have historically been considered limited to providing 
        low-latency, inaccurate, or speculative results, typically coupled with a
        batch system for eventually correct results.
        * This is the foundation of *Lambda Architectures*.
        * Lambda Architectures require maintaing two parallel systems: one
        that delivers quick results and one that delivers correct results.
        * [Questioning the Lambda Architecture](https://www.oreilly.com/radar/questioning-the-lambda-architecture/)
        was a seminal blog post that detailed shortcomings of this architecture.
        * The idea is to use a replayable system (such as Kafka) as part of a
        single pipeline to achieve the task at hand.
    * To design a streaming system that can surpass Lamba Architecture, you
    need two things:
        * *Correctness*, which requires consistent storage.
        * *Tools for reasoning about time*, which will be discussed later in
        book.
* Event Time Versus Processing Time
    * *Event Time*
        * The time at which events actually occurred.
    * *Processing Time*
        * The time at which events are observed in a system.
    * In a perfect world, event time = processing time.
        * This does not happen in the real work.
        * The difference between Event Time and Processing Time is called
        *skew*.
* Data Processing Patterns
    * Bounded Data
    * Unbounded Data: Batch - Involves slicing up unbounded data into a
        collection of bounded datasets for batch processing.
        * *Fixed Windows* (aka *Tumbling Windows*)
            * Involves dividing up the data by fixed time range.
            * Presents some problems: What to do when events are received late?
            Some mitigation may be necessary, such as delyaing processing until
            all events are received.
        * *Sessions*
            * Sessions are periods of activity terminated by a gap of
            inactivity.
            * Increased complexity, because sessions may span multiple batches
            (when using a typical batching engine).
    * Unbounded Data: Streaming
        * *Time-agnostic* - Time doesn't matter, all logic is data driven.
        * *Filtering* - Remove elements from stream that don't meet certain
        criteria.
        * *Inner Joins*
            * When joining two unbounded sources, process when an element from
            both sources arrives.
            * When seeing a message from one state, buffer the message in a
            persistent state until the corresponding value arrives from the
            second source.
            * Requires some sort of garbage collection for messages that have
            no corresponding messages to join on.
        * *Approximation Algorithms*
            * Algorithms such as Top-N and k-means.
            * Relatively few exist, and are often complicated.
        * *Windowing* - Chopping data source along temporal boundaries into
            smaller processing chunks.
            * *Fixed Windows* (aka *Tumbling Windows*)
            * *Sliding Windows* (aka *Hopping Windows*)
                * A generalization of fixed windows. (If period is < length,
                windows overlap. If periods equal length, fixed windows.)
            * *Sessions*
                * Common for analyzing user behavior over time.
                * Lengths cannot be defined beforehand, they are dependent on
                the data involved.
        * *Windowing By Processing Time*
            * Create windows based upon when messages are processed.
            * Advantages: simple, "completeness" is straightforward, useful for
            inferring infomration about the data as it is observed (useful for
            application monitoring).
            * Downside: If the data has associated event times, the data must
            arrive in event-time order in order for the windows to be correct.
            This is problematic when events can't be received in a timely
            manner due to network faults. Messages received late have a high
            event-time skew. Windows may contain missing or out-of-date data.
        * *Windowing By Event Time*
            * Create windows based upon event. The "gold standard" of
            windowing.
            * Guarantees event-time correctness. Out of date messages get
            assigned to their respective event-time windows upon processing.
            * Drawbacks
                * Windows must exist for longer than the actual length
                of the window. This requires more storage for buffering.
                * It's impossible to gauge completeness, as we don't always
                know if we have all the data. Many systems provide heuristics
                to give a estimate of window compleition.
                * For situations where absolute correctness is required, custom
                means of closing windows must be implemented.

## Chapter 2 - The What, Where, When, and How of Data Processing
        
* A *trigger* is a mechanism for declaring when output for a window should
    be materialized relative to some signal. They make it possible to observe
    the output of a window multiple times as it evolves.
    * *Repeated update triggers* generate updated window panes as its
    contents evolve. These can occur on every new record at regular
    intervals. Usually used to balance latency and cost. These is the most
    common type encountered in streaming systems.
    * *Completeness triggers* materialize a window pane only after it's
    believed that all the input has been received. These tend to be more
    batch-oriented. Completeness triggers are driven by watermarks.
    * Per-record triggering is ideal for use cases where the output stream is
    directed at a table that can be polled for updates. These can be chatty. To
    mitigate chattiness, you can implement a processing-time delay.
    * The *aligned delay* approach is more predictable, but results in
    bursty workloads that require greater provisioning.
    * The *unaligned delay* approach spreads the load more evenly across time,
    but with less predictable latency.
    * Triggers are OK for eventual consistency. They are problematic with
    varying levels of skew. It makes it more difficult to determinate if
    output is accurate at any given point.
* A *watermark* is a notion of input completeness with respect to event
    times. A watermark with value `X` means that all input data with times less
    than `X` have been observed.They are the way the system measured progress and
    completeness relative to event time.
    * *Perfect watermarks* are based on perfect knowledge of input data. In
    this case, late data is impossible.
    * *Heuristic watermarks* use whatever information it get from the input
    data to provide an estimate of progress. These generally work well but can
    lead to ate data.
    * Shortcomings of watermarks:
        * Too slow: A watermark may be delayed due to known unprocessed data.
        Occurs when output depends on watermark avancement.
        * Too fast: When data with event times before the water mark arrive
        some time later.
    * The shortcomings of these individual types of watermarks can be overcome
        by using both types at the same time. Enter the *early/on-time/late trigger*,
        which partitions the panes into three categories:
        * > 0 *early panes*, which generate speculative results early on as the
        data arrive. This compensates for too slow watermarking.
        * At most one *on-time* pane, which results from watermark
        completions. This makes an assertion that the system believes it's
        received all input. Helps us reason about missing data.
        * > 0 *late panes*, which account for late data. This compensates for
        watermarks being too fast.
* *Garbage collection* is necessary for long-lived, out-of-order streamprocessing.
    * It's not practical to keep all persistent data from an unbounded data
    source.
    * It's necessary to place a *horizon* (or *bound*) on how late any given
    record may be relative to the watermark. Any date that exceeds the lateness
    horizon may be dropped without processing.
    * The horizon should be based on the event-time domain rather than
    processing-time domain. This ties garbage collection to progress on the
    pipeline. If tied to processing-time, data would be unnecessarily garbage
    collected if the system were to go down.
* An *accumulation mode* specifies the relationship between multiple results
    that are observed for the same window. What do we do with the existing data
    for each successive update of the window?
    * *Discarding* - Each time a pane is materialized, discard the stored
    state. Useful for replacing previous results.
    * *Accumulating* - Each time a pane is materialized, retain stored state and
    accumulate future inputs into existing state. Useful for overwriting
    previous results.
    * Accumulating and Retracting* - Same as accumulating mode, but with an
    independent retracting of the previous pane. "I previously told you the
    result was X. Get rid of X and replace it with Y". This is important for
    cases where downstream consumers are regrouping data by a different
    dimension. The retraction helps them correctly update their groupings.

