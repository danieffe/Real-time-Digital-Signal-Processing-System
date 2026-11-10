# Real-time-Digital-Signal-Filtering-System

## 🏗️ System Architecture

The system is organized into modular software components coordinated through periodic threads scheduled under the **Rate Monotonic (RM)** policy:

1. **`generator` (50 Hz)**: Generates samples of a sine wave affected by noise and writes them to the shared buffer `sig_noise`.
2. **Multiple Filters (`filter1`, `filter2`, `filter3` at 50 Hz)**: Concurrently process the noisy signal using distinct filtering strategies (e.g., Moving Average, 2nd-order Butterworth, etc.) and store the outputs in the protected `sig_filtered` structure.
3. **`peak_counter` (1 Hz)**: Periodically analyzes the latest samples of the active filter to count local peaks (local minima/maxima) and sends the metrics via a message queue.
4. **`store` (5 Hz)**: Retrieves data from the filter queues (`/print_q*`) and writes it to disk (`signal.txt`), while also logging peak statistics to the terminal.
5. **`conf` (Aperiodic)**: Dynamically configures which filter to monitor and record at runtime through dedicated queues (`/conf_store_q`, `/conf_peak_q`).
6. **`live_plot.py`**: An external Python script for real-time graphical visualization of the raw and filtered signals.

---

## ⚙️ Synchronization and Concurrency

* **Mutual Exclusion**: Access to shared data structures (`sig_filtered`) is strictly protected to prevent race conditions.
* **Priority Inversion Prevention**: Uses proper synchronization protocols (such as priority inheritance mechanisms) to comply with hard/soft real-time requirements.
* **Inter-Task Communication**: Implemented using dedicated POSIX / `rt-lib` message queues (`/print_q1`, `/print_q2`, `/print_q3`, `/peak_q`, etc.).

## 🚀 Requirements and Execution

### 1. System Prerequisites
* Linux operating system with support for POSIX threads and the `rt-lib` support library.
* `gcc` compiler and makefile support.
* Python 3 interpreter with the `matplotlib` library for live plotting.

To install Python dependencies on Ubuntu/Debian:
```bash
sudo apt update
sudo apt install python3 python3-pip
pip install matplotlib 
```    

### 2. Compilation and Execution
It is recommended to bind periodic tasks to the same CPU core (using `taskset` or CPU affinity settings) to preserve Rate Monotonic schedulability assumptions[cite: 12]:

```bash
# Example execution pinned to a single core (e.g., core 0)
taskset -c 0 ./filter
taskset -c 0 ./store
taskset -c 0 ./conf

# Run the Python visualization script
python3 live_plot.py
```
