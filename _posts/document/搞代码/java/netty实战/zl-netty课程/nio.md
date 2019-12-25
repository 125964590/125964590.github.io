NIO架构

- Channelconnections to filel, socket ETC that support non-blocking reads;
- Bufffers Array-like objects that can be directly read or written by channel;
- Selectors tell which of a set of channels have IO events;
- SelectionKeys Maintain IO event status and bindings.

### Coordinating Tasks

- Handoffs:Each task enables, triggers, or calls next one Usually fastest but can be brittle;
- Callbacks:to pre-handler despatcher Sets state,attachment,etc a variant of GoF Mediator pattern;
- Queues:For example, passing buffers accross stages;
- Futures:When each task produces a result coordination layered on top of join or wait/notify.



## ChannelPipline

An outbound event is handler by the out-bound handler in the top-down