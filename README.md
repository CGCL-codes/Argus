# Argus

Argus is a  novel  job scheduling scheme in RDMA-assisted big data processing system. Argus exploits the structure feature of stage dependency and prioritizes the stages whose completion can enable more schedulable stages. Comprehensive experiments using large-scale traces collected from real world show that compared to RDMA-Spark and Branch Scheduling (IWQoS'19), Argus reduces job completion time by 21% and 41%, respectively.

## Introduction

Since the emergence of big data applications, big data processing systems, such as Hadoop and Spark, have been widely used in both industry and academia. A job in the processing system is commonly represented as a direct acyclic graph (DAG), where a vertex denotes a computation stage
that executes user-defined function and an edge depicts the data transfers between two stages as well as the dependency between them. A stage that reads data from its dependent stages cannot start until all of them are finished. Typically, a computation stage consists of multiple small tasks that execute the same user-defined function. A task is the basic execution unit to perform computation, while multiple tasks can run in parallel to improve resource efficiency. As different jobs with diverse characteristics (e.g., different DAG structures) commonly coexist in the big data processing system, efficient task scheduling is vital for overall system performance.

Traditional scheduling schemes in big data processing systems often give priority to data locality during job scheduling because the network transferring can dominate the job execution time. For a scheduled task, if its input data resides on the same machine with it, we say the task achieves data locality. Otherwise, it needs to read input data across network. Besides, shuffle between stages often involve all to all transfer, where every downstream task needs to read input data from every upstream task. The efficiency of such a communication intensive operation is subject to the network capability. Therefore, existing efficient task scheduling schemes commonly follow the network-optimized design principle. Existing designs optimize data locality during task placement, reduce shuffle traffics by deploying tasks to fewer and closer racks, or balance traffics among cross-rack links.

However, today’s modern datacenters are commonly equipped with high performance networks such as Remote Direct Memory Access (RDMA). With the equipment of RDMA, network is no longer a bottleneck of big data processing systems and the traditional network optimized
scheduling strategies become unsatisfied.

To exploit RDMA networks, Lu et al. recently proposed RDMA-Spark, which integrates RDMA with Apache Spark to accelerate big data processing. However, RDMA-Spark does not elaborate job scheduling. We examine the system efficiency of RDMA-Spark using experiments with real world. The result shows a significant fraction of 30% of slots are wasted during job execution. Such serious under-utilization of CPU slots indicates new opportunities for improving the system performance for RDMA-assisted big data processing systems.

According to our analysis, we verify that the root cause of computing resource under-utilization in RDMA-assisted big data processing system is the lack of schedulable tasks while the system has available slots. Based on the observation, we propose Argus, a novel job scheduling scheme in RDMA-assisted big data processing system. Argus investigates stage dependencies, and proposes a stage dependency aware scheduling scheme, which maintains a long-term vision and prioritizes stages whose completion can enable more stages to execute in the subsequent scheduling. We implement Argus on top of RDMA-Spark, and conduct comprehensive experiments to evaluate its performance using real world cluster traces. Results show that Argus reduces job completion time and job makespan compared to existing schemes.

## Architecture of Argus 

![Architecture of Argue](https://github.com/TelescopeScheduler/img-folder/blob/master/architecture.jpg)

Argus contains two main processing module: influence assessment module and influence-aware scheduling module. When a job is submitted, the influence assessment module gets DAG structure of the job. Then it investigates the stage dependencies and assesses influence for every stage. It also maintains a parallel stage queue to store parallel stages waiting for scheduling. After pre-processed by the influence assessment module, the influence-aware scheduling module schedules stages according to their influences. Specifically,
when some slots become available, the influence-aware scheduling module always firstly schedules tasks from the stage with the maximum influence. As the job runs, the influence assessment module also collects runtime states of tasks and dynamically adjusts the scheduling scheme. We also extend our design to further utilize available job profiles, e.g., stage duration, when the profile can be known apriori.

# How to use?

We implement Argus on top of [RDMA-Spark](http://hibd.cse.ohio-state.edu/#spark) and [Apache Spark](http://spark.apache.org/) (verson 2.1.0). As we can not get the source code of RDMA-Spark, we implement our scheduling strategy atop Apache Spark and replace the corresponding jar files of RDMA-Spark with our jars, e.g. spark-core_2.11-2.1.0.jar. 

## Building Argus

Download the source code of [Spark 2.1.0](https://github.com/apache/spark/tree/branch-2.1). And download our implementation code from the path core/. Replace corresponding source file of Spark with our code. Then, building the core module with the following command:
```
./build/mvn -Pyarn -Phadoop-2.7 -Dhadoop.version=2.7.3 -Phive -Phive-thriftserver -DskipTests clean package -pl core
```

For ease of use, one can also download the compiled jar file spark-core_2.11-2.1.0.jar from core/target/spark-core_2.11-2.1.0.jar.

## Using Argus
After building the core module, one can get spark-core_2.11-2.1.0.jar in terget file. Replace the same file in RDMA-Spark/jars/ with this file, and one can deploy the system refer to the RDMA-Spark document from [RDMA-Spark Userguide](http://hibd.cse.ohio-state.edu/static/media/rdma-spark/rdma-spark-0.9.5-userguide.pdf).

## Authors and Copyright
Argus is developed in National Engineering Research Center for Big Data Technology and System, Cluster and Grid Computing Lab, Services Computing Technology and System Lab, School of Computer Science and Technology, Huazhong University of Science and Technology, Wuhan, China by Sijie Wu (wsj@hust.edu.cn), Hanhua Chen (chen@hust.edu.cn), Yonghui Wang (yhw@hust.edu.cn), Hai Jin (hjin@hust.edu.cn).

Copyright (C) 2020, [STCS & CGCL](http://grid.hust.edu.cn/) and [Huazhong University of Science and Technology](https://www.hust.edu.cn/).
