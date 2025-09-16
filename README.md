### Introduction
This project analyzes GPU/CPU cluster traces to understand scheduling delays, runtime, placement and resource efficiency across latency-sensitive inference workloads. The workflow demonstrates an end-to-end approach, from data cleaning and exploratory analysis, to multivariate LSTM forecasting. Forecasts are intended to inform Scaling and guide efficient resource allocation, with the potential for further optimization strategies. Hhowever, the optimization component is not implemented in this work.

### Objective
- Analyze scheduling delay and runtime across node types, applications, and placement rules
- Evaluate resource allocation efficiency for CPU, GPU, memory, RDMA, and disk.
- Study the impact of placement constraints
- Forecast short-term future resource demand using historical instance data with a multivariate LSTM.

### Dataset Overview
- Source: 2025 Alibaba GPU-disaggregated cluster traces from Kaggle.
- Size: 23,871 rows (instances), 156 unique inference services.
- Workload type: 100% latency-sensitive inference jobs.
- Columns of interest:
  - Identifiers & Roles: instance_sn, role (CN = Compute Node, HN = Head Node), app_name (service/instance name)
  - Resource requests & limits: cpu_request/limit, gpu_request/limit, memory_request/limit, disk_request/limit, rdma_request/limit
  - Placement constraints: max_instance_per_node
  - Lifecycle timestamps: creation_time, scheduled_time, deletion_time

### Data Cleaning Notes
- Dropped rows missing creation_time or scheduled_time because these timestamps are essential to calculate scheduling delay. Without them, we cannot determine when a job entered the system or started executing..
- Rows missing deletion_time were kept and flagged as is_running = True. These represent jobs that were still active when the trace ended and are useful for scheduling analysis but cannot be used for runtime computation.
- Times converted to timedelta in seconds for analysis.
- Scheduling analysis uses all rows to capture queue delays while runtime analysis excludes running jobs since their total execution time is unknown..

 ### Exploratory Data Analysis
**1. Scheduling Delay & Runtime Analysis**
- Instances wait a short time in the queue before starting (~3 min 13 sec average).
- Execution dominates the lifecycle (~28 hr 46 min average runtime).
- Head Nodes experience higher scheduling delays; Compute Nodes have longer runtimes.
- Top applications vary: some apps have significantly longer delays or runtimes, highlighting resource-intensive workloads.

**2. Resource Allocation Efficiency**
- CPU, GPU, memory, and RDMA requests closely match limits hence efficient allocation, no major waste.
- Disk efficiency ~85%, indicating some capacity left unallocated.
- Only Head Nodes request GPUs while Compute Nodes show slight over-provisioning on disk.
   
**3. Placement Rules Impact**
- Strict limits such as 2–4 instances per node, increase scheduling delay over 5 min.
- Looser limits reduce delay (seconds).
- Runtime largely unaffected, rules mainly influence start time, not execution length

**High-Level Observation: Since all workloads are latency-sensitive, RDMA requests and max_instance_per_node reveal constraints affecting placement and queueing.**

### Forecasting Resource Demand
**Goal: Predict future CPU, GPU, memory, RDMA, and disk demand for short-term planning.**
**1. Approach**
- Aggregate resource requests over time (resampled every 5 minutes).
- Normalize features using MinMaxScaler.
- Generate sequences of past resource usage for multivariate LSTM input.
- Train LSTM to predict next 5-minute resource demand for all features.

**2. Observations**
- CPU, memory, and disk predictions moderately accurate (NRMSE ~2).
- GPU and RDMA predictions poor due to bursty, spiky behavior (NRMSE ~5–6).

**Model overfits: low training loss vs higher validation loss**

**3. Next steps for improvement**
- Add regularization (dropout, L2 penalties).
- Tune sequence length and hidden units.
- Explore advanced architectures (stacked LSTM, GRU(Gated Reccurent Unit), Transformers).
- Apply walk-forward validation and ensembling.

**4. Future Work: Optimization & Placement Modeling**
**Objective: Efficiently allocate forecasted resource demand across cluster nodes while respecting system constraints.**
- Constraints:
 - RDMA requirement: Some jobs need nodes with high-speed GPU-to-GPU networking.
 - Max instances per node: Limits how many jobs can co-exist on a single node.
 - Resource capacities: CPU, GPU, memory, and disk usage cannot exceed node limits.

This step would build on the forecasting results to simulate and optimize placement, helping the infrastructure team maximize utilization and reduce unallocated resources.





.


















.


  




  



       

































