---
title: Profiling Data Collection and Export Flow
---
This document outlines how users collect and analyze profiling data using Perfetto. Users provide arguments to define profiling parameters, review or record the configuration, and receive processed results for analysis.

```mermaid
flowchart TD
  node1["Parsing arguments and preparing profiling setup"]:::HeadingStyle
  click node1 goToHeading "Parsing arguments and preparing profiling setup"
  node1 --> node2["Building or loading the Perfetto profiling configuration"]:::HeadingStyle
  click node2 goToHeading "Building or loading the Perfetto profiling configuration"
  node2 --> node3{"Print configuration or record trace?
(Printing or recording the trace based on user request)"}:::HeadingStyle
  click node3 goToHeading "Printing or recording the trace based on user request"
  node3 -->|"Print"| node3
  node3 -->|"Record"| node4["Running and monitoring the profiling session on device"]:::HeadingStyle
  click node4 goToHeading "Running and monitoring the profiling session on device"
  node4 --> node5["Post-processing and exporting profiling results"]:::HeadingStyle
  click node5 goToHeading "Post-processing and exporting profiling results"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Parsing arguments and preparing profiling setup

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Prepare profile target and trace configuration"]
  click node1 openCode "python/tools/cpu_profile.py:506:509"
  node1 --> node2{"Print configuration?"}
  
  node2 -->|"Yes"| node3["Printing or recording the trace based on user request"]
  
  node2 -->|"No"| node4["Post-processing and exporting profiling results"]
  
  node4 --> node5["Post-processing and exporting profiling results"]
  
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node2 goToHeading "Printing or recording the trace based on user request"
node2:::HeadingStyle
click node3 goToHeading "Printing or recording the trace based on user request"
node3:::HeadingStyle
click node4 goToHeading "Post-processing and exporting profiling results"
node4:::HeadingStyle
click node5 goToHeading "Post-processing and exporting profiling results"
node5:::HeadingStyle

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Prepare profile target and trace configuration"]
%%   click node1 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:506:509"
%%   node1 --> node2{"Print configuration?"}
%%   
%%   node2 -->|"Yes"| node3["Printing or recording the trace based on user request"]
%%   
%%   node2 -->|"No"| node4["Post-processing and exporting profiling results"]
%%   
%%   node4 --> node5["Post-processing and exporting profiling results"]
%%   
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
%% click node2 goToHeading "Printing or recording the trace based on user request"
%% node2:::HeadingStyle
%% click node3 goToHeading "Printing or recording the trace based on user request"
%% node3:::HeadingStyle
%% click node4 goToHeading "Post-processing and exporting profiling results"
%% node4:::HeadingStyle
%% click node5 goToHeading "Post-processing and exporting profiling results"
%% node5:::HeadingStyle
```

<SwmSnippet path="/python/tools/cpu_profile.py" line="506">

---

In <SwmToken path="python/tools/cpu_profile.py" pos="506:2:2" line-data="def main(argv):">`main`</SwmToken>, we parse arguments, prep the profiling target, and immediately fetch the Perfetto config. We need to call <SwmToken path="python/tools/cpu_profile.py" pos="509:5:5" line-data="  trace_config = get_perfetto_config(args)">`get_perfetto_config`</SwmToken> next because it builds or loads the config that tells Perfetto what and how to profile, based on user input or defaults.

```python
def main(argv):
  args = parse_and_validate_args()
  profile_target = get_and_prepare_profile_target(args)
  trace_config = get_perfetto_config(args)
```

---

</SwmSnippet>

## Building or loading the Perfetto profiling configuration

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Start: Generate Perfetto config"] --> node2{"User provided config file?"}
  click node1 openCode "python/tools/cpu_profile.py:199:206"
  node2 -->|"Yes"| node3["Load and return user config file"]
  click node2 openCode "python/tools/cpu_profile.py:206:210"
  click node3 openCode "python/tools/cpu_profile.py:208:209"
  node2 -->|"No"| node4{"Any matching processes?"}
  click node4 openCode "python/tools/cpu_profile.py:240:245"
  node4 -->|"No"| node5["Exit: No processes matched"]
  click node5 openCode "python/tools/cpu_profile.py:245:245"
  node4 -->|"Yes"| node6["Prepare config with user options (frequency, duration, kernel frames)"]
  click node6 openCode "python/tools/cpu_profile.py:213:239"

  subgraph loop1["For each profiling event"]
    node6 --> node7["Add event-specific config"]
    click node7 openCode "python/tools/cpu_profile.py:250:273"
  end
  node7 --> node8["Format final config"]
  click node8 openCode "python/tools/cpu_profile.py:286:290"
  node8 --> node9{"Print process names?"}
  click node9 openCode "python/tools/cpu_profile.py:280:284"
  node9 -->|"Yes"| node10["For each matching process, print name"]
  click node10 openCode "python/tools/cpu_profile.py:282:283"
  node10 --> node11["Return generated config"]
  click node11 openCode "python/tools/cpu_profile.py:292:292"
  node9 -->|"No"| node11
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Start: Generate Perfetto config"] --> node2{"User provided config file?"}
%%   click node1 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:199:206"
%%   node2 -->|"Yes"| node3["Load and return user config file"]
%%   click node2 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:206:210"
%%   click node3 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:208:209"
%%   node2 -->|"No"| node4{"Any matching processes?"}
%%   click node4 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:240:245"
%%   node4 -->|"No"| node5["Exit: No processes matched"]
%%   click node5 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:245:245"
%%   node4 -->|"Yes"| node6["Prepare config with user options (frequency, duration, kernel frames)"]
%%   click node6 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:213:239"
%% 
%%   subgraph loop1["For each profiling event"]
%%     node6 --> node7["Add event-specific config"]
%%     click node7 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:250:273"
%%   end
%%   node7 --> node8["Format final config"]
%%   click node8 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:286:290"
%%   node8 --> node9{"Print process names?"}
%%   click node9 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:280:284"
%%   node9 -->|"Yes"| node10["For each matching process, print name"]
%%   click node10 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:282:283"
%%   node10 --> node11["Return generated config"]
%%   click node11 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:292:292"
%%   node9 -->|"No"| node11
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/python/tools/cpu_profile.py" line="199">

---

In <SwmToken path="python/tools/cpu_profile.py" pos="199:2:2" line-data="def get_perfetto_config(args):">`get_perfetto_config`</SwmToken>, we either load a user-supplied config file or build a config string with fixed and dynamic sections. We match process names if given, and bail if none are found. The config embeds constants and formats in blocks for each event, controlling what gets profiled and how.

```python
def get_perfetto_config(args):
  """Returns a Perfetto config with CPU profiling enabled for the selected
  processes.

  Args:
    args: The command-line arguments provided to this script.
  """
  if args.config is not None:
    try:
      with open(args.config, 'r') as config_file:
        return config_file.read()
    except IOError as error:
      sys.exit("Unable to read config file: {}".format(error))

  CONFIG_INDENT = '          '
  CONFIG = textwrap.dedent('''\
  buffers {{
    size_kb: 2048
  }}

  buffers {{
    size_kb: 63488
  }}

  data_sources {{
    config {{
      name: "linux.process_stats"
      target_buffer: 0
      process_stats_config {{
        proc_stats_poll_ms: 100
      }}
    }}
  }}

  duration_ms: {duration}
  write_into_file: true
  flush_timeout_ms: 30000
  flush_period_ms: 604800000
  ''')

  matching_processes = []
  if args.name is not None:
    names_to_match = [name.strip() for name in args.name.split(',')]
    matching_processes = get_matching_processes(args, names_to_match)

  if not matching_processes:
    sys.exit("No running processes matched for profiling.")

  target_config = "\n".join(
      [f'{CONFIG_INDENT}target_cmdline: "{p}"' for p in matching_processes])

  events = args.event or ['SW_CPU_CLOCK']
  for event in events:
    CONFIG += (
        textwrap.dedent('''
    data_sources {{
      config {{
        name: "linux.perf"
        target_buffer: 1
        perf_event_config {{
          timebase {{
            counter: %s
            frequency: {frequency}
            timestamp_clock: PERF_CLOCK_MONOTONIC
          }}
          callstack_sampling {{
            scope {{
    {target_config}
            }}
            kernel_frames: {kernel_config}
          }}
        }}
      }}
    }}
    ''') % (event))
```

---

</SwmSnippet>

<SwmSnippet path="/python/tools/cpu_profile.py" line="275">

---

Here we set whether kernel frames are included in the config, based on args. We also print the list of matched processes unless <SwmToken path="python/tools/cpu_profile.py" pos="280:7:7" line-data="  if not args.print_config:">`print_config`</SwmToken> is set, so the user knows what will be profiled.

```python
  if args.kernel_frames:
    kernel_config = "true"
  else:
    kernel_config = "false"

  if not args.print_config:
    print("Configured profiling for these processes:\n")
    for matching_process in matching_processes:
      print(matching_process)
```

---

</SwmSnippet>

<SwmSnippet path="/python/tools/cpu_profile.py" line="283">

---

Finally we format the config string with all the user-supplied and computed values, then return it for use in profiling.

```python
      print(matching_process)
    print()

  config = CONFIG.format(
      frequency=args.frequency,
      duration=args.duration,
      target_config=target_config,
      kernel_config=kernel_config)

  return config
```

---

</SwmSnippet>

## Printing or recording the trace based on user request

<SwmSnippet path="/python/tools/cpu_profile.py" line="510">

---

Back in <SwmToken path="python/tools/cpu_profile.py" pos="506:2:2" line-data="def main(argv):">`main`</SwmToken>, after getting the config, we either print it and exit if requested, or call <SwmToken path="python/tools/cpu_profile.py" pos="513:1:1" line-data="  record_trace(trace_config, profile_target)">`record_trace`</SwmToken> to actually start profiling. Calling <SwmToken path="python/tools/cpu_profile.py" pos="513:1:1" line-data="  record_trace(trace_config, profile_target)">`record_trace`</SwmToken> is what kicks off the trace collection on the device.

```python
  if args.print_config:
    print(trace_config)
    return 0
  record_trace(trace_config, profile_target)
```

---

</SwmSnippet>

## Running and monitoring the profiling session on device

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Check Android version"]
  click node1 openCode "python/tools/cpu_profile.py:341:342"
  node1 --> node2{"Is Android T or newer?"}
  click node2 openCode "python/tools/cpu_profile.py:341:342"
  node2 -->|"Yes"| node3["Push tracing config to device"]
  click node3 openCode "python/tools/cpu_profile.py:347:353"
  node2 -->|"No"| node4["Exit: Device not supported"]
  click node4 openCode "python/tools/cpu_profile.py:342:342"
  node3 --> node5["Start Perfetto profiling"]
  click node5 openCode "python/tools/cpu_profile.py:358:364"
  node5 --> node6["Profiling active"]
  click node6 openCode "python/tools/cpu_profile.py:368:368"

  subgraph loop1["While profiling is active"]
    node6 --> node7{"Was profiling interrupted?"}
    click node7 openCode "python/tools/cpu_profile.py:373:376"
    node7 -->|"No"| node6
    node7 -->|"Yes"| node8["Send interrupt signal"]
    click node8 openCode "python/tools/cpu_profile.py:381:381"
  end

  node8 --> node9["Restore signal handler"]
  click node9 openCode "python/tools/cpu_profile.py:384:384"

  subgraph loop2["Wait for profiling to finish"]
    node9 --> node10["Check if profiling process is still running"]
    click node10 openCode "python/tools/cpu_profile.py:386:389"
    node10 -->|"Still running"| node10
    node10 -->|"Finished"| node11["Retrieve trace results"]
    click node11 openCode "python/tools/cpu_profile.py:391:392"
  end

  node11 --> node12["Clean up trace and config files"]
  click node12 openCode "python/tools/cpu_profile.py:393:394"
  node12 --> node13["Done"]
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Check Android version"]
%%   click node1 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:341:342"
%%   node1 --> node2{"Is Android T or newer?"}
%%   click node2 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:341:342"
%%   node2 -->|"Yes"| node3["Push tracing config to device"]
%%   click node3 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:347:353"
%%   node2 -->|"No"| node4["Exit: Device not supported"]
%%   click node4 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:342:342"
%%   node3 --> node5["Start Perfetto profiling"]
%%   click node5 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:358:364"
%%   node5 --> node6["Profiling active"]
%%   click node6 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:368:368"
%% 
%%   subgraph loop1["While profiling is active"]
%%     node6 --> node7{"Was profiling interrupted?"}
%%     click node7 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:373:376"
%%     node7 -->|"No"| node6
%%     node7 -->|"Yes"| node8["Send interrupt signal"]
%%     click node8 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:381:381"
%%   end
%% 
%%   node8 --> node9["Restore signal handler"]
%%   click node9 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:384:384"
%% 
%%   subgraph loop2["Wait for profiling to finish"]
%%     node9 --> node10["Check if profiling process is still running"]
%%     click node10 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:386:389"
%%     node10 -->|"Still running"| node10
%%     node10 -->|"Finished"| node11["Retrieve trace results"]
%%     click node11 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:391:392"
%%   end
%% 
%%   node11 --> node12["Clean up trace and config files"]
%%   click node12 openCode "<SwmPath>[python/tools/cpu_profile.py](python/tools/cpu_profile.py)</SwmPath>:393:394"
%%   node12 --> node13["Done"]
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/python/tools/cpu_profile.py" line="329">

---

In <SwmToken path="python/tools/cpu_profile.py" pos="329:2:2" line-data="def record_trace(config, profile_target):">`record_trace`</SwmToken>, we check device compatibility, push the config to a unique path, start Perfetto profiling, and monitor the process. We use UUIDs for file paths to keep sessions separate, and set up for handling user interrupts.

```python
def record_trace(config, profile_target):
  """Runs Perfetto with the provided configuration to record a trace.

  Args:
    config: The Perfetto config to be used for tracing/profiling.
    profile_target: The directory where the recorded trace is output.
  """
  NULL = open(os.devnull)
  NO_OUT = {
      'stdout': NULL,
      'stderr': NULL,
  }
  if not release_or_newer('T'):
    sys.exit("This tool requires Android T+ to run.")

  # Push configuration to the device.
  # On Windows, temp files cannot be accessed by external processes while open
  # due to file locking, so we must close before adb push and manually cleanup.
  tf = tempfile.NamedTemporaryFile(delete=False)
  try:
    tf.write(config.encode('utf-8'))
    tf.flush()
    tf.close()
    profile_config_path = '/data/misc/perfetto-configs/config-' + UUID
    adb_check_output(['adb', 'push', tf.name, profile_config_path])
  finally:
    os.remove(tf.name)

  profile_device_path = '/data/misc/perfetto-traces/profile-' + UUID
  perfetto_command = ('perfetto --txt -c {} -o {} -d')
  try:
    perfetto_pid = int(
        adb_check_output([
            'adb', 'exec-out',
            perfetto_command.format(profile_config_path, profile_device_path)
        ]).strip())
  except ValueError as error:
    sys.exit("Unable to start profiling: {}".format(error))

  print("Profiling active. Press Ctrl+C to terminate.")

  old_handler = signal.signal(signal.SIGINT, sigint_handler)

  perfetto_alive = True
  while perfetto_alive and not IS_INTERRUPTED:
    perfetto_alive = subprocess.call(
        ['adb', 'shell', '[ -d /proc/{} ]'.format(perfetto_pid)], **NO_OUT) == 0
    time.sleep(0.25)
```

---

</SwmSnippet>

<SwmSnippet path="/python/tools/cpu_profile.py" line="376">

---

Here we handle user interrupts by sending SIGINT to the profiling process on the device, then restore the signal handler and wait for the process to exit cleanly.

```python
    time.sleep(0.25)

  print("Finishing profiling and symbolization...")

  if IS_INTERRUPTED:
    adb_check_output(['adb', 'shell', 'kill', '-INT', str(perfetto_pid)])

  # Restore old handler.
  signal.signal(signal.SIGINT, old_handler)

  while perfetto_alive:
    perfetto_alive = subprocess.call(
        ['adb', 'shell', '[ -d /proc/{} ]'.format(perfetto_pid)]) == 0
    time.sleep(0.25)
```

---

</SwmSnippet>

<SwmSnippet path="/python/tools/cpu_profile.py" line="389">

---

Finally we pull the trace file from the device to the host, then clean up the config and trace files on the device.

```python
    time.sleep(0.25)

  profile_host_path = os.path.join(profile_target, 'raw-trace')
  adb_check_output(['adb', 'pull', profile_device_path, profile_host_path])
  adb_check_output(['adb', 'shell', 'rm', profile_config_path])
  adb_check_output(['adb', 'shell', 'rm', profile_device_path])
```

---

</SwmSnippet>

## Post-processing and exporting profiling results

<SwmSnippet path="/python/tools/cpu_profile.py" line="514">

---

Back in <SwmToken path="python/tools/cpu_profile.py" pos="506:2:2" line-data="def main(argv):">`main`</SwmToken>, after recording the trace, we symbolize it, generate profiles, and copy them to the destination. This wraps up the profiling flow and prepares results for analysis.

```python
  traceconv = get_traceconv()
  trace_file = symbolize_trace(traceconv, profile_target)
  copy_profiles_to_destination(
      profile_target, generate_pprof_profiles(traceconv, trace_file, args))
  return 0
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
