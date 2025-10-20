# Optimizing Resource Usage and Cost Efficiency for Jenkins and Google Cloud Storage

```{abstract}
This technical note presents a case study of migrating a Jenkins system from a hybrid infrastructure to Google Kubernetes Engine (GKE) and the subsequent optimization efforts to reduce costs while maintaining performance. The initial migration to GKE, along with the addition of an aarch64 distribution support, resulted in a significant increase in monthly expenses. Through a series of strategic changes to the infrastructure, including the adoption of autoscaling pods, the utilization of more cost-effective machine types, and the separation of workloads into dedicated node pools, the monthly costs were reduced by approximately 75% while preserving the system's requirements and improving overall performance. The note also outlines our future plans to optimize resource utilization and reduce costs while ensuring efficient CI/CD pipelines and reliable storage solutions.
```

## Introduction

Our Jenkins pipelines are critical components to our software development. As the complexity and scale of these pipelines grow, it becomes increasingly important to optimize the underlying infrastructure for cost-efficiency and performance. This technical note explores the challenges faced during the migration of our Jenkins system to GKE and the subsequent optimization efforts to reduce costs while maintaining or enhancing performance.

## Initial Migration and Challenges:

The Jenkins system was initially running in a hybrid location, with managers on SLAC infrastructure and workers on GKE. The GKE cluster consisted of a node pool with 6 nodes, each running on an n2-highmem-32 machine type. This migration led to a significant increase in monthly expenses, ranging from $9,000 to $12,000. To support aarch64 distribution, an additional node pool with 6 nodes running on T2a-highmem-32 machine type was added, further increasing the monthly costs to $18,000 to $20,000. During this period, the entire system was moved onto GKE, eliminating the hybrid approach, but increasing our expenses.

## Optimization Strategies:

1. Machine Type Selection:
   To improve the performance per dollar ratio, the T2a machine type was replaced with the more powerful C4a-highmem-32 machine type. Although marginally cheaper, this change allowed for better resource utilization and faster task completion.

2. Autoscaling Pods:
   By migrating to autoscaling pods, the system was able to dynamically allocate resources based on the actual workload, eliminating the costs associated with idle compute. This change resulted in a monthly savings of approximately $4,000 to $5,000.

3. Dedicated Node Pools:
   A new node pool was created using the n2-standard-4 machine type for longer-running tasks, such as managers, which require fewer resources compared to other tasks. Additionally, the main x86 chip workhorse was switched from n2 to C4d-highmem-32, which, despite being slightly more expensive, offered higher performance. This allowed for faster task completion, ultimately leading to cost savings.

## Results and Trade-offs:

The implemented optimizations reduced the monthly costs to approximately $5,000, while maintaining the system's requirements and improving overall performance. However, these changes introduced a slightly longer total time to compute due to the increased time required for workers to start compared to having machines waiting for work. Additionally, a custom caching mechanism was developed, adding complexity and a minor increase in job start time. Despite these trade-offs, the optimized system achieved a 3-40 minute reduction in job execution time, depending on the specific job and caching.

## Future Improvements:

To further optimize the infrastructure, several improvements are being considered:

1. Workload Separation:
   Creating additional node pools to separate workloads based on resource requirements, allowing for better allocation and tracking of expenses.

2. Quick Small Work Node Pool:
   Implementing a dedicated node pool with faster chips for quick, small tasks, such as API calls to minimize idle time.

3. Pipeline Stack-OS-Matrix Isolation:
   Separating the pipeline stack-OS-matrix from other workloads to enable more accurate cost tracking and resource allocation.

4. Right-Sizing Node Pools:
   Adjusting the size of node pools based on the specific demands of each workload, such as using highmem-32 machine types exclusively for stack-OS-matrix or exploring smaller machine types for tasks that do not require high memory, like building afw.

5. Integrate better monitoring tools:
   Add better tools to track expenses and bottlenecks. Jenkins does not come with great tooling for tracking pipelines and job durations. Implementing a tool like Grafana can provide some much needed insights.
