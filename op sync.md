Date: 2023-06-09 18:46

type: #process
links: 

# TL;DR
In the chain of messages that are sent in the process of syncing two nodes is largely symmetric for all node types (i.e. clients and backends). This details that series of messages. A proof for why the op sync process will not prevent infinite feedback loops.

# Framing
A node needs to fire off an [[OpSync]] to another node. It might be helpful to think about that firing node as a client that just got user input. We will see why we can generalize this to any node type. The [[OpSync]] contains everything needed to identify a tournament and the last known common state.

The next will receive the firing node's message. When comparing the receiving node's [[OpLog]] with [[OpSync]], the receiving node can find itself in one of two state:
 1) The first operation in the received [[OpSync]] is the same as the last operation in the log
 2) One or more [[Operation]]s have be added to the log since the last sync

If the first case, we have it easy. The receiving node can simply apply those operations to its log, send the original firing node a "op sync successful" message, and then forward the given sync to all other nodes it is connected to (for backends mostly). At this time, the sender and receiver have the same state, but the sender does not know that. For this reason, the client needs to periodically resend the original sync message. This is stop once the "op sync successful" message is received. The backend will know the "op sync successful" message was received once it stops seeing updates for that sync. This could also indicate a lost connection, so we can't know completely for sure until we receive a handful of messages for from that client. Lastly, note that the last step here put the receiving node in the firing node position since its now in the position of the original firing node at the start of this process.

Now, the second case. One or more operations have been added to the receiving node since these two have last synced. The receiving node will know this since the last [[Operation]] in its [[OpLog]] ought to be first [[Operation]] in the [[OpSync]]. The receiver will attempt to apply the sender's operations on top of its own. This too forks. 
 1) The sync errors
 2) The sync succeeds and returns a slice of [[Operation]]s whose first operation is the first operation of the sync.

If the sync errors, the receiving node simply responds with an [[OpProcessor]]. This processor contains the part of sender's original [[OpSync]]'s op slice, the partially-rectified log, the problematic operation, and the error that occurred. The original sender can ignore some or all of the operations it was trying to sync. The original sender responds with its decision. If there are still operations, the receiver continues to sync. This brings us either back to the start of this step (sync errors) or now the states are synced and the receiver sends back a "op sync successful" message.

If the initial syncing done by the receiver completes without error, the receiver needs to send the original sender an "op sync successful with new ops" message. The original sender now overwrites is history. This ends this chain.

One note here. If a node finalizes an update while it also has an on-going sync happening, its next message in the on-going sync must be "sync failed. new operations available". Any other messages associated with the failed sync will be either ignored or responded to with the "sync failed" message. The original sender of the chain should then start the chain over. This helps prevent two possible issues. First, it terminates communication about a sync if a server finalizes an update with another client. This makes the original client aware that it needs to start over. Second, the avoid problems with one client sending multiple syncs in a row. It is likely that the first sync task will finish first. This will cancel the other syncs. This will tell the client to send more new syncs (equal to the number of cancelled syncs). These will all be identical. One will process first. For the rest, be cancelled with the "sync failed. new operations available" message. So, that message needs to include the last operations in the log. If the original sending node's last operation is matches, then that node will know that the states are sync, despite the "confusing" message.

TODO: discuss what messages need to be tracked and resent periodically.
TODO: discuss how message chains are tracked and resolved in both clients and servers

# Example
While the process follows the same high-level pattern regardless if a server or backend started the sync chain, there are some key differences (most of which have to do with error resolution). As such, let's discuss each case in more detail.

## Client to Server Sync
Generally speaking, a client will be ready to send a new update once its user has triggered some sort of operation. When this happens, it will send an "initialize sync" message, which just contains an [[OpSync]]. The server can then send back one of a few messages:
 1) The was successful and no new operations existed
	 - The incoming operations are merged into the backend's log
	 - The backend returns a "completed" message with no new operations
 1) Backend had operations that the client didn't but the sync was successful
	 - The incoming operations are merged into the backend's log
	 - The backend returns an "completed" message with the new operations
 1) Backend had operations that the client didn't but the sync was successful
	 - The backend returns an "in-progress" message with the [[OpProcessor]].
 2) The backend has terminated the chain do to some internal error
	 - Often, this error will be that another update has occurred and the chain needs to be re-initialized by the client.
	 - Other errors can occur too, such as a `serde` error because the packets arrived incorrectly.
	 - The server will respond with whatever error arises

The first, second, and fourth cases don't not require the client to send back any message. This is because the client should be periodically sending duplicate messages in case either its original message or the response from the server got lost. The server will know that the client received the message or that the connection is lost when it stops receiving messages from the client on this chain. In either case, it is up to the client to re-establish the chain or a new chain.

In the third case, the client needs to alter its history of events. The [[OpProcessor]] that it receives contains three important things: the partially-rectified log, the remaining list of its operations, and the operation that caused the issue. The log contains all of the server's new operations and, potentially, some operations from the client (following the processor's algorithm). The client must decide what to do. It must either discard the problematic operation or it can discard both that operation and the rest of the log. In either case, it must send its [[OpDecision]] back to the server. The server can then encounter a few things while processing this decision:
 1) The decision just removed one operation and needs to be processed
	 a) The processing leads to a fully rectified log
	  - The backend returns an "completed" message with the new operations
	 b) The processing leads to another problem
	  - The backend returns an "in-progress" message with the [[OpProcessor]]
 2) The client removed all remaining operations
      - The backend merges this into its log
      - The backend returns an "completed" message with the new operations
 3) An error can occur (such as new operations arriving). The process for this is identical

The only case that don't terminate the chain is 1b. In this case, however, the chain repeats at the point were the client receives an [[OpProcessor]]. Each cycle of the process will remove one operation from the to-be processed log. Eventually, this process will end.

## Server to Client Sync
When updates are sent from server to client, the process is much simpler. The backend will never rollback its history due to a problematic sync. The backend is the "source of truth" for a tournament that needs to be synced between multiple clients. As you'll see, the client must rectify its history if a problem arises. It does this by tapping into the already established process of client-to-server syncing.

This process starts with the backend sending an "initialize sync" message to the client. This messages contains an [[OpSync]]. When the client receives this, it can find itself in one of a few states:
 1) The client has no new operations and can merge the logs without issue
   - The client updates it log and sends back a "sync successful" message
 2) The client might have outstanding operations that it has not synced with the server.
   - The client does not attempt to update.
   - It responds with a "sync aborted" message to signal
   - It will start a new sync chain by sending a new "initialize sync" message
 3) Some other, non-tournament errors might occur (such as a `serde` error)
   - The client will do nothing in this case

This is the whole process. In the first case, the sync is successful and nothing more need to happen. In the second case, the starts a new sync chain, which we know will eventually terminate. In the last case, the backend will continuously send the messages (to a limit) until it hears back from the client. If it hears nothing after `N` retries, the connection is assumed to be lost. Again, the client is responsible for restarting the connection.

# Notes
This process of ensuring message chains come to completion in bi-directional communication channels should be made into a crate.