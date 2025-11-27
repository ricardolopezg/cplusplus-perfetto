---
title: Managing service binding replies and queued connections
---
This document explains how the system manages replies to service binding requests and processes queued service bindings. When a reply is received, the system determines if the binding can proceed, registers the service if successful, and notifies the relevant component. Queued binding requests are managed and retried as needed to ensure valid service registration and timely notifications.

# Handling service binding replies

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1{"Is service proxy valid?"}
  click node1 openCode "src/ipc/client_impl.cc:267:269"
  node1 -->|"No"| node2["Return"]
  click node2 openCode "src/ipc/client_impl.cc:269:269"
  node1 -->|"Yes"| node3{"Did reply indicate success?"}
  click node3 openCode "src/ipc/client_impl.cc:271:274"
  node3 -->|"No"| node4["Notify failure to connect"]
  click node4 openCode "src/ipc/client_impl.cc:272:273"
  node3 -->|"Yes"| node5{"Is another service already bound with same ID?"}
  click node5 openCode "src/ipc/client_impl.cc:276:283"
  node5 -->|"Yes"| node6["Notify failure to connect"]
  click node6 openCode "src/ipc/client_impl.cc:278:282"
  node5 -->|"No"| node7["Build method map for service_name/service_id"]
  click node7 openCode "src/ipc/client_impl.cc:285:294"
  subgraph loop1["For each method in reply"]
    node8{"Is method valid?"}
    click node8 openCode "src/ipc/client_impl.cc:288:291"
    node8 -->|"No"| node9["Skip method"]
    click node9 openCode "src/ipc/client_impl.cc:289:291"
    node8 -->|"Yes"| node10["Add method to map"]
    click node10 openCode "src/ipc/client_impl.cc:293:293"
  end
  node7 --> node11["Initialize binding with method map"]
  click node11 openCode "src/ipc/client_impl.cc:295:296"
  node11 --> node12["Register service binding"]
  click node12 openCode "src/ipc/client_impl.cc:297:297"
  node12 --> node13["Notify success to connect"]
  click node13 openCode "src/ipc/client_impl.cc:298:298"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1{"Is service proxy valid?"}
%%   click node1 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:267:269"
%%   node1 -->|"No"| node2["Return"]
%%   click node2 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:269:269"
%%   node1 -->|"Yes"| node3{"Did reply indicate success?"}
%%   click node3 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:271:274"
%%   node3 -->|"No"| node4["Notify failure to connect"]
%%   click node4 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:272:273"
%%   node3 -->|"Yes"| node5{"Is another service already bound with same ID?"}
%%   click node5 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:276:283"
%%   node5 -->|"Yes"| node6["Notify failure to connect"]
%%   click node6 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:278:282"
%%   node5 -->|"No"| node7["Build method map for service_name/service_id"]
%%   click node7 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:285:294"
%%   subgraph loop1["For each method in reply"]
%%     node8{"Is method valid?"}
%%     click node8 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:288:291"
%%     node8 -->|"No"| node9["Skip method"]
%%     click node9 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:289:291"
%%     node8 -->|"Yes"| node10["Add method to map"]
%%     click node10 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:293:293"
%%   end
%%   node7 --> node11["Initialize binding with method map"]
%%   click node11 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:295:296"
%%   node11 --> node12["Register service binding"]
%%   click node12 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:297:297"
%%   node12 --> node13["Notify success to connect"]
%%   click node13 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:298:298"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/ipc/client_impl.cc" line="265">

---

In <SwmToken path="src/ipc/client_impl.cc" pos="265:4:4" line-data="void ClientImpl::OnBindServiceReply(QueuedRequest req,">`OnBindServiceReply`</SwmToken>, we start by validating the service proxy and reply, handle failure cases, check for duplicate service bindings, and build a map of method names to remote IDs from the reply. This sets up the context for binding the service and enables method invocation by name.

```c++
void ClientImpl::OnBindServiceReply(QueuedRequest req,
                                    const Frame::BindServiceReply& reply) {
  base::WeakPtr<ServiceProxy>& service_proxy = req.service_proxy;
  if (!service_proxy)
    return;
  const char* svc_name = service_proxy->GetDescriptor().service_name;
  if (!reply.success()) {
    PERFETTO_DLOG("BindService(): unknown service_name=\"%s\"", svc_name);
    return service_proxy->OnConnect(false /* success */);
  }

  auto prev_service = service_bindings_.find(reply.service_id());
  if (prev_service != service_bindings_.end() && prev_service->second.get()) {
    PERFETTO_DLOG(
        "BindService(): Trying to bind service \"%s\" but another service "
        "named \"%s\" is already bound with the same ID.",
        svc_name, prev_service->second->GetDescriptor().service_name);
    return service_proxy->OnConnect(false /* success */);
  }

  // Build the method [name] -> [remote_id] map.
  std::map<std::string, MethodID> methods;
  for (const auto& method : reply.methods()) {
    if (method.name().empty() || method.id() <= 0) {
      PERFETTO_DLOG("OnBindServiceReply(): invalid method \"%s\" -> %" PRIu64,
                    method.name().c_str(), static_cast<uint64_t>(method.id()));
      continue;
    }
    methods[method.name()] = method.id();
  }
```

---

</SwmSnippet>

<SwmSnippet path="/src/ipc/client_impl.cc" line="295">

---

After setting up the binding and method map, we notify the service proxy of a successful connection by calling <SwmToken path="src/ipc/client_impl.cc" pos="298:3:3" line-data="  service_proxy-&gt;OnConnect(true /* success */);">`OnConnect`</SwmToken>(true). This lets the proxy know it can start interacting with the bound service.

```c++
  service_proxy->InitializeBinding(weak_ptr_factory_.GetWeakPtr(),
                                   reply.service_id(), std::move(methods));
  service_bindings_[reply.service_id()] = service_proxy;
  service_proxy->OnConnect(true /* success */);
}
```

---

</SwmSnippet>

# Managing connection attempts and queued bindings

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Connection attempt"] --> node2{"Was connection successful?"}
    click node1 openCode "src/ipc/client_impl.cc:154:155"
    node2 -->|"No"| node3{"Is retry enabled?"}
    click node2 openCode "src/ipc/client_impl.cc:155:155"
    node3 -->|"Yes"| node4["Schedule retry with increased delay (add 1s up to 10s, then 30s)"]
    click node3 openCode "src/ipc/client_impl.cc:156:167"
    node4 --> node9["End"]
    click node4 openCode "src/ipc/client_impl.cc:168:169"
    node3 -->|"No"| node5["Process queued service bindings"]
    node2 -->|"Yes"| node5["Process queued service bindings"]
    click node5 openCode "src/ipc/client_impl.cc:171:175"
    subgraph loop1["For each queued service binding"]
        node5 --> node6{"Is connection successful?"}
        click node6 openCode "src/ipc/client_impl.cc:177:177"
        node6 -->|"Yes"| node7["Bind service"]
        click node7 openCode "src/ipc/client_impl.cc:178:178"
        node6 -->|"No"| node8["Notify binding of failure"]
        click node8 openCode "src/ipc/client_impl.cc:180:181"
    end
    node5 --> node10["End"]
    click node10 openCode "src/ipc/client_impl.cc:183:184"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Connection attempt"] --> node2{"Was connection successful?"}
%%     click node1 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:154:155"
%%     node2 -->|"No"| node3{"Is retry enabled?"}
%%     click node2 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:155:155"
%%     node3 -->|"Yes"| node4["Schedule retry with increased delay (add 1s up to 10s, then 30s)"]
%%     click node3 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:156:167"
%%     node4 --> node9["End"]
%%     click node4 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:168:169"
%%     node3 -->|"No"| node5["Process queued service bindings"]
%%     node2 -->|"Yes"| node5["Process queued service bindings"]
%%     click node5 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:171:175"
%%     subgraph loop1["For each queued service binding"]
%%         node5 --> node6{"Is connection successful?"}
%%         click node6 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:177:177"
%%         node6 -->|"Yes"| node7["Bind service"]
%%         click node7 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:178:178"
%%         node6 -->|"No"| node8["Notify binding of failure"]
%%         click node8 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:180:181"
%%     end
%%     node5 --> node10["End"]
%%     click node10 openCode "<SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>:183:184"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/ipc/client_impl.cc" line="154">

---

In <SwmToken path="src/ipc/client_impl.cc" pos="154:4:4" line-data="void ClientImpl::OnConnect(base::UnixSocket*, bool connected) {">`OnConnect`</SwmToken>, we handle connection failures with exponential backoff and schedule retries safely using weak pointers. We also drain and process queued service bindings, making sure to avoid accessing 'this' after potentially destructive calls.

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

After draining the queue and notifying service proxies, we avoid touching 'this' since the object might have been deleted during those calls.

```c++
  // Don't access |this| below here.
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
