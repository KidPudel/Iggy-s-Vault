Informally, pipeline is a series of stages where each connected by channels.
Where each stage is a group of goroutines that run the same function

Flow of each stage:
- receive a value from *upstream* via *inbound* channels
- performs some functions on data and producing new value
- passing/sending values *downstream* via *outbound* channels

The first stage called *the source* or *producer*
The last stage is called *the sink* or *consumer*