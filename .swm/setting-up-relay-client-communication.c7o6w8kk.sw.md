---
title: Setting Up Relay Client Communication
---
This document describes how a relay client is set up to communicate with an IPC service. By initializing the relay proxy and binding it to an IPC channel, the system enables message exchange necessary for tracing and profiling. The process takes connection arguments, an event listener, and a task runner as input, and results in an active IPC channel with a bound relay proxy.

# Setting up the relay client and IPC channel

<SwmSnippet path="/src/tracing/ipc/producer/relay_ipc_client.cc" line="27">

---

In <SwmToken path="src/tracing/ipc/producer/relay_ipc_client.cc" pos="27:0:0" line-data="RelayIPCClient::RelayIPCClient(ipc::Client::ConnArgs conn_args,">`RelayIPCClient`</SwmToken>, we're setting up the relay proxy and IPC channel, then binding the relay proxy to the channel. This hands off control to the IPC layer, so we need to call into <SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath> to actually connect the service and start communication.

```c++
RelayIPCClient::RelayIPCClient(ipc::Client::ConnArgs conn_args,
                               base::WeakPtr<EventListener> listener,
                               base::TaskRunner* task_runner)
    : listener_(std::move(listener)),
      task_runner_(task_runner),
      ipc_channel_(
          ipc::Client::CreateInstance(std::move(conn_args), task_runner)),
      relay_proxy_(new protos::gen::RelayPortProxy(this /* event_listener */)) {
  ipc_channel_->BindService(relay_proxy_->GetWeakPtr());
```

---

</SwmSnippet>

## Connecting the service proxy to the IPC channel

<SwmSnippet path="/src/ipc/client_impl.cc" line="83">

---

In <SwmToken path="src/ipc/client_impl.cc" pos="83:4:4" line-data="void ClientImpl::BindService(base::WeakPtr&lt;ServiceProxy&gt; service_proxy) {">`BindService`</SwmToken>, we're checking if the service proxy and socket are ready. If not, we queue the binding. If everything's good, we send a frame to bind the service, which lets the relay proxy start handling IPC messages.

```c++
void ClientImpl::BindService(base::WeakPtr<ServiceProxy> service_proxy) {
  if (!service_proxy)
    return;
  if (!sock_->is_connected()) {
    queued_bindings_.emplace_back(service_proxy);
    return;
  }
  RequestID request_id = ++last_request_id_;
  Frame frame;
  frame.set_request_id(request_id);
  Frame::BindService* req = frame.mutable_msg_bind_service();
  const char* const service_name = service_proxy->GetDescriptor().service_name;
  req->set_service_name(service_name);
  if (!SendFrame(frame)) {
    PERFETTO_DLOG("BindService(%s) failed", service_name);
    return service_proxy->OnConnect(false /* success */);
  }
```

---

</SwmSnippet>

### Handling service connection result

See <SwmLink doc-title="Managing Connection Attempts and Deferred Service Requests">[Managing Connection Attempts and Deferred Service Requests](/.swm/managing-connection-attempts-and-deferred-service-requests.29b4m90s.sw.md)</SwmLink>

### Tracking pending service requests

<SwmSnippet path="/src/ipc/client_impl.cc" line="100">

---

After returning from <SwmToken path="src/tracing/ipc/producer/relay_ipc_client.cc" pos="35:3:3" line-data="  ipc_channel_-&gt;BindService(relay_proxy_-&gt;GetWeakPtr());">`BindService`</SwmToken> in <SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>, we store the request details so we can match incoming responses to the right service proxy later.

```c++
  QueuedRequest qr;
  qr.type = Frame::kMsgBindServiceFieldNumber;
  qr.request_id = request_id;
  qr.service_proxy = service_proxy;
  queued_requests_.emplace(request_id, std::move(qr));
}
```

---

</SwmSnippet>

## Verifying thread context after service binding

<SwmSnippet path="/src/tracing/ipc/producer/relay_ipc_client.cc" line="36">

---

After returning from <SwmToken path="src/tracing/ipc/producer/relay_ipc_client.cc" pos="35:3:3" line-data="  ipc_channel_-&gt;BindService(relay_proxy_-&gt;GetWeakPtr());">`BindService`</SwmToken> in <SwmToken path="src/tracing/ipc/producer/relay_ipc_client.cc" pos="27:0:0" line-data="RelayIPCClient::RelayIPCClient(ipc::Client::ConnArgs conn_args,">`RelayIPCClient`</SwmToken>, we verify that we're still on the correct thread, which keeps things safe and predictable for further IPC operations.

```c++
  PERFETTO_DCHECK_THREAD(thread_checker_);
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
