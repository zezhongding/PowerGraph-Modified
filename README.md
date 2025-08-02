# PowerGraph-Modified

PowerGraph is a graph computing system paper published in 2012 at the **OSDI** conference. Because it incorporates multiple traditional graph algorithms and supports distributed computing, it remains a common baseline for comparison in many research papers even today. Additionally, PowerGraph is often used to validate the effectiveness of various algorithms.

However, due to the lack of maintenance over the years, PowerGraph can no longer be directly compiled from its GitHub source code. The main reason is that the URLs of several third-party libraries have become invalid. In our recent research, we needed to use the PowerGraph framework and encountered many issues during the process. Here, we document our experience in the hope that it will help others who need to use PowerGraph for testing in the future.

**Stable and Fixed Version**:[GitHub - BearBiscuit05/PowerGraph_update](https://link.zhihu.com/?target=https%3A//github.com/BearBiscuit05/PowerGraph_update)

**Original Version**:[https://github.com/jegonzal/PowerGraph](https://link.zhihu.com/?target=https%3A//github.com/jegonzal/PowerGraph)

## Environment Requirements

- Ubuntu16.04
- gcc 5.4
- g++5.4
- jdk1.8
- build-essential
- Zlib

## Configuration Process

```bash
sudo apt-get update

sudo apt-get install gcc g++ build-essential libopenmpi-dev openmpi-bin default-jdk cmake zlib1g-dev git

git clone https://github.com/BearBiscuit05/PowerGraph_update.git

cd PowerGraph_update

./configure

cd release/toolkits/graph_analytics

make -j2
```

## Or pull the image directly from Docker Hub

```bash
docker pull bearbiscuit/pgimage:latest
```


## Scenario 1: Testing the Designed Graph Partitioning Algorithm

### Custom Partition Result Import

PowerGraph provides a convenient framework for executing traditional graph computing tasks and is frequently used as a baseline in research papers, particularly for evaluating the robustness of partitioning algorithms. To facilitate testing the performance of various partitioning algorithms in real distributed environments using PowerGraph, we have modified its codebase to allow PowerGraph to directly accept external partitioning results, making it easier for researchers to conduct such experiments.

### Format of the Partition File

Take a `graph.txt` file as an example. Each line contains three parameters: `src`, `dst`, and `partid`, where `partid` indicates which partition the corresponding edge (`src`, `dst`) is finally assigned to. The three parameters should be separated by tabs (`\t`).

```text
#SRC\tDST\tpartid
1	2	0
2	3	1
3	0	1
```

### Parameters for Running the File

To use the custom partitioning results, we have modified the import interface to replace the original random method. Therefore, the command to run the program has been adjusted as follows:

```bash
./pagerank --graph_opts="ingress=random" --graph /data/in_S5P.txt --format self_tsv
```
- The pagerank program is compiled using the previous `make` command, and the executable is located in `release/toolkits/graph_analytics`.
- The `--graph` parameter specifies the path to the graph file.
- The `--format` parameter tells PowerGraph the structure of your graph file; `self_tsv` refers to the three-parameter file format mentioned above.


## Distributed PG Running Process

## 1. Download Docker Image

Detailed information has been provided in the previous section.

```bash
docker pull bearbiscuit/pgimage:latest
```

## 2. Download the Updated Version of PowerGraph

```bash
git clone https://github.com/BearBiscuit05/PowerGraph_update.git
```

## 3. Build the Cluster Environment

First, the Docker cluster must be on the same subnet, so you need to create a subnet:

```bash
docker network create --driver bridge --subnet 43.0.0.0/16 --gateway 43.0.0.1 pg_network
```

Next, modify the `runDocker.sh` script as needed (for example, to mount PowerGraph or graph data).

## 4. Run PowerGraph

After setting up the Docker cluster, enter any container and navigate to the directory where the PowerGraph program is located. Then run:

```bash
mpiexec --allow-run-as-root -n 2 -hostfile /root/data/machines ./pagerank --graph_opts="ingress=random" --graph /root/data/test.edges --format self_tsv
```
