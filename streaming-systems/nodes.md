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

## Chapter 3 - Watermarks

* Definition
    * With processing unbounded data, we want to solve problem of knowing when
    it is safe to call an event-time window closed, meaning that we don't expect
    to receive more data once the window closes.
    * A naive approach is to base windows on processing time, which fails when
    any hiccups or spikes in the pipeline cause us to incorrectly put messages
    into windows.
    * We can make one fundamental assumption about streaming data: each message
    has an associated logical event timestamp.
    * Def: The *watermark* is a monotonically increasing timestamp of the
        oldest work not yet completed. There are two fundamental properties of
        watermarks that make them useful:
        * *Completeness* - If the watermark has advanced past a timestamp `T`,
        no more processing will happen for on-time events at or before `T`. We
        can therefore materialize any messages for messages at or before `T`.
        * *Visibility* - The watermark stalls if a message gets stuck in the
        pipeline. We can debug the source of the problem by debugging the
        message that is preventing the watermark from advancing.
* Source Watermark Creation
    * In order to establish a watermark, we must assign a timestamp to every
    message entering the pipeline.
    * Once a watermark is created as perfect or heuristic, it remains so for
    the remainder of the pipeline.
    * Perfect Watermark Creation
        * Perfect watermarks are a strict guarantee that no late messages will
        occur.
        * Can be created when the source assigns ingress times as event times
        for data entering the system. Timestamps must come from a single,
        monotonically-increasing source. A downside to this approach is that it
        does not capture useful event processing times.
        * Can be created from static sets of time-ordered logs, like Kafka.
        This is more useful than ingress timestamping because the watermark
        track meaningful event times.
    * Heuristic Watermark Creation
        * An *estimate* that no data with event times less than the watermark
        will be seen again.
        * The better the heuristic watermark, the less late data.
        * This approach scales better.
        * Can be achieved by tracking minimum event times of unprocessed data,
        monitoring growth rates, and utilizing external information like
        network topology.
* Watermark Propagation
    * Most real-work pipelines consist of multiple stages. Each stage tracks
    its own watermark.
    * *Input watermarks* captures the progress for everything upstream of a
    stage. For sources, the input watermark is for source data. For non-source
    stages, the input watermark is defined as the minimm of the output
    watermarks of all upstream sources and stages.
    * *Output watermarks* capture the progress of a stage, and represents the
    minimum of the stage's input watermark and the event times of all nonlate
    active messages within the stage.
    * Subtracting input from output watermarks givesn the amount of event-time
    legacy (lag) introduced by the stage.
* *Percentile watermarks* are guarantees to have processed `X%` of all events
    with earlier timestamps.
    * This works in cases where "mostly correct" is sufficient.
    * The watermark advances more quickly and is not delayed by outliers, which
    lowers processing time.
* Processing-Time Watermarks
    * By only examining event-time watermark, you cannot distinguish between
    old data and a delayed system.
    * Processing-time watermarks provide a notion of processing delay separate
    from data delay.
    * An increase in processing time delay can be caused by networking issues,
    etc.

## Chapter 4 - Advanced Windowing

* When/Where: Processing-Time Windows
    * Use processing-time monitoring is appropriate for monitoring data as it's
        observed, such as monitoring web traffic. It's not appropriate for use
        cases for which the time events happened is important.
    * When working with a model where windowing is strictly event-time based,
        there are two ways to achieve processing-time windowing:
        * Triggers - Use a global event-time window.
        * Ingress Time - As data arrive, their event times are updated to match
            their ingress time.
    * The downside of processing-time windows is that the contents of the
        windows change when the observation order of the inputs changes.
        * Each section below illustrates two graphs with the same events. Both
            graphs process the same event at different times.
        * Event-Time Windowing
            * The results are the same even when observation order differs.
            * See [http://streamingsystems.net/fig/4-2](http://streamingsystems.net/fig/4-2).
        * Process-Time Windowing via Triggers
            * Observation time makes results different.
            * See [http://streamingsystems.net/fig/4-3](http://streamingsystems.net/fig/4-3).
        * Process-Time Windowing via Ingress Time
            * Observation time makes results different.
            * See [http://streamingsystems.net/fig/4-4](http://streamingsystems.net/fig/4-4).
    * tl;dr: Event-time windowing is order-agnostic, processing-time windowing
        is not.
* Where: Session Windows
    * Sessions are a type of window that capture a period of activity in data
        followed by a period of inactivity.
    * Useful for measuring user activitiy.
    * Session windows are an example of data-driven window: the location and
        window sizes are a consequence of the input data themselves.
    * They are also an example of an unaligned window: the window does not
        apply uniformly across data (contrasts with fixed and sliding windows
        which apply uniformly across data).
    * Sometimes the windowing key is known in advance (i.e. a session ID).
        Other times, the session window is created created out of set of
        smaller, overlapping windows containing a single record.
* Where: Custom Windowing
    * Most systems don't support custom windowing, except Beam. This section
        just focuses on Beam.
    * A custom windowing strategy consists of two things: window assignment and
        window merging.
    * Variations on Fixed Windows
        * Fixed Windows
            * Elements are placed into the appropriate fixed-window based on
                its timestamp and the window size and offset parameters.
        * Unaligned fixed windows
            * Aligned windows all close at the same time, which causes CPU
                usage bursts.
            * Unaligned windows spread window completion out over time at the
                expense of not being able to compare data across windows.
        * Per-element/key fixed windows
            * Element data determines window size.
            * Useful when you want to support arbitrary window sizes.
            * Example given is a Cloud Dataflow customer that generates
                analytics data for its customers. Each customer is allowed to
                configure the window size over which it aggregates metrics.
    * Variations on Session Windows
        * Implementation
            * Assignment: Each element is placed into a proto-session window
                that starts at the element's timestamp and extends for the gap
                duration.
            * Merging: At grouping time, all eligible windows are sorted.
                Overlapping windows are merged together.
        * Bounded Sessions
            * Sessions that are not allowed to grow beyond a certain size.

## Chapter 5 - Exactly-Once and Side Effects

* *Exactly-once processing* means ensuring that every record is processed
    exactly once.
* This chapter uses Cloud Datastore as an example.
* Accuracy Versus Completeness
    * Dataflow does not guarantee that custom code is run only once per record.
        Many invocations can happen, and only one will "win."
    * Nonidempotent side effects are not guaranteed to execute exactly once.
        Strategies for dealing with this issue is discussed later on.
* Problem Definition
    * Dataflow workers shuffle data between themselves via RPC.
    * This chapter focuses on three things:
        * *Shuffle*: How Dataflow guarantees that every record is shuffled
            exactly once.
        * *Sources*: How Dataflow guarnatees that every source record is
            processed exactly once.
        * *Sinks*: How Datflow guarantees that every sink produces accurate
            output.
* Ensuring Exactly Once in Shuffle
    * RPCs can fail for many reasons: network interrupts, timeouts, etc.
    * Dataflow guarantees that records are not lost by employing *upstream
        backup*, which means that the sender retries RPCs until it receives
        positive acknowledgement of receipt, even if the sender crashes. This
        guarantees *exactly once* delivery.
    * RPC calls may succeed even if they report failures. Therefore we need to
        contend with duplicates.
    * To avoid duplicates, Dataflow tags every message sent with a unique ID.
        Each receiver keeps a list of all unique IDs it has seen, discarding
        incoming messages with unique IDs that are in the list.
* Addressing Determinism
    * User-supplied processing functions may nondeterministic. A function may
        execute twice on the same input record (due to retry), yet produce
        different output on retry.
    * Dataflow solves this problem by using checkpointing to make
        nondeterministic processing effectively deterministic. Output from a
        transform is saved to stable storage before being delivered to the next
        stage. Upon retry, the same output stored at the check point is resent
        without reinvoking the callback.
* Performance
    * Each receiver maintains a catalog of record IDs. For every record,
        Dataflow checks to see if the ID is in the catalog to determine if the
        record is a duplicate. Dataflow implements two techniques to reduce the
        I/O overhead of duplicate processing: *graph optimization* and *Bloom
        filters*.
* Graph Optimization
    * This is an optimization run on a pipeline before execution.
    * *Fusion* is a optimization that fuses many logical steps into a single
        execution stage. All steps are run in the same process-unit, which
        reduces the amount of data transfer needed to store exactly-once data
        for each step.
    * Refresher: 
        * *Commutative property* states that you can process numbers in any
            order and arrive at the same answer.
        * *Associative propertY* states you can group numbers in any
            combination and arrive at the same answer.
    * Dataflow optimizes associative and commutative `Combine` operations (such
        as `count` and `sum`) by performing partial combining locally before
        sending the data to the main grouping operation. This can reduce the
        number of messages for delivery.
* Bloom Filters
    * Bloom filters are compact data structures that allow for quick membership
        checks. They can return false positives but never false negatives.
    * Dataflow uses Bloom Filters to optimize the duplicate record check. When
        a new message arrives, Dataflow looks up its record ID in the bloom
        filter. If the bloom filter returns false, the record is not a
        duplicate, and the expensive membership check can be skipped, saving
        I/O. If it returns true, it must do the lookup, because the record may
        or may not be a duplicate.
    * This technique works best in a healthy pipeline, where most arriving
        records are not duplicates.
    * Dataflow limits bloom filter size by creating a seaprate one for every
        10-minute range.
* Garbage Collection
    * Every Dataflow worker persistently stores a catalog of unique record IDs
        it has seen. Grabage collection prevents old records from filling up
        storage.
    * Dataflow calculates a garbage collection watermark based upon the
        timestamp used for bucketing exactly-once bloom filters. This watermark
        is based on the amount of physical time spent waiting at a given stage,
        and provides insight into which parts of the pipeline are slow.
    * *Network remnants* are messages that get stuck for an indefinite period 
        of time in the network and suddenly appear. If a message appears that
        is older than the watermark, we know that the remnant message is a
        duplicate, because the garbage collection watermark only advances when
        all delivers have been acknowledged.
* Exactly Once in Sources
    * Some data sources are *deterministic*, meaning the input will be the same
        on each retry.
    * Some data sources are *nondeterministic*. These sources a required to
        provide a way to inform the system what the record IDs should be.

## Chapter 6 - Streams and Tables

* Streams-and-Table Basics Or: a Special Theory of Stream and Table Relativity
    * The data structure underlying most databases is an append-only log, which
        is effectively a stream.
    * To create a table from a stream, you serially apply updates to a table.
    * To create a stream from a table, you record a changelog of the table.
    * SQL materialized views are created from the changelog of an underlying
        source table.
    * Streams → tables: The aggregation of a stream of updates over time yield
         a table.
    * Tables → streams: The observation of changes to a table over time yields
        a stream.
* Toward a General Theory of Stream and Table Relativity
    * Tables are data at rest. They are essentially a snapshot of data in time.
    * Streams are data in motion. They capture of the evolution of data over
        time.
* **Batch Processing Versus Streams and Tables**
    * **A Streams and Tables Analysis of MapReduce**
        * MapReduce is broken down into these steps:
            * *MapRead*: Consumes the input data and preprocesses them into
                standard key/value mapping.
            * *Map*: Repeatedly consumes a single key/value pair from the
                prepocessed input and outputs more key/value pairs.
            * *MapWrite*: Writes key/value pairs to persistent storage.
            * *ReduceRead*: Consumes shuffled data from previous step and
                converts them into a standard key/value-list form for
                reduction.
            * *Reduce*: Repeatedly consumes a single key and its value-list of
                records and outputs more records.
            * *ReduceWrite*: Writes output from previous step to output
                datastore.
        * The *MapRead* and *ReduceWrite* are commonly referred to as sources
            and sinks.
        * How does batch processing fit into stream/table theory? Map/Reduce is
            a series of stream and table creation.
            * [TABLE] → *MapRead* → [STREAM] → *Map* → [STREAM] → *MapWrite* →
                [TABLE] → *ReduceRead* → [STREAM] → *Reduce* → [STREAM] →
                *ReduceWrite* → [TABLE]
        * Streams work with bounded data (typically considered batch data) just
            as they do with unbounded data.

* *What, Where, When, and How* in a Streams and Tables World
    * *What*: Transformations
        * There are two types of *what* stream/table theory transformations:
            * *Nongrouping*: Transforming a stream of records and produce a
                stream of transformed records.
            * *Grouping*: Transforming a stream of records and grouping them
                together in some way, thereby forming a table.
        * If the grouping operation is the final stage in a pipeline, it may
            make sense to export the table directly. Kafka and Flink provide a
            way to read the table data directly instead of having to
            rematerialize them somewhere else.
    * *Where*: Windowing
        * There are two aspects of windowing that interact with stream/table
            theory:
            * *Window assignment*: Placing a record into one or more windows.
                When a record is placed into a window, the definition of the
                window is combined with the record's user-assigned key, which
                creates an implicit composite key used at groping time.
            * *Window Merging*: Logic that makes dynamic, data-driven types of
                windows, such as sessions, possible.
    * *When*: Triggers
        * Grouping means stream-to-table conversion.
        * Triggers are a complement to grouping. They are an "ungrouping"
            operation that drive table-to-stream conversion.
        * Triggers are special procedures applied to a table that allow for
            data within that table to be materialized in response to relevant
            events.
        * The trigger operation in streams/table theory is similar to classic
            database triggers. A trigger is in effect code that is evaluated
            for every row in the state table as time progresses. When the
            trigger fires, it takes the corresponding data that are currently
            at rest in the talble and puts them into motion, yielding a new
            stream.
        * The main semantic difference between batch and streaming systems is
            the ability to trigger tables incrementally. It's not really a
            semantic difference, but more of a latency/throughput tradeoff.
        * There's not much difference between batch and streaming systems
            except for an efficiency delta (in favor of batch) and a natural
            ability to deal with unbounded data (in favor of streaming).
        * Apache Beam is a system that provides the best of both worlds: one
            which provides the ability to handle unbounded data naturally but
            can also balance the tensions between latency, throughput, and cost.
    * *How*: Accumulation
        * Accumulation modes in the streams/tables model:
            * *Discarding mode*: Requires the system to throw away the previous
                value for the window when triggering or keep around a copy of
                the previous value and compute the delta the next time the
                window triggers.
            * *Accumulating mode*: Requires no additional work, the current
                value for the window in the table at triggering time is what is
                emitted.
            * *Accumulating and retracting mode*: Requires keeping around
                copies of all previously triggered (but not yet retracted)
                values for the window.

## Chapter 7 - The Practicalities of Persistent State

* **The Inevitability of Failure**
    * Pipelines running on unbounded data are meant to run forever, which is a
        difficult SLO to achieve. Service interruptions are inevitabile.
    * Persistent state helps ensure that long-running pipelines can resume from
        where they left off after a failure.
* **Correctness and Efficiency**
    * Persistent state provides a *basis for correctness* in light of
        epehemeral inputs. Processing intermediate data can continue to happen
        even after the input source is gone.
    * Persistent state provides a way to *minimize work duplicated and data
        persisted*. Any work that wasn't checkpointed (saved) in a pipeline
        must be re-done after a failure. Checkpointing progress in persistent
        state can help save on time and cost of reprocessing after failure. The
        amount of data persisted can be lowered by calcualting some smaller,
        intermediate form of the input data and persisting it (such as total
        sum and count).
* **Implicit State**
    * We must make tradeoffs between persisting everything (good for
        consistency, bad for efficiency) and never pesisting anything (bad for
        consistency, good for efficiency).
    * **Raw Grouping**
        * Any time a new element arrives in the group, append it to the list of
            elements seen in the group. (This is a always-persist-everything
            end of the spectrum.)
        * Disadvantages
            * We're storing a lot of data.
            * Duplicate effort in the case of multiple trigger firings.
            * We must buffer up raw inputs to grouping operation before
                processing the group as a whole, so there's no way to partially
                process the data.
    * **Incremental Combining**
        * A form of automatic state built upon a user-defined associative and
            commutative combining operator.
        * Advtangages of aggregatations:
            * Captures partial progress, which is smaller than the input size.
            * Incremenatal aggregations are indifferent to ordering across two
                dimensions:
                * *Individual elements*: (commutativity) `COMBINE(a, b) == COMBINE(b, a)`
                * *Groupings of elements*: (associativity) `COMBINE(COMBINE(a, b), c) == COMBINE(a, COMBINE(b, c))`
            * This leads to less buffering (we can compute the results as they
                arrive) and increased parallelization (we're free to
                arbitrarily distibute computation across a multiple machines.).
        * Disadvantage: the grouping operation must fit within a restricted
            structure, which is fine for sums, means, etc. Many real-world
            cases require a more general approach that allows precise control
            over trade-offs of complexity and efficiency.
* **Generalized State**
    * The implicit state aproaches are inflexible. To support a more
        generalized approach to streaming persistent state, we need flexibility
        in three dimensions:
        * **Data Structures**: The ability to structure data we read and write
            in ways that are most appropriate to the task at hand. Raw grouping
            only provides an appendable list, incremental aggregation only a
            single value.
        * **Write and Read Granularity**: The ability to tailor the amount and
            type of data written or read at any given time for optimal
            efficiency.
        * **Scheduling of Processing**: The ability to bind the time at which
            specific types of processing occur to the progress of time in
            either event-time or processing time.
    * The chapter continues by explaining how Apache Beam handles generalized
        state and then details a case study about advertising conversation
        attribution.

## Chapter 8 - Streaming SQL

> This chapter discusses a hypothetical Streaming SQL implementation. There's
not a lot of practical knowledge to be gained from this chapter. This section
will only contain notes on some high-level concepts.

* **Relational Algebra**
    * Relational algebra is a mathematical way of describing relationships
        between data that consist of named, typed tuples.
    * The *closure property* states that applying any operator from the
        relational algebra to any valid relation always yields another
        relation. Basically, applying any operator on a table creates another
        table.
    * Existing streaming SQL implementations fail to support the closure
        property. The author argues that, in order for streaming SQL to gain
        mainstream adoption, an implementation must cleanly support the closure
        property.
* **Time-Varying Relations**
    * In order to integrate streaming into SQL, relations must be extended to
        represent data *over time* rather than a set of data at a *specific
        point* in time.
    * Traditionally, SQL relations are 2-dimensional, consisting of columns in
        the x-axis and rows of records in the y-axis. Streaming SQL would
        support a z-axis, a third dimension that tracks snapshots of data over
        time. Time-varying relations are a sequence of classic relations that
        each exist independently within their own disjointed (but adjacent)
        time ranges.
    * Time-varying relations have two important properties:
        * The full set of operators from classic relational algebra must remain
            valid when applied to time-varying relations.
        * The closure property of relational algebra remains intact when
            applied to time-varying relations.

## Chapter 9 - Streaming Joins

> This chapter discusses a hypothetical Streaming SQL implementation. There's
not a lot of practical knowledge to be gained from this chapter. This section
will only contain notes on some high-level concepts.

* **All Your Joins Are Belong to Streaming**
    * Joins group data, grouping operations always consume a stream to yield a
        table, therefore all joins are streaming joins.
* **Unwindowed Joins**
    * Because joins are simply another type of grouping operation, joins do not
        require windows. In order to consue a joined table as a stream, we only
        need to apply an ungrouping (or *trigger*) operation that doesn't block
        until it's seen all the data.
        * You can window the join into a nonglobal window and use a watermark trigger.
        * ... or you could trigger on every record....
        * ... or you could periodically trigger as time advances.
    * There's only really one type of join at its core: the `FULL OUTER` join.
        All other types (`LEFT/RIGHT OUTER1`, `INNER`, etc.) are a subset of
        `FULL OUTER`.
    * `FULL OUTER`: The full list of rows in both datasets, with rows in the
        two datasets that share the same join key combined together as well as
        unmatched rows for either side.
    * `LEFT OUTER`: The same as `FULL OUTER` with any unjoined rows from the
        right dataset removed.
    * `RIGHT OTUER`: The same as `FULL OUTER` with any unjoined rows from the
        left dataset removed.
    * `INNER`: The intersection of `LEFT OUTER` and `RIGHT OUTER`.
    * `ANTI`: The obverse of `INNER`. They contain all of the unjoined rows.
    * `SEMI`: Returns a row from the left table if there is at least one
        matching row in the other table. Rows in the left table will be
        returned at most once.
        * In SQL, this is usually achieved with a `WHERE... IN...` construct.
* **Windowed Joins**
    * Two reasons for windowing joins:
        1. To partition time in some meaningful way (such as currency
            conversion).
        2. To provide a meaningful reference point for timing out a join (in
            unbounded data situations).