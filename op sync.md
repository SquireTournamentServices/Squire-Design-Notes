Date: 2023-06-09 18:46

type: #process
links: 

# TL;DR
In the chain of messages that are sent in the process of syncing two nodes is largely semetric for all node types (i.e. clients and backends). This details that series of messages. A proof for why the op sync process will not prevent inifinite feedback loops.

# Framing
A node needs to fire off an [[OpSync]] to another node. It might be helpful to think about that firing node as a client that just got user input. We will see why we can generalize this to any node type. The [[OpSync]] contains everything needed to identify a tournament and the last known common state.

The next will receive the firing node's message. When comparing the receiving node's [[OpLog]] with [[OpSync]], the receiving node can find itself in one of two state:
 1) The first operation in the received [[OpSync]] is the same as the last operation in the log
 2) One or more [[Operation]]s have be added to the log since the last sync

If the first case, we have it easy. The receiving node can simply apply those operations to its log, send the original firing node a "op sync successful" message, and then forward the given sync to all other nodes it is connected to (for backends mostly). At this time, the sender and receiver have the same state, but the sender does not know that. For this reason, the client needs to periodically resend the original sync message. This is stop once the "op sync successful" message is received. The backend will know the "op sync successful" message was received once it stops seeing updates for that sync. This could also indicate a lost connection, so we can't know completely for sure until we receive a handful of messages for from that client. Lastly, note that the last step here put the receiving node in the firing node position since its now in the position of the original firing node at the start of this process.

Now, the second case. One or more operations have been added to the receiving node since these two have last synced. The receiving node will know this since the last [[Operation]] in its [[OpLog]] ought to be first [[Operation]] in the [[OpSync]]. The receiver will attempt to apply the sender's operations on top of its own. This too forks. 
 1) The sync errors
 2) The sync succeeds and returns a slice of [[Operation]]s whose first operation is the first operation of the sync.

If the sync errors, the receiving node simply responds with an [[OpProcessor]]. This processor contains the part of sender's original [[OpSync]]'s op slice, the partially-rectified log, the problematic operation, and the error that occured. The original sender can ignore some or all of the operations it was trying to sync. The original sender resonds with its decision. If there are still operations, the receiver continues to sync. This brings us either back to the start of this step (sync errors) or now the states are synced and the receiver sends back a "op sync successful" message.

If the inital syncing done by the receiver completes without error, the receiver needs to send the original sender an "op sync successful with new ops" message. The original sender now overwrites is history. This ends this chain.

One note here. If a node finalizes an update while it also has an on-going sync happening, its next message in the on-going sync must be "sync failed. new operations available". Any other messages associated with the failed sync will be either ignored or responded to with the "sync failed" message. The original sender of the chain should then start the chain over. This helps prevent two possible issues. First, it terminates communication about a sync if a server finalizes an update with another client. This makes the original client aware that it needs to start over. Second, the avoid problems with one client sending multiple syncs in a row. It is likely that the first sync task will finish first. This will cancel the other syncs. This will tell the client to send more new syncs (equal to the number of cancelled syncs). These will all be identical. One will process first. For the rest, be cancelled with the "sync failed. new operations available" message. So, that message needs to include the last operations in the log. If the original sending node's last operation is matches, then that node will know that the states are sync, dispite the "consusing" message.

TODO: discuss what messages need to be tracked and resent periodically.

# Example


# Notes
This process of ensuring message chains come to completion in bi-directional communication channels should be made into a crate.