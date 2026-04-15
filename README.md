# 📦 MapReduceX

<!-- PROJECT BANNER -->
<p align="center">
  <img src="assets/banner.png" alt="MapReduceX Banner" width="100%">
</p>

<p align="center">
  <b>A fault-tolerant, distributed MapReduce framework built from scratch.</b><br>
  Designed for scalability, clarity, and real-world distributed systems learning.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/language-Python-blue">
  <img src="https://img.shields.io/badge/status-Active-success">
  <img src="https://img.shields.io/badge/license-MIT-green">
  <img src="https://img.shields.io/badge/build-passing-brightgreen">
</p>

---

## 📌 Overview

MapReduceX is a distributed data processing framework inspired by Google’s original MapReduce paper. It enables users to execute large-scale data transformations across multiple worker nodes with built-in fault tolerance, dynamic task scheduling, and parallel execution.

This project exists to make the internals of distributed systems transparent and understandable rather than hidden behind complex abstractions. Instead of only showing users how to run the framework, this README also explains what the system is doing internally and why those behaviors matter in distributed execution.

MapReduceX is designed to expose how scheduling, heartbeat monitoring, task reassignment, intermediate data handling, and reduce-stage coordination actually work. This makes it a useful educational framework for understanding distributed systems behavior rather than treating that behavior as a black box.

### Who this is for

- Computer science students learning distributed systems
- Engineers curious about how MapReduce works under the hood
- Anyone seeking a clean, readable reference implementation

---

## 📚 Table of Contents

- [🛠 Fresh Setup (From a Clean Clone)](#-fresh-setup-from-a-clean-clone)
- [🚀 Quickstart](#-quickstart)
- [✨ Features](#-features)
- [🏗 Architecture](#-architecture)
- [💡 Usage Examples](#-usage-examples)
- [🧪 Tests](#-tests)
- [❓ FAQ](#-faq)
- [📦 Dependencies](#-dependencies)
- [🤝 Contributing](#-contributing)
- [🙏 Acknowledgements](#-acknowledgements)

---

## 🛠 Fresh Setup (From a Clean Clone)

These steps describe how to set up MapReduceX starting from a fresh clone of the repository.

### 1️⃣ Clone the Repository

    git clone https://github.com/Naumanbo/485-p4-map-reduce/tree/main
    cd 485-p4-map-reduce

---

### 2️⃣ Create a Python Virtual Environment

Creating a virtual environment keeps project dependencies isolated from your system Python and helps ensure that the framework and test suite run consistently.

#### macOS / Linux

    python3 -m venv venv
    source venv/bin/activate

#### Windows (PowerShell)

    python -m venv venv
    venv\Scripts\Activate.ps1

After activation, your shell prompt should indicate that the virtual environment is active.

---

### 3️⃣ Install Required Dependencies

With the virtual environment activated, install dependencies from the requirements file:

    pip install -r requirements.txt

This installs all runtime and development dependencies needed to run the framework and tests.

---

### 4️⃣ Verify Installation (Optional)

You can verify that dependencies are installed correctly by running:

    python --version
    pip list

---

### 5️⃣ Deactivate the Environment (Optional)

When finished, you can exit the virtual environment:

    deactivate

---

## 🚀 Quickstart

Once your environment is set up, you can run the system locally and simulate a small distributed cluster.
### 1️⃣ Run the Coordinator
Start the coordinator first. It is responsible for tracking worker state, assigning tasks, and managing recovery when failures occur.

    python coordinator.py

---

### 2️⃣ Start Worker Nodes

Open additional terminals (with the virtual environment activated) and run:

    python worker.py

You can launch multiple workers to simulate a distributed cluster. Each worker registers with the coordinator, requests tasks when idle, and sends heartbeat messages so the coordinator knows it is still active.

### 3️⃣ Submit a Job

Once the coordinator and one or more workers are running, submit a job:

    python submit_job.py --input data/books --output results/wordcount


## What happens when you run the system?

At a high level, the workflow looks like this:

1. The coordinator starts and waits for workers to connect.
2. Workers register with the coordinator.
3. A job is submitted with an input path and output path.
4. The coordinator splits the job into map tasks.
5. Workers process map tasks in parallel.
6. Intermediate data is partitioned and later fetched by reducers.
7. Reduce tasks aggregate the data and write final output.

The GIF below is meant to reinforce that startup flow by showing coordinator startup, worker registration, and job submission in sequence.

![Installation and startup walkthrough showing environment setup, manager startup, and worker initialization](assets/gifs/starting_worker_and_manager.gif)


---

## ✨ Features

### 🔹 Distributed Task Scheduling
- Coordinator assigns map and reduce tasks dynamically
- Workers request tasks when idle
- Unfinished tasks are re-queued automatically

MapReduceX uses a coordinator-worker model in which workers do not statically own a specific portion of the job. Instead, they repeatedly request work as they become available, which naturally balances task distribution across workers.

### 🔹 Fault Tolerance
- Worker health monitored via heartbeats
- Crashed workers are detected and replaced
- No data loss during execution

The coordinator monitors workers using heartbeat messages. If a worker stops responding, unfinished work can be reassigned to another worker. This allows the framework to continue making progress even when a worker process fails.

### 🔹 Parallel Execution
- Map tasks execute concurrently across workers
- Reduce phase begins only after all map tasks complete

Parallel execution is most visible during the map stage, where multiple workers can operate on different input splits at the same time. The reduce phase starts only after all required map outputs are available.

### 🔹 Simple Developer Interface
- Users define custom map and reduce functions
- Framework manages execution, recovery, and coordination

Users only need to provide the map and reduce logic for their application. The framework handles the distributed execution behavior around those functions.

---

## 🏗 Architecture

MapReduce follows a Coordinator–Worker architecture commonly used in distributed systems.

### Components

Coordinator:
- Tracks task states
- Assigns work to workers
- Detects worker failures and reassigns tasks

The coordinator is the central control process. It does not perform the map or reduce computations itself. Instead, it monitors the system, distributes work, and ensures the job progresses correctly.

Workers:
- Execute map or reduce tasks
- Send periodic heartbeat messages
- Write intermediate results to disk

Workers perform the actual computation. During the map stage, they process input data and write intermediate outputs locally. During the reduce stage, they retrieve the intermediate data they need and produce final outputs.

### 📐 Architecture Diagram
The diagram below is best read from left to right.

* Input data is split into multiple map tasks
* Each map task emits intermediate key-value pairs
* Intermediate outputs are partitioned so the same keys are routed to the same reducer
* Reduce tasks merge grouped data and write final output

This diagram is useful because it shows both the map-stage parallelism and the flow of intermediate data into the reduce stage.

![Architecture Diagram](assets//architecture_diagram.png)

---

## Internal Execution Flow
MapReduceX follows the same general execution flow used in classic MapReduce systems:

1. Input data is divided into splits.
2. Each split becomes a map task.
3. Map workers apply the user-defined map function to their input.
4. Intermediate key-value pairs are partitioned by key.
5. Partitions are stored locally until reducers fetch them.
6. Reducers gather partitions from all map workers.
7. Reducers merge and sort the data by key.
8. The user-defined reduce function aggregates values for each key.
9. Final outputs are written to result files.

This flow matters because many performance and fault tolerance behaviors appear in the transitions between these stages. For example, worker failure during the map stage may require recomputation of intermediate output, while the shuffle and merge steps often dominate runtime in data-intensive jobs.

## 💡 Usage Examples

### Example: Word Count
Word count is the most common introductory MapReduce example because it makes the system behavior easy to see. Each map task emits a (word, 1) pair for every word it encounters, and each reduce task sums the values for a specific word.

    def map(key, value):
        for word in value.split():
            emit(word, 1)

    def reduce(key, values):
        emit(key, sum(values))

Submit the job:

    python submit_job.py --input data/books --output results/wordcount

Example output:

    distributed: 124
    systems: 98
    mapreduce: 76

---

## What happens internally during word count?
1. Input files are split across map tasks.
2. Each worker reads its assigned split.
3. The map function emits pairs such as (distributed, 1) and (systems, 1).
4. Intermediate outputs are partitioned so identical words go to the same reducer.
5. Reducers fetch the relevant partitions from all map workers.
6. Each reducer sums the counts for the words it owns.
7. Final totals are written to output files.
The GIF below is most useful when viewed alongside this explanation. It shows how job submission leads to active task assignment and distributed execution rather than simply repeating the command-line steps above.
## 🎥 GIF Walkthroughs

### ▶️ Submitting a Job

Job Submission Flow  
Shows:
- Starting the coordinator
- Workers connecting
- Job submission
- Tasks being assigned
![Job submission flow showing manager startup, worker registration, job submission, and task assignment](assets/gifs/submitting_workflow.gif)


---

### ▶️ Fault Tolerance Demo

Worker Failure Recovery  
Shows:
- Worker process crashing
- Coordinator detecting failure
- Task reassignment to another worker

![Fault tolerance demo showing worker crash, manager detecting failure, and task reassignment to another worker](assets/gifs/fault_tolerance.gif)


---

## 🧪 Tests

This project includes a test suite located in the tests directory.

### Running All Tests

    pytest tests/

---

### Running a Specific Test File

    pytest tests/test_coordinator.py

---

### Running Tests Verbosely

    pytest tests/ -v

---

### Notes
- Ensure no coordinator or worker processes are running before testing
- Some tests mock worker behavior for isolation

The tests cover important parts of the framework behavior, including coordinator logic and worker interaction. Running tests in isolation is especially important for a distributed systems project because stale processes can interfere with expected results.
---

## ❓ FAQ

**Why build this instead of using Hadoop or Spark?**  
This project is designed for education and clarity. Production frameworks prioritize performance and scale, while MapReduceX prioritizes transparency and learning.

**How many workers can I run?  **
As many as your system supports. Each worker runs as an independent process.

**Does this use real networking?  **
Yes. Workers and the coordinator communicate over sockets to simulate real distributed execution.

**Is this production-ready?  **
No. This is an educational framework meant for learning and experimentation.

**What does fault tolerance look like in practice?**
If a worker crashes or stops sending heartbeat messages, the coordinator can detect the failure and reassign incomplete work to another worker.

---

## 📦 Dependencies

- Python 3.10+
- socket
- threading
- multiprocessing
- json
- pytest (for testing)

Install all dependencies with:

    pip install -r requirements.txt

---

## 🤝 Contributing

Contributions are welcome and encouraged.

1. Fork the repository
2. Create a feature branch
3. Submit a pull request with a clear explanation

Please ensure code is readable and well-documented.

---

## 🙏 Acknowledgements

- Google Research – MapReduce: Simplified Data Processing on Large Clusters
- Univ
