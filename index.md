# Optimizing Resource Usage and Cost Efficiency for Jenkins and Google Cloud Storage

```{abstract}
This technical note presents a case study of migrating a Jenkins system from a hybrid infrastructure to Google Kubernetes Engine (GKE) and the subsequent optimization efforts to reduce costs while maintaining performance. Hosting Jenkins workers on GKE, along with the addition of an aarch64 distribution support, resulted in significant expenses of up to \$250,000 per year. Through a series of strategic changes to the infrastructure, including the adoption of autoscaling pods, the utilization of more cost-effective machine types, and the separation of workloads into dedicated node pools, the monthly costs were reduced by approximately 75% while preserving the system's requirements and improving overall performance. This note also outlines our future plans to optimize resource utilization and reduce costs while ensuring efficient CI/CD pipelines and reliable storage solutions.
```

## Introduction

Our Jenkins pipelines are critical components to our software development. As the complexity and scale of these pipelines grow, it becomes increasingly important to optimize the underlying infrastructure for cost-efficiency and performance. This technical note explains the challenges faced during the migration of our Jenkins system to Google Kubernetes Engine (GKE) and the subsequent optimization efforts to reduce costs while maintaining or enhancing performance.

## Initial Migration and Challenges

The Jenkins system was initially running in a hybrid configuration, with controllers on SLAC infrastructure and workers on GKE. The GKE cluster consisted of a node pool with six nodes using the n2-highmem-32 machine type. Hosting the Jenkins workers on GKE comes with significant expenses, ranging from \$9,000 to \$12,000 per month. To support aarch64 distribution, an additional node pool with 6 nodes running on t2a-highmem-32 machine type was added, further increasing the monthly costs to \$18,000 to \$20,000.

At this stage, the Jenkins controllers still operated at SLAC and communicated with agents at GKE via port tunneling and web-sockets. The connection between SLAC and GKE suffered from intermittent disconnections, particularly when high load caused network interruptions. These failures resulted in aborted builds, wasted compute time, and frequent manual re-runs, contributing to business costs, decreased productivity, and increased time debugging. To address these issues, both the production and development Jenkins controllers were migrated to the GKE cluster. This eliminated the hybrid design and allowed for complete in-cluster networking at GKE, removing the use for port tunneling or web-sockets. 

Direct in-cluster communication between the controller and agents improved build reliability by preventing session drops. Additionally, other supporting applications such as the secret manager Hashicorp Vault, and cert-manager for TLS certificate provisioning, were co-located on the same cluster. This reduced operational complexity and simplified the management of credentials and certificates for Jenkins. Consequently, we have seen a significant reduction in network errors due to the in-cluster networking. 

The following figure shows the aforementioned increases in monthly expenses. 

```{figure} /assets/chart.png
```

## Optimization Strategies

1. **Machine Type Selection:**
   To improve the performance per dollar ratio, the T2a machine type was replaced with the more powerful c4a-highmem-32 machine type. Although marginally cheaper, this change allowed for better resource utilization and faster task completion. Additionally, the linux x86 agents were switched from n2-highmem-32 to c4d-highmem-32, which, despite being slightly more expensive, allowed for faster job completion, ultimately leading to cost savings.
   
2. **Autoscaling Pods:**
   By migrating to autoscaling pods, the system was able to dynamically allocate resources based on the actual workload, eliminating the costs associated with idle compute. This change resulted in a monthly savings of approximately \$4,000 to \$5,000.

3. **Storage:**
   To support the auto-scaling model, the ephemeral storage on nodes was increased from 100GB to 300GB, allowing each node to run a single agent with its own temporary workspace. This allowed us to move away from Persistent Volume based storage for the linux and aarch64 agents (previously 1500GB for each of the twelve agents), simplifying storage management and cutting associated costs. Additionally, the Persistent Volume sizes for all manager and snowflake agents were reduced to 300GB. 

4. **Dedicated Node Pools:**
   A new node pool was created using the n2-standard-4 machine type for longer-running tasks, such as managers, Vault and cert-manager, which require fewer resources compared to other tasks. These were migrated from an n2-highmem-32 node pool. Snowflake agents, which run nightly and weekly clean builds and official releases, have their own node pool to ensure node availability for these critical jobs. 

5. **Custom Caching:**
   Due to the architecture of the auto-agents, builds started from clean environments. Custom caching was implemented and substantially reduced job execution times.

## Results and Trade-offs

The aforementioned implemented optimizations **reduced the monthly costs to approximately \$5,000 – from an initial \$18,000 - \$20,000 – while maintaining the system's requirements and improving overall performance**. These automated agents and the custom caching mechanism adds complexity and a minor increase in job start time of approximately 2-5 minutes. However, the total job execution time has been reduced by 3-40 minutes (depending on specific job and caching), due to the new optimized system. 

The following figure shows expense reduction per month due to these changes.

``` {figure} /assets/chart2.png
```

## Future Improvements

To further optimize the infrastructure, several improvements are being considered:

1. **Workload Separation:**
   Creating additional node pools to separate workloads based on resource requirements, allowing for better allocation and tracking of expenses.

2. **Quick Small Work Node Pool:**
   Implementing a dedicated node pool with faster chips for quick, small tasks, such as API calls to minimize idle time.

3. **Pipeline Stack-OS-Matrix Isolation:**
   Separating the pipeline stack-os-matrix from other workloads to enable more accurate cost tracking and resource allocation.

4. **Right-Sizing Node Pools:**
   Adjusting the size of node pools based on the specific demands of each workload, such as using highmem-32 machine types exclusively for stack-os-matrix or exploring smaller machine types for tasks that do not require high memory, like building the job *afw*.

5. **Right-Sizing Resource Requests:**
   Reviewing CPU and memory usage across the containers and pods to lower request values where possible and reduce over-allocation.

7. **Integrate better monitoring tools:**
   Add better tools to track expenses and bottlenecks. Jenkins does not come with great tooling for tracking pipelines and job durations. Implementing a tool like Grafana can provide some much needed insights.
