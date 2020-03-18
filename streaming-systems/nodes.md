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
                * Lengths cannot be defined beforehand, they are dependent on the data involved.
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
                

        
