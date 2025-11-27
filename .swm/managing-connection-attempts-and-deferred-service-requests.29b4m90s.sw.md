---
title: Managing Connection Attempts and Deferred Service Requests
---
This document describes how connection attempts to a remote service are managed. When a connection attempt is made, the result determines the next steps: if the attempt fails and retries are allowed, a new attempt is scheduled after a delay; if the attempt succeeds, any pending service requests are processed.

# Handling Connection Results and Scheduling Retries

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Was connection successful?"}
    click node1 openCode "src/ipc/client_impl.cc:154:169"
    node1 -->|"No and retry allowed"| node2["Initiating a New Socket Connection Attempt"]
    
    node1 -->|"Yes"| node3["Process queued service bindings"]
    click node3 openCode "src/ipc/client_impl.cc:171:182"
    
    subgraph loop1["For each queued service binding"]
      node3 -->|"Connected"| node4["Bind service"]
      click node4 openCode "src/ipc/client_impl.cc:178:179"
      node3 -->|"Not connected"| node5["Notify service of failure"]
      click node5 openCode "src/ipc/client_impl.cc:180:181"
    end
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node2 goToHeading "Initiating a New Socket Connection Attempt"
node2:::HeadingStyle

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1{"Was connection successful?"}
%%     click node1 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:154:169"
%%     node1 -->|"No and retry allowed"| node2["Initiating a New Socket Connection Attempt"]
%%     
%%     node1 -->|"Yes"| node3["Process queued service bindings"]
%%     click node3 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:171:182"
%%     
%%     subgraph loop1["For each queued service binding"]
%%       node3 -->|"Connected"| node4["Bind service"]
%%       click node4 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:178:179"
%%       node3 -->|"Not connected"| node5["Notify service of failure"]
%%       click node5 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:180:181"
%%     end
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
%% click node2 goToHeading "Initiating a New Socket Connection Attempt"
%% node2:::HeadingStyle
```

<SwmSnippet path="/src/ipc/client_impl.cc" line="154">

---

<SwmToken path="src/ipc/client_impl.cc" pos="154:4:4" line-data="void ClientImpl::OnConnect(base::UnixSocket*, bool connected) {">`OnConnect`</SwmToken> handles failed connection attempts by scheduling a retry with increasing delay, using a weak pointer to avoid use-after-free if the object is destroyed. <SwmToken path="src/ipc/client_impl.cc" pos="165:10:10" line-data="            static_cast&lt;ClientImpl&amp;&gt;(*weak_this).TryConnect();">`TryConnect`</SwmToken> is called next to actually perform the retry.

```c++
void ClientImpl::OnConnect(base::UnixSocket*, bool connected) {
  if (!connected && socket_retry_) {
    socket_backoff_ms_ =
        (socket_backoff_ms_ < 10000) ? socket_backoff_ms_ + 1000 : 30000;
    PERFETTO_DLOG(
        "Connection to traced's UNIX socket failed, retrying in %u seconds",
        socket_backoff_ms_ / 1000);
    auto weak_this = weak_ptr_factory_.GetWeakPtr();
    task_runner_->PostDelayedTask(
        [weak_this] {
          if (weak_this)
            static_cast<ClientImpl&>(*weak_this).TryConnect();
        },
        socket_backoff_ms_);
    return;
  }

```

---

</SwmSnippet>

## Initiating a New Socket Connection Attempt

<SwmSnippet path="/src/ipc/client_impl.cc" line="76">

---

<SwmToken path="src/ipc/client_impl.cc" pos="76:4:4" line-data="void ClientImpl::TryConnect() {">`TryConnect`</SwmToken> just kicks off a new connection attempt by calling <SwmToken path="src/ipc/client_impl.cc" pos="78:7:7" line-data="  sock_ = base::UnixSocket::Connect(">`UnixSocket`</SwmToken>::Connect with the relevant parameters. This hands off the actual socket connection work to the lower-level socket code, keeping the client logic clean.

```c++
void ClientImpl::TryConnect() {
  PERFETTO_DCHECK(socket_name_);
  sock_ = base::UnixSocket::Connect(
      socket_name_, this, task_runner_, base::GetSockFamily(socket_name_),
      base::SockType::kStream, base::SockPeerCredMode::kIgnore);
}
```

---

</SwmSnippet>

## Creating and Connecting the Socket Object

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Create new UNIX socket for communication"] --> node2["Attempt to connect to endpoint (socket name)"]
    click node1 openCode "src/base/unix_socket.cc:928:929"
    click node2 openCode "src/base/unix_socket.cc:930:930"
    node2 --> node3{"Is socket ready to connect?"}
    click node3 openCode "src/base/unix_socket.cc:1033:1034"
    node3 -->|"Yes"| node4["Begin connection process"]
    click node4 openCode "src/base/unix_socket.cc:1036:1037"
    node3 -->|"No"| node5["Notify connection failure and return socket (not connected)"]
    click node5 openCode "src/base/unix_socket.cc:1034:1034"
    node4 --> node6{"Did connection succeed or start asynchronously?"}
    click node6 openCode "src/base/unix_socket.cc:1036:1041"
    node6 -->|"Yes"| node7["Set connection state to Connecting"]
    click node7 openCode "src/base/unix_socket.cc:1041:1041"
    node6 -->|"No"| node5
    node7 --> node8["Return prepared socket for use"]
    click node8 openCode "src/base/unix_socket.cc:931:932"
    node5 --> node8
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Create new UNIX socket for communication"] --> node2["Attempt to connect to endpoint (socket name)"]
%%     click node1 openCode "<SwmPath>[src/base/unix_socket.cc](src/base/unix_socket.cc)</SwmPath>:928:929"
%%     click node2 openCode "<SwmPath>[src/base/unix_socket.cc](src/base/unix_socket.cc)</SwmPath>:930:930"
%%     node2 --> node3{"Is socket ready to connect?"}
%%     click node3 openCode "<SwmPath>[src/base/unix_socket.cc](src/base/unix_socket.cc)</SwmPath>:1033:1034"
%%     node3 -->|"Yes"| node4["Begin connection process"]
%%     click node4 openCode "<SwmPath>[src/base/unix_socket.cc](src/base/unix_socket.cc)</SwmPath>:1036:1037"
%%     node3 -->|"No"| node5["Notify connection failure and return socket (not connected)"]
%%     click node5 openCode "<SwmPath>[src/base/unix_socket.cc](src/base/unix_socket.cc)</SwmPath>:1034:1034"
%%     node4 --> node6{"Did connection succeed or start asynchronously?"}
%%     click node6 openCode "<SwmPath>[src/base/unix_socket.cc](src/base/unix_socket.cc)</SwmPath>:1036:1041"
%%     node6 -->|"Yes"| node7["Set connection state to Connecting"]
%%     click node7 openCode "<SwmPath>[src/base/unix_socket.cc](src/base/unix_socket.cc)</SwmPath>:1041:1041"
%%     node6 -->|"No"| node5
%%     node7 --> node8["Return prepared socket for use"]
%%     click node8 openCode "<SwmPath>[src/base/unix_socket.cc](src/base/unix_socket.cc)</SwmPath>:931:932"
%%     node5 --> node8
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/base/unix_socket.cc" line="921">

---

<SwmToken path="src/base/unix_socket.cc" pos="921:7:9" line-data="std::unique_ptr&lt;UnixSocket&gt; UnixSocket::Connect(">`UnixSocket::Connect`</SwmToken> allocates a new <SwmToken path="src/base/unix_socket.cc" pos="921:4:4" line-data="std::unique_ptr&lt;UnixSocket&gt; UnixSocket::Connect(">`UnixSocket`</SwmToken> object and immediately calls <SwmToken path="src/base/unix_socket.cc" pos="930:3:3" line-data="  sock-&gt;DoConnect(socket_name);">`DoConnect`</SwmToken> to start the connection process. Returning a <SwmToken path="src/base/unix_socket.cc" pos="921:2:2" line-data="std::unique_ptr&lt;UnixSocket&gt; UnixSocket::Connect(">`unique_ptr`</SwmToken> keeps ownership clear and avoids leaks.

```c++
std::unique_ptr<UnixSocket> UnixSocket::Connect(
    const std::string& socket_name,
    EventListener* event_listener,
    TaskRunner* task_runner,
    SockFamily sock_family,
    SockType sock_type,
    SockPeerCredMode peer_cred_mode) {
  std::unique_ptr<UnixSocket> sock(new UnixSocket(
      event_listener, task_runner, sock_family, sock_type, peer_cred_mode));
  sock->DoConnect(socket_name);
  return sock;
}
```

---

</SwmSnippet>

<SwmSnippet path="/src/base/unix_socket.cc" line="1029">

---

<SwmToken path="src/base/unix_socket.cc" pos="1029:4:4" line-data="void UnixSocket::DoConnect(const std::string&amp; socket_name) {">`DoConnect`</SwmToken> tries to connect the socket, and regardless of whether the connection is instant or pending, it posts a task to handle the connection event asynchronously. This keeps the logic consistent across platforms and avoids use-after-free by using a <SwmToken path="src/base/unix_socket.cc" pos="1051:1:1" line-data="  WeakPtr&lt;UnixSocket&gt; weak_ptr = weak_ptr_factory_.GetWeakPtr();">`WeakPtr`</SwmToken>.

```c++
void UnixSocket::DoConnect(const std::string& socket_name) {
  PERFETTO_DCHECK(state_ == State::kDisconnected);

  // This is the only thing that can gracefully fail in the ctor.
  if (!sock_raw_)
    return NotifyConnectionState(false);

  if (!sock_raw_.Connect(socket_name))
    return NotifyConnectionState(false);

  // At this point either connect() succeeded or started asynchronously
  // (errno = EINPROGRESS).
  state_ = State::kConnecting;

  // Even if the socket is non-blocking, connecting to a UNIX socket can be
  // acknowledged straight away rather than returning EINPROGRESS.
  // The decision here is to deal with the two cases uniformly, at the cost of
  // delaying the straight-away-connect() case by one task, to avoid depending
  // on implementation details of UNIX socket on the various OSes.
  // Posting the OnEvent() below emulates a wakeup of the FD watch. OnEvent(),
  // which knows how to deal with spurious wakeups, will poll the SO_ERROR and
  // evolve, if necessary, the state into either kConnected or kDisconnected.
  WeakPtr<UnixSocket> weak_ptr = weak_ptr_factory_.GetWeakPtr();
  task_runner_->PostTask([weak_ptr] {
    if (weak_ptr)
      weak_ptr->OnEvent();
  });
}
```

---

</SwmSnippet>

## Processing Deferred Service Bindings After Connection

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Drain queued service connection requests"]
    click node1 openCode "src/ipc/client_impl.cc:171:175"
    node1 --> loop1
    subgraph loop1["For each queued service"]
        node2{"Is client connected to host?"}
        click node2 openCode "src/ipc/client_impl.cc:176:177"
        node2 -->|"Yes"| node3["Connect service"]
        click node3 openCode "src/ipc/client_impl.cc:178:178"
        node2 -->|"No"| node4["Notify service of connection failure"]
        click node4 openCode "src/ipc/client_impl.cc:179:180"
    end
    loop1 --> node5["Function ends"]
    click node5 openCode "src/ipc/client_impl.cc:183:184"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Drain queued service connection requests"]
%%     click node1 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:171:175"
%%     node1 --> loop1
%%     subgraph loop1["For each queued service"]
%%         node2{"Is client connected to host?"}
%%         click node2 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:176:177"
%%         node2 -->|"Yes"| node3["Connect service"]
%%         click node3 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:178:178"
%%         node2 -->|"No"| node4["Notify service of connection failure"]
%%         click node4 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:179:180"
%%     end
%%     loop1 --> node5["Function ends"]
%%     click node5 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:183:184"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/ipc/client_impl.cc" line="171">

---

After returning from <SwmToken path="src/ipc/client_impl.cc" pos="76:4:4" line-data="void ClientImpl::TryConnect() {">`TryConnect`</SwmToken>, <SwmToken path="src/ipc/client_impl.cc" pos="173:3:3" line-data="  // OnConnect below might delete |this|, so move everything on the stack first.">`OnConnect`</SwmToken> processes any service bindings that were queued while waiting for a connection. It moves the queue to a local variable to avoid use-after-free if a callback deletes the client, then binds or notifies each service proxy depending on the connection result.

```c++
  // Drain the BindService() calls that were queued before establishing the
  // connection with the host. Note that if we got disconnected, the call to
  // OnConnect below might delete |this|, so move everything on the stack first.
  auto queued_bindings = std::move(queued_bindings_);
  queued_bindings_.clear();
  for (base::WeakPtr<ServiceProxy>& service_proxy : queued_bindings) {
    if (connected) {
      BindService(service_proxy);
    } else if (service_proxy) {
      service_proxy->OnConnect(false /* success */);
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/src/ipc/client_impl.cc" line="183">

---

<SwmToken path="src/ipc/client_impl.cc" pos="154:4:4" line-data="void ClientImpl::OnConnect(base::UnixSocket*, bool connected) {">`OnConnect`</SwmToken> ends here with a comment warning not to access member variables, since the object might have been deleted by callbacks above.

```c++
  // Don't access |this| below here.
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
