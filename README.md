# ride-sharing-spark-streaming
Spark structured streaming example

## Overview
This project demonstrates how to process real-time taxi ride data using PySpark Streaming. The incoming data is streamed via a socket and processed in three different tasks:

1. **Task 1:** Parse and store raw streaming data in CSV format.
2. **Task 2:** Process the data in micro-batches and save each batch as a separate CSV file.
3. **Task 3:** Perform windowed aggregations to calculate total fare within 5-minute windows and save results as CSV files.

---

## Prerequisites
Before running the scripts, ensure you have the following installed:

- Python 3.x
- Apache Spark 3.x
- PySpark (`pip install pyspark`)
- Netcat (for testing socket streams)

---

## Running the Tasks

Run data generator code to stream data in some port then run below task codes.

### Task 1: Parse and Store Raw Streaming Data

#### **Code Explanation**
- Initializes a Spark Streaming session.
- Reads JSON data from a socket (`localhost:9998`).
- Parses the JSON and converts timestamps.
- Writes parsed data to CSV format in the `output/parsed_stream/` directory.

#### **Commands to Run**
1. Start a socket stream using Netcat:
   ```sh
   python data_generator.py
   ```
2. Run the PySpark script:
   ```sh
   python task1.py
   ```
3. Send sample JSON data via Netcat:
   ```json
   {"trip_id": "1", "driver_id": "D123", "distance_km": 5.2, "fare_amount": 15.0, "timestamp": "2025-04-01 10:15:00"}
   ```
4. Check the output CSV in `output/parsed_stream/`.

---

### Task 2: Process and Store Data in Micro-Batches

#### **Code Explanation**
- Reads and parses streaming JSON data.
- Applies a **watermark** (to handle late data).
- Uses `foreachBatch` to write each batch to a separate CSV file.

#### **Commands to Run**
1. Start Netcat:
   ```sh
   python data_generator.py
   ```
2. Run the PySpark script:
   ```sh
   python task2.py
   ```
3. Send JSON data (same as Task 1).
4. Check batch files in `output/aggregated/`.

---

### Task 3: Perform Windowed Aggregations

#### **Code Explanation**
- Reads and parses streaming data.
- Converts timestamps and applies a **watermark**.
- Groups data into **5-minute time windows** and sums `fare_amount`.
- Uses `foreachBatch` to store windowed aggregations in CSV format.

#### **Commands to Run**
1. Start Netcat:
   ```sh
   python data_generator.py
   ```
2. Run the PySpark script:
   ```sh
   python task3.py
   ```
3. Send JSON data (same as Task 1).
4. Check windowed aggregation files in `output/window/`.

---

## Sample Output

### **Task 1 Output (CSV file example)**
```
trip_id,driver_id,distance_km,fare_amount,timestamp
1,D123,5.2,15.0,2025-04-01 10:15:00
```

### **Task 2 Output (Aggregated batch CSV)**
```
trip_id,driver_id,distance_km,fare_amount,timestamp
1,D123,5.2,15.0,2025-04-01 10:15:00
```

### **Task 3 Output (Windowed Aggregation CSV)**
```
window_start,window_end,total_fare
2025-04-01 10:15:00,2025-04-01 10:20:00,25.5
```

---

