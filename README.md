OCaml bindings for Kafka
====================================

Pre-requisites
--------------
* [OCaml](http://caml.inria.fr/)
* [Apache Kafka](http://kafka.apache.org/)
* [librdkafka](https://github.com/edenhill/librdkafka)

License
-------
MIT License

Install
-------
    $ make
    $ make test       # assuming kakfa is running at localhost:9092 with a 'test' topic.
    $ make install

Usage
-----

    #load "okafka.cma";;

    (* Prepare a producer handler. *)
    let producer = Kafka.new_producer ["metadata.broker.list","localhost:9092"];;
    let producer_topic = Kafka.new_topic producer "test" ["message.timeout.ms","10000"];;

    (* Prepare a consumer handler *)
    let consumer = Kafka.new_consumer ["metadata.broker.list","localhost:9092"];;
    let consumer_topic = Kafka.new_topic consumer "test" ["auto.commit.enable","false"];;
    let partition = 1;;
    let timeout_ms = 1000;;

    (* Start collecting messages *)
    (* Here we start from offset_end, i.e. we will consume only messages produced from now. *)
    Kafka.consume_start consumer_topic partition Kafka.offset_end;;

    (* Produce some messages *)
    Kafka.produce producer_topic partition "message 0";;
    Kafka.produce producer_topic partition "message 1";;
    Kafka.produce producer_topic partition "message 2";;
   
    (* Consume messages *)
    let consume t p = match Kafka.consume t p timeout_ms with
      | Kafka.Message(_,_,_,msg) -> msg
      | Kafka.PartitionEnd(_,_,_) -> assert false
    in
    let msg = consume consumer_topic partition in assert (msg = "message 0");
    let msg = consume consumer_topic partition in assert (msg = "message 1");
    let msg = consume consumer_topic partition in assert (msg = "message 2");

    (* Stop collecting messages. *)
    Kafka.consume_stop consumer_topic partition;;

    (* Topics and producers must be released. *)
    Kafka.destroy_topic producer_topic;;
    Kafka.destroy_handler producer;;
    Kafka.destroy_topic consumer_topic;;
    Kafka.destroy_handler consumer;;
