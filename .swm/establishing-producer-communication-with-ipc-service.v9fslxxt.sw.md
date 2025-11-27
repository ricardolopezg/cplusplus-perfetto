---
title: Establishing Producer Communication with IPC Service
---
This document describes how a producer establishes a communication channel with the IPC service to enable trace data transmission. The flow receives connection arguments, producer details, and shared memory configuration as input, and outputs a ready-to-use producer IPC client.

# Setting Up Producer IPC Client

<SwmSnippet path="/src/tracing/ipc/producer/producer_ipc_client_impl.cc" line="92">

---

In <SwmToken path="src/tracing/ipc/producer/producer_ipc_client_impl.cc" pos="92:0:0" line-data="ProducerIPCClientImpl::ProducerIPCClientImpl(">`ProducerIPCClientImpl`</SwmToken>, we kick off the setup by checking if shared memory is provided, binding the arbiter to the producer endpoint, and calculating the buffer page size. Then, depending on whether async socket creation is requested, we either set up the IPC channel asynchronously (posting a task to bind the service) or do it synchronously. This leads us to call <SwmToken path="src/tracing/ipc/producer/producer_ipc_client_impl.cc" pos="140:5:5" line-data="            weak_this-&gt;ipc_channel_-&gt;BindService(">`BindService`</SwmToken> in <SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>, which actually connects the producer port to the IPC service, letting us start communication.

```c++
ProducerIPCClientImpl::ProducerIPCClientImpl(
    ipc::Client::ConnArgs conn_args,
    Producer* producer,
    const std::string& producer_name,
    base::TaskRunner* task_runner,
    TracingService::ProducerSMBScrapingMode smb_scraping_mode,
    size_t shared_memory_size_hint_bytes,
    size_t shared_memory_page_size_hint_bytes,
    std::unique_ptr<SharedMemory> shm,
    std::unique_ptr<SharedMemoryArbiter> shm_arbiter,
    CreateSocketAsync create_socket_async)
    : producer_(producer),
      task_runner_(task_runner),
      receive_shmem_fd_cb_fuchsia_(
          std::move(conn_args.receive_shmem_fd_cb_fuchsia)),
      producer_port_(
          new protos::gen::ProducerPortProxy(this /* event_listener */)),
      shared_memory_(std::move(shm)),
      shared_memory_arbiter_(std::move(shm_arbiter)),
      name_(producer_name),
      shared_memory_page_size_hint_bytes_(shared_memory_page_size_hint_bytes),
      shared_memory_size_hint_bytes_(shared_memory_size_hint_bytes),
      smb_scraping_mode_(smb_scraping_mode) {
  // Check for producer-provided SMB (used by Chrome for startup tracing).
  if (shared_memory_) {
    // We also expect a valid (unbound) arbiter. Bind it to this endpoint now.
    PERFETTO_CHECK(shared_memory_arbiter_);
    shared_memory_arbiter_->BindToProducerEndpoint(this, task_runner_);

    // If the service accepts our SMB, then it must match our requested page
    // layout. The protocol doesn't allow the service to change the size and
    // layout when the SMB is provided by the producer.
    shared_buffer_page_size_kb_ = shared_memory_page_size_hint_bytes_ / 1024;
  }

  if (create_socket_async) {
    PERFETTO_DCHECK(conn_args.socket_name);
    auto weak_this = weak_factory_.GetWeakPtr();
    create_socket_async(
        [weak_this, task_runner = task_runner_](base::SocketHandle fd) {
          task_runner->PostTask([weak_this, fd] {
            base::ScopedSocketHandle handle(fd);
            if (!weak_this) {
              return;
            }
            ipc::Client::ConnArgs args(std::move(handle));
            weak_this->ipc_channel_ = ipc::Client::CreateInstance(
                std::move(args), weak_this->task_runner_);
            weak_this->ipc_channel_->BindService(
                weak_this->producer_port_->GetWeakPtr());
          });
        });
  } else {
    ipc_channel_ =
        ipc::Client::CreateInstance(std::move(conn_args), task_runner);
    ipc_channel_->BindService(producer_port_->GetWeakPtr());
  }
```

---

</SwmSnippet>

## Connecting Producer Port to IPC Service

<SwmSnippet path="/src/ipc/client_impl.cc" line="83">

---

In <SwmToken path="src/ipc/client_impl.cc" pos="83:4:4" line-data="void ClientImpl::BindService(base::WeakPtr&lt;ServiceProxy&gt; service_proxy) {">`BindService`</SwmToken>, we check if the service proxy is valid and if the socket is connected. If not, we queue the binding for later. If connected, we send a <SwmToken path="src/ipc/client_impl.cc" pos="83:4:4" line-data="void ClientImpl::BindService(base::WeakPtr&lt;ServiceProxy&gt; service_proxy) {">`BindService`</SwmToken> frame to the service, which kicks off the connection process and triggers the service proxy's <SwmToken path="src/ipc/client_impl.cc" pos="98:5:5" line-data="    return service_proxy-&gt;OnConnect(false /* success */);">`OnConnect`</SwmToken> callback based on the result.

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

### Handling Service Connection Result

See <SwmLink doc-title="Managing Connection Attempts and Service Bindings">[Managing Connection Attempts and Service Bindings](/.swm/managing-connection-attempts-and-service-bindings.5tqo25x4.sw.md)</SwmLink>

### Tracking <SwmToken path="src/tracing/ipc/producer/producer_ipc_client_impl.cc" pos="140:5:5" line-data="            weak_this-&gt;ipc_channel_-&gt;BindService(">`BindService`</SwmToken> Requests

<SwmSnippet path="/src/ipc/client_impl.cc" line="100">

---

After returning from <SwmToken path="src/tracing/ipc/producer/producer_ipc_client_impl.cc" pos="140:5:5" line-data="            weak_this-&gt;ipc_channel_-&gt;BindService(">`BindService`</SwmToken> in <SwmPath>[src/ipc/client_impl.cc](src/ipc/client_impl.cc)</SwmPath>, we store the details of the <SwmToken path="src/tracing/ipc/producer/producer_ipc_client_impl.cc" pos="140:5:5" line-data="            weak_this-&gt;ipc_channel_-&gt;BindService(">`BindService`</SwmToken> request in <SwmToken path="src/ipc/client_impl.cc" pos="104:1:1" line-data="  queued_requests_.emplace(request_id, std::move(qr));">`queued_requests_`</SwmToken> so we can match incoming responses to the correct service proxy and handle connection results properly.

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

## Finalizing Producer IPC Client Setup

<SwmSnippet path="/src/tracing/ipc/producer/producer_ipc_client_impl.cc" line="149">

---

After returning from <SwmToken path="src/tracing/ipc/producer/producer_ipc_client_impl.cc" pos="140:5:5" line-data="            weak_this-&gt;ipc_channel_-&gt;BindService(">`BindService`</SwmToken> in <SwmToken path="src/tracing/ipc/producer/producer_ipc_client_impl.cc" pos="92:0:0" line-data="ProducerIPCClientImpl::ProducerIPCClientImpl(">`ProducerIPCClientImpl`</SwmToken>, we verify that all setup happened on the correct thread, making sure the client is ready for further operations without thread safety issues.

```c++
  PERFETTO_DCHECK_THREAD(thread_checker_);
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
