# Learning Log — 14 November 2026 Linux Text Processing: AWK & SED 🐧

## 🎯 Today's Learning Objective

Understand how to process, extract, filter, analyse, and transform text directly from the Linux command line using two powerful tools: AWK and SED.

In Linux administration and DevOps, we frequently work with server logs, configuration files, command output, and structured text data.

Manually inspecting all this information is inefficient. Today, I explored how AWK and SED help automate text-processing tasks.

## 📚 Topics Explored

### 1. Introduction to Text Processing
- Why text processing matters in Linux.
- Common use cases involving server logs, configuration files, and command output.
- Understanding how AWK and SED complement tools such as `grep`, `sort`, and `uniq`.

### 2. AWK — Text Processing and Data Analysis
- What AWK is and where it is useful.
- Understanding AWK's records and fields.
- Using `$0`, `$1`, `$2`, `$NF`, `NR`, and `NF`.
- Printing selected columns and formatting output.
- Filtering records using patterns and conditions.
- Searching for keywords and matching regular expressions.
- Changing field separators using `-F`.
- String functions such as `gsub()`, `length()`, `index()`, `toupper()`, and `tolower()`.
- Counting records and performing calculations.
- Understanding `BEGIN` and `END` blocks.
- Using variables, arrays, conditions, and loops.
- Writing reusable AWK scripts using `-f`.

### 3. SED — Stream Editor
- What SED is and how it processes text.
- Understanding the Pattern Space.
- Searching for matching lines.
- Printing selected lines and line ranges.
- Substitution using `s/old/new/`.
- Understanding the global (`g`) flag.
- Editing files in place using `-i`.
- Deleting lines using `d`.
- Inserting and appending text.
- Using regular expressions and address ranges.
- Running multiple expressions with `-e`.
- Storing SED commands in separate files.

### 4. Combining AWK and SED
- Understanding the difference between line-oriented editing and field-oriented processing.
- Combining text-processing tools through pipelines.
- Using `sort`, `uniq`, and `head` to aggregate results.
- Extracting information from logs and command output.

## 💡 My Biggest Takeaway

Today, I understood how much can be accomplished directly from the Linux terminal without manually opening files or writing a separate program.

AWK makes it possible to extract fields, filter records, perform calculations, and generate useful summaries.

SED makes it possible to transform text streams by replacing, deleting, and manipulating lines.

Together, they make command-line text processing extremely powerful.

## 🎯 Connection to DevOps

These tools are particularly useful when working with:

- Nginx and Apache access logs.
- Application logs and error reports.
- Linux system command output.
- Configuration files.
- Deployment and maintenance scripts.
- Basic monitoring and reporting tasks.

For example, extracting frequent client IP addresses from web server logs or counting HTTP status codes can be done with a small command-line pipeline.

## 🔭 What I Want to Explore Next

- More advanced AWK patterns and associative arrays.
- AWK functions and formatted output.
- SED address ranges and Hold Space.
- Regular expressions in greater depth.
- Practical log-analysis exercises.
- Combining Linux text-processing tools in Bash scripts.

## 📖 Detailed Technical Notes

Today's exploration is documented in the following technical chapter:

➡️ [Linux Text Processing](../../notes/linux/09-shell-scripting/README.md)


## 📝 Learning Philosophy

I'm not trying to memorize every command or option.

I'm focusing on understanding the underlying concepts, experimenting with real examples, and learning when a particular tool is appropriate.