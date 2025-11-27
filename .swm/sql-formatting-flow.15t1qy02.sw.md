---
title: SQL Formatting Flow
---
This document explains how users can check and enforce consistent SQL formatting. Users may provide SQL files, directories, or direct SQL input. Depending on the selected option, the system will verify formatting compliance, update files to match formatting rules, or format ad-hoc SQL content and print the result.

```mermaid
flowchart TD
  node1["Parsing arguments and dispatching actions"]:::HeadingStyle
  click node1 goToHeading "Parsing arguments and dispatching actions"
  node1 -->|"Check formatting"| node2["Verifying SQL file formatting"]:::HeadingStyle
  click node2 goToHeading "Verifying SQL file formatting"
  node2 -->|"Show result"| node4["Handling formatting and output"]:::HeadingStyle
  click node4 goToHeading "Handling formatting and output"
  node1 -->|"Format files or stdin"| node3["Formatting SQL content"]:::HeadingStyle
  click node3 goToHeading "Formatting SQL content"
  node3 -->|"Show or write result"| node4
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Parsing arguments and dispatching actions

<SwmSnippet path="/python/tools/format_sql.py" line="648">

---

In <SwmToken path="python/tools/format_sql.py" pos="648:2:2" line-data="def main() -&gt; None:">`main`</SwmToken>, we parse CLI arguments to figure out what the user wants (check formatting, format in place, or read from stdin). If <SwmToken path="python/tools/format_sql.py" pos="664:3:5" line-data="      &#39;--check-only&#39;,">`check-only`</SwmToken> is set, we call <SwmToken path="python/tools/format_sql.py" pos="674:5:5" line-data="    properly_formatted = check_sql_formatting(args.paths, args.indent_width,">`check_sql_formatting`</SwmToken> to verify if the SQL files match the formatting rules. This lets us decide early if we need to exit or proceed to formatting.

```python
def main() -> None:
  """Main entry point."""
  parser = argparse.ArgumentParser(
      description='Format SQL queries with consistent style')
  parser.add_argument(
      'paths',
      nargs='*',
      help='Paths to SQL files or directories containing SQL files')
  parser.add_argument(
      '--indent-width',
      type=int,
      default=2,
      help='Number of spaces for indentation (default: 2)')
  parser.add_argument(
      '--in-place', action='store_true', help='Format files in place')
  parser.add_argument(
      '--check-only',
      action='store_true',
      help='Check for formatting violations without making changes')
  parser.add_argument(
      '--verbose',
      action='store_true',
      help='Print status messages during execution')
  args = parser.parse_args()

  if args.check_only:
    properly_formatted = check_sql_formatting(args.paths, args.indent_width,
                                              args.verbose)
```

---

</SwmSnippet>

## Verifying SQL file formatting

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start SQL formatting check"] --> node2["For each path in input"]
    click node1 openCode "python/tools/format_sql.py:616:629"
    subgraph loop1["For each path"]
        node2 --> node3{"Is path a directory?"}
        click node3 openCode "python/tools/format_sql.py:630:632"
        node3 -->|"Yes"| node4["Collect all .sql files in directory"]
        click node4 openCode "python/tools/format_sql.py:633:634"
        node3 -->|"No"| node5["Use path as single SQL file"]
        click node5 openCode "python/tools/format_sql.py:635:635"
        node4 --> node6["For each SQL file"]
        node5 --> node6
        subgraph loop2["For each SQL file"]
            node6 --> node7["Check formatting with indent width"]
            click node7 openCode "python/tools/format_sql.py:638:640"
            node7 --> node8{"Is formatting correct?"}
            click node8 openCode "python/tools/format_sql.py:641:641"
            node8 -->|"No"| node9["Report violation (print to stderr if verbose)"]
            click node9 openCode "python/tools/format_sql.py:642:643"
            node8 -->|"Yes"| node6
            node9 --> node6
        end
        node6 --> node2
    end
    node2 --> node11{"Were all files formatted correctly?"}
    click node11 openCode "python/tools/format_sql.py:629:645"
    node11 -->|"Yes"| node12["Return True"]
    click node12 openCode "python/tools/format_sql.py:645:645"
    node11 -->|"No"| node13["Return False"]
    click node13 openCode "python/tools/format_sql.py:645:645"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start SQL formatting check"] --> node2["For each path in input"]
%%     click node1 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:616:629"
%%     subgraph loop1["For each path"]
%%         node2 --> node3{"Is path a directory?"}
%%         click node3 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:630:632"
%%         node3 -->|"Yes"| node4["Collect all .sql files in directory"]
%%         click node4 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:633:634"
%%         node3 -->|"No"| node5["Use path as single SQL file"]
%%         click node5 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:635:635"
%%         node4 --> node6["For each SQL file"]
%%         node5 --> node6
%%         subgraph loop2["For each SQL file"]
%%             node6 --> node7["Check formatting with indent width"]
%%             click node7 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:638:640"
%%             node7 --> node8{"Is formatting correct?"}
%%             click node8 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:641:641"
%%             node8 -->|"No"| node9["Report violation (print to stderr if verbose)"]
%%             click node9 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:642:643"
%%             node8 -->|"Yes"| node6
%%             node9 --> node6
%%         end
%%         node6 --> node2
%%     end
%%     node2 --> node11{"Were all files formatted correctly?"}
%%     click node11 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:629:645"
%%     node11 -->|"Yes"| node12["Return True"]
%%     click node12 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:645:645"
%%     node11 -->|"No"| node13["Return False"]
%%     click node13 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:645:645"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/python/tools/format_sql.py" line="616">

---

<SwmToken path="python/tools/format_sql.py" pos="616:2:2" line-data="def check_sql_formatting(paths: List[Union[str, Path]],">`check_sql_formatting`</SwmToken> loops through all provided files and directories, reads each SQL file, and uses <SwmToken path="python/tools/format_sql.py" pos="640:5:5" line-data="      formatted = format_sql(sql_file, sql, indent_width, verbose)">`format_sql`</SwmToken> to generate the expected formatting. If the formatted output differs from the original, it flags the file as needing changes. This is how we detect formatting violations.

```python
def check_sql_formatting(paths: List[Union[str, Path]],
                         indent_width: int = 2,
                         verbose: bool = False) -> bool:
  """Check SQL files for formatting violations without making changes.

  Args:
      paths: List of file or directory paths to check
      indent_width: Number of spaces for indentation
      verbose: Whether to print status messages

  Returns:
      True if all files are properly formatted, False otherwise
  """
  all_formatted = True
  for path in paths:
    path = Path(path)
    if path.is_dir():
      sql_files = path.rglob('*.sql')
    else:
      sql_files = [path]

    for sql_file in sql_files:
      with open(sql_file) as f:
        sql = f.read()
      formatted = format_sql(sql_file, sql, indent_width, verbose)
      if formatted != sql:
        print(f"Would format {sql_file}", file=sys.stderr)
        all_formatted = False

  return all_formatted
```

---

</SwmSnippet>

## Formatting SQL content

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1{"Does SQL contain '-- sqlformat file off'?"}
  click node1 openCode "python/tools/format_sql.py:557:560"
  node1 -->|"Yes"| node2{"Is verbose enabled?"}
  click node2 openCode "python/tools/format_sql.py:558:559"
  node2 -->|"Yes"| node3["Print status message"]
  click node3 openCode "python/tools/format_sql.py:559:559"
  node3 --> node4["Return SQL unchanged"]
  click node4 openCode "python/tools/format_sql.py:560:560"
  node2 -->|"No"| node4
  node1 -->|"No"| node5["Extract comment blocks"]
  click node5 openCode "python/tools/format_sql.py:563:563"
  node5 --> node6["Process macros"]
  click node6 openCode "python/tools/format_sql.py:566:566"
  subgraph loop1["For each SQL statement"]
    node7["Format SQL statement with indent width"]
    click node7 openCode "python/tools/format_sql.py:569:577"
  end
  node6 --> node7
  node7 --> node8["Restore macros"]
  click node8 openCode "python/tools/format_sql.py:580:580"
  node8 --> node9["Restore comment blocks"]
  click node9 openCode "python/tools/format_sql.py:581:581"
  node9 --> node10["Return formatted SQL"]
  click node10 openCode "python/tools/format_sql.py:581:581"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1{"Does SQL contain '-- sqlformat file off'?"}
%%   click node1 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:557:560"
%%   node1 -->|"Yes"| node2{"Is verbose enabled?"}
%%   click node2 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:558:559"
%%   node2 -->|"Yes"| node3["Print status message"]
%%   click node3 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:559:559"
%%   node3 --> node4["Return SQL unchanged"]
%%   click node4 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:560:560"
%%   node2 -->|"No"| node4
%%   node1 -->|"No"| node5["Extract comment blocks"]
%%   click node5 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:563:563"
%%   node5 --> node6["Process macros"]
%%   click node6 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:566:566"
%%   subgraph loop1["For each SQL statement"]
%%     node7["Format SQL statement with indent width"]
%%     click node7 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:569:577"
%%   end
%%   node6 --> node7
%%   node7 --> node8["Restore macros"]
%%   click node8 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:580:580"
%%   node8 --> node9["Restore comment blocks"]
%%   click node9 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:581:581"
%%   node9 --> node10["Return formatted SQL"]
%%   click node10 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:581:581"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/python/tools/format_sql.py" line="539">

---

In <SwmToken path="python/tools/format_sql.py" pos="539:2:2" line-data="def format_sql(file: Path,">`format_sql`</SwmToken>, we first check for a special comment to skip formatting if needed. Then we extract comments and preprocess macros to avoid messing with them during formatting. We parse the SQL using the 'Perfetto' dialect, format each statement, and later restore macros and comments. The next step is parsing the SQL ASTs, which is handled by the logic in <SwmPath>[python/…/sql_processing/docs_parse.py](python/generators/sql_processing/docs_parse.py)</SwmPath>.

```python
def format_sql(file: Path,
               sql: str,
               indent_width: int = 2,
               verbose: bool = False) -> str:
  """Format SQL content with consistent style.

  Args:
    file: Path to the SQL file (for error reporting)
    sql: SQL content to format
    indent_width: Number of spaces for indentation
    verbose: Whether to print status messages

  Returns:
    Formatted SQL string

  Raises:
    Exception: If SQL parsing or formatting fails
  """
  if sql.find('-- sqlformat file off') != -1:
    if verbose:
      print(f"Ignoring {file}", file=sys.stderr)
    return sql

  # First extract comment blocks
  sql_with_placeholders, comment_blocks = extract_comment_blocks(sql)

  # Then process macros
  processed, macros = preprocess_macros(sql_with_placeholders)
  try:
    formatted = ''
    for ast in sqlglot.parse(sql=processed, dialect=Perfetto):
      formatted += ast.sql(
          pretty=True,
          dialect=Perfetto,
          indent=indent_width,
          normalize=True,
          normalize_functions='lower',
      )
      formatted += ";\n\n"

```

---

</SwmSnippet>

<SwmSnippet path="/python/generators/sql_processing/docs_parse.py" line="243">

---

<SwmToken path="python/generators/sql_processing/docs_parse.py" pos="243:3:3" line-data="  def parse(self, doc: DocsExtractor.Extract) -&gt; Optional[TableOrView]:">`parse`</SwmToken> enforces repo-specific rules: it blocks 'CREATE OR REPLACE' in stdlib modules, restricts table creation to 'CREATE PERFETTO' unless allowlisted, and skips internal tables unless explicitly included. It unpacks the input assuming a fixed structure, parses columns, and builds a <SwmToken path="python/generators/sql_processing/docs_parse.py" pos="243:20:20" line-data="  def parse(self, doc: DocsExtractor.Extract) -&gt; Optional[TableOrView]:">`TableOrView`</SwmToken> object for downstream use.

```python
  def parse(self, doc: DocsExtractor.Extract) -> Optional[TableOrView]:
    assert doc.obj_kind == ObjKind.table_view

    or_replace, perfetto_or_virtual, type, self.name, schema = doc.obj_match

    # Skip internal artifacts early if not including them
    if is_internal(self.name) and not self.options.include_internal:
      return None

    if or_replace is not None:
      self._error(
          f'{type} "{self.name}": CREATE OR REPLACE is not allowed in stdlib '
          f'as standard library modules can only included once. Please just '
          f'use CREATE instead.')
      return

    if (type.lower() == "table" and not perfetto_or_virtual and
        self.name not in CREATE_TABLE_ALLOWLIST):
      self._error(
          f'{type} "{self.name}": Can only expose CREATE PERFETTO tables')
      return

    cols = self._parse_columns(schema, ObjKind.table_view)
    return TableOrView(
        name=self._parse_name(),
        type=type,
        desc=self._parse_desc_not_empty(doc.description),
        cols=cols)
```

---

</SwmSnippet>

<SwmSnippet path="/python/tools/format_sql.py" line="579">

---

Back in <SwmToken path="python/tools/format_sql.py" pos="539:2:2" line-data="def format_sql(file: Path,">`format_sql`</SwmToken>, after parsing and formatting, we restore macros and comment blocks to the SQL output. This keeps non-SQL content unchanged. If anything fails, we print an error and raise the exception.

```python
    # Restore macros first, then comment blocks
    with_macros = postprocess_macros(formatted, macros)
    return restore_comment_blocks(with_macros, comment_blocks).rstrip() + '\n'
  except Exception as e:
    print(f"Failed to format SQL: file {file}, {e}", file=sys.stderr)
    raise e
```

---

</SwmSnippet>

## Handling formatting and output

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start main function"] --> node2{"Should check formatting?"}
    click node1 openCode "python/tools/format_sql.py:676:677"
    node2 -->|"Yes"| node3["Exit with status: formatted or not"]
    click node2 openCode "python/tools/format_sql.py:676:677"
    click node3 openCode "python/tools/format_sql.py:676:677"
    node2 -->|"No"| node4{"Format files in place?"}
    click node4 openCode "python/tools/format_sql.py:677:678"
    node4 -->|"Yes"| node5["Format files in place"]
    click node5 openCode "python/tools/format_sql.py:678:678"
    node4 -->|"No"| node6{"Input from stdin?"}
    click node6 openCode "python/tools/format_sql.py:680:681"
    node6 -->|"Yes"| node7["Format SQL from stdin"]
    click node7 openCode "python/tools/format_sql.py:682:684"
    node6 -->|"No"| node8["Show help"]
    click node8 openCode "python/tools/format_sql.py:687:687"

    subgraph loop1["For each file or directory path"]
      node5 --> node9["Format each SQL file (indent width, verbose)"]
      click node9 openCode "python/tools/format_sql.py:587:613"
    end
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start main function"] --> node2{"Should check formatting?"}
%%     click node1 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:676:677"
%%     node2 -->|"Yes"| node3["Exit with status: formatted or not"]
%%     click node2 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:676:677"
%%     click node3 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:676:677"
%%     node2 -->|"No"| node4{"Format files in place?"}
%%     click node4 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:677:678"
%%     node4 -->|"Yes"| node5["Format files in place"]
%%     click node5 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:678:678"
%%     node4 -->|"No"| node6{"Input from stdin?"}
%%     click node6 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:680:681"
%%     node6 -->|"Yes"| node7["Format SQL from stdin"]
%%     click node7 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:682:684"
%%     node6 -->|"No"| node8["Show help"]
%%     click node8 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:687:687"
%% 
%%     subgraph loop1["For each file or directory path"]
%%       node5 --> node9["Format each SQL file (indent width, verbose)"]
%%       click node9 openCode "<SwmPath>[python/tools/format_sql.py](python/tools/format_sql.py)</SwmPath>:587:613"
%%     end
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/python/tools/format_sql.py" line="676">

---

Back in <SwmToken path="python/tools/format_sql.py" pos="648:2:2" line-data="def main() -&gt; None:">`main`</SwmToken>, after checking formatting, we either exit or move to <SwmToken path="python/tools/format_sql.py" pos="662:3:5" line-data="      &#39;--in-place&#39;, action=&#39;store_true&#39;, help=&#39;Format files in place&#39;)">`in-place`</SwmToken> formatting if requested. Calling <SwmToken path="python/tools/format_sql.py" pos="678:1:1" line-data="    format_files_in_place(args.paths, args.indent_width, args.verbose)">`format_files_in_place`</SwmToken> lets us update files directly to match the style.

```python
    sys.exit(0 if properly_formatted else 1)
  elif args.in_place:
    format_files_in_place(args.paths, args.indent_width, args.verbose)
  else:
```

---

</SwmSnippet>

<SwmSnippet path="/python/tools/format_sql.py" line="587">

---

<SwmToken path="python/tools/format_sql.py" pos="587:2:2" line-data="def format_files_in_place(paths: List[Union[str, Path]],">`format_files_in_place`</SwmToken> goes through each file or directory, finds all .sql files, reads their content, formats them using <SwmToken path="python/tools/format_sql.py" pos="609:5:5" line-data="      formatted = format_sql(sql_file, sql, indent_width, verbose)">`format_sql`</SwmToken>, and writes the result back. This updates all files to the expected style.

```python
def format_files_in_place(paths: List[Union[str, Path]],
                          indent_width: int = 2,
                          verbose: bool = False) -> None:
  """Format multiple SQL files in place.

  Args:
      paths: List of file or directory paths to format
      indent_width: Number of spaces for indentation
      verbose: Whether to print status messages
  """
  for path in paths:
    path = Path(path)
    if path.is_dir():
      # Process all .sql files in directory recursively
      sql_files = path.rglob('*.sql')
    else:
      # Single file
      sql_files = [path]

    for sql_file in sql_files:
      with open(sql_file) as f:
        sql = f.read()
      formatted = format_sql(sql_file, sql, indent_width, verbose)
      with open(sql_file, 'w') as f:
        f.write(formatted)
      if verbose:
        print(f"Formatted {sql_file}", file=sys.stderr)
```

---

</SwmSnippet>

<SwmSnippet path="/python/tools/format_sql.py" line="680">

---

Back in <SwmToken path="python/tools/format_sql.py" pos="648:2:2" line-data="def main() -&gt; None:">`main`</SwmToken>, if no files are given, we check for SQL input from stdin, format it using <SwmToken path="python/tools/format_sql.py" pos="683:5:5" line-data="      formatted_sql = format_sql(Path(&quot;stdin&quot;), sql_input, args.indent_width)">`format_sql`</SwmToken>, and print the result. If there's no input, we show the help message.

```python
    # Read from stdin if no files provided
    if not sys.stdin.isatty():
      sql_input = sys.stdin.read()
      formatted_sql = format_sql(Path("stdin"), sql_input, args.indent_width)
      print(formatted_sql)
    else:
      # Print help if no input provided
      parser.print_help()
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
