# NONA - A Framework for Elastic Streaming Forward Provenance over Dynamic Query Sets

This is the repository accompanying the research paper _NONA - A Framework for Elastic Streaming Forward Provenance over Dynamic Query Sets_. It enables the replication of all experiments in the paper.

## Setup

This setup has been tested on the Linux distributions CentOS 7 (x64), Ubuntu 20.04.2 LTS (x64) and Ubuntu 18.04.6 LTS (arm).

### Requirements

Executing the experiments requires access to two machines. One of these will serve as the data provider for the experiments and must be reachable via `ssh`.
You will need the following programs on your main machine:

*  `git`
* ` wget`
*  `tar`
*  `conda`
*  `java 8`
*  `mvn`
*  `gcc`
*  `g++`
*  `make` (install via `build-essential`)
*  `cmake`

Furthermore, on the remote machine / data provider machine you will need:

* git
* java 8 
* mvn
* unzip


### Procedure

1. Download this repository. We will in the following refer to the root folder of this repository as `ROOT`.
2. To download and unpack Apache Flink 1.10, execute the following commands in `ROOT`: 
```
wget https://archive.apache.org/dist/flink/flink-1.10.0/flink-1.10.0-bin-scala_2.11.tgz
tar zxvf flink-1.10.0-bin-scala_2.11.tgz
```
3. To download and unpack Apache Kafka 3.2.1, execute the following commands in `ROOT`:
```
wget https://archive.apache.org/dist/kafka/3.2.1/kafka_2.13-3.2.1.tgz
tar zxvf kafka_2.13-3.2.1.tgz
```
4. To download and install kcat on your machine, first create a new directory NOT in `ROOT`:
```
mkdir /some/folder
```
Then, to download the kcat repository, in `/some/folder` execute
```
git clone https://github.com/edenhill/kcat.git
```
Move into the folder and run the setup script:
```
cd /some/folder/kcat; ./bootstrap.sh
```
Finally, add the following to the end of your `bashrc` file (usually found at `~/.bashrc`)
```
kcat() {
        /some/folder/kcat/kcat "$@"
}
export -f kcat
```
And source your edited `bashrc`:
```
source ~/.bashrc
```
Now, the command `kcat` is functional.
5. On your remote machine at `XXX.XX.XX.XXX`, download this repository. We will refer to the root folder of this repository on your remote machine as `REMOTE_ROOT`. Then, in `REMOTE_ROOT`, execute the following to package the required java classes:
```
mvn clean package
```
Then, download the datasets on the remote machine. On the remote machine, execute the following commands in `REMOTE_ROOT/scripts`:
```
bash linear_road_downloader.sh
bash car_local_downloader.sh
bash mhealth_downloader.sh
```
This will download and extract the datasets into the folder `REMOTE_ROOT/input`.
Finally, download Kafka on the remote machine. In `REMOTE_ROOT`, execute
```
wget https://archive.apache.org/dist/kafka/3.2.1/kafka_2.13-3.2.1.tgz
tar zxvf kafka_2.13-3.2.1.tgz
```

6. Back on your main machine, to enable communication with your remote machine at `XXX.XX.XX.XXX`, edit the file `ROOT/configs/global_config.yaml`: 

* at `kafka_server_port_remote`, enter `"XXX.XX.XX.XXX:9092"`
* at `remote_root`, enter `"REMOTE_ROOT"`
* at `ssh_prefix`, enter `ssh your_remote_username@XXX.XX.XX.XXX`.

**NOTE:** It is important that you have set up passwordless ssh login to your remote machine, as explained for example [here](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/).

7. To setup the Anaconda environment, run in `ROOT`:
```
conda env create -f environment.yml
```


## Running Experiments

There are four individual experiments in the paper, corresponding to experiment scripts in this repository in the folder `ROOT/experiments`. 
These are run on specific devices, server (Intel Xeon Phi, 72 cores, 1.5GHz, 102GB Ram) and Odroid (Samsung Exynos 5422, Cortex A15 / Cortex A7 octacore, up to 2Hz, 2GB Ram) 
(your hardware configuration may differ).
Furthermore, the maximum duration of each experiment and the number of repetitions may vary.
The table lists the mapping between figure number in the paper, experiment script, device, duration (in minutes), repetitions, and the plot name (used later):

| figure number | script                            |  device | duration | reps |  plot_name   | 
| ------------- | --------------------------------- | ------- | -------- | ---- | ------------ |
| 5 (a)             | LR_static_overheads_odroid.yaml   | odroid  | 10       | 10   | lr_overheads |
| 5 (b)             | CL_static_overheads_server.yaml   | server  | 10       | 10   | cl_overheads |
| 6 (a)             | LR_static_odroid.yaml             | odroid  | 10       | 10   | lr_static    | 
| 6 (b)            | RI_static_odroid.yaml             | odroid  | 10       | 10   | ri_static    | 
| 6 (c)            | CL_static_server.yaml             | server  | 10       | 10   | cl_static    | 
| 7 (a)            | LR_pyramid_server.yaml            | server  | 25       | 10   | lr_dynamic   | 
| 7 (b)            | RI_pyramid_server.yaml            | server  | 25       | 10   | ri_dynamic   | 
| 8            | synthetic_descending_server.yaml  | server  | 25       | 10   | synthetic    |

(`odroid`: Odroid XU4 2016a, ARM; `server`: Intel Xeon-Phi server with 72 1.5GHz cores, x64)
**Note:** Experiment results will differ on different hardware. 


To run an experiment as the server or the Odroid, execute the following steps (using `device` as placeholder for `server` or `odroid`):

1. Copy the correct Flink configuration. In `ROOT`, execute
```
cp configs/device/flink-conf.yaml flink-1.10.0/conf/.
```
2. Activate the `conda` environment:
```
conda activate nona
```
3. In the folder `ROOT/scripts`, execute
```
python run.py ../experiments/SCRIPT REPS DURATION
```
where script is the chosen experiment script. This will run the experiment described in `SCRIPT` for `REPS` times, with each run taking at most `DURATION` minutes.

**NOTE:** During experiment execution, a host of debugging and logging information is printed to screen in addition to information about the remaining runtime of the experiment. 
It is safe to ignore this information, at the end of each experiment the script will output in detail which runs succeeded, which failed, and where the experiment output is stored.

### Running custom experiments

To run custom experiments, take a look at the `yaml` experiment scripts in `ROOT/experiments`. Especially, each `yaml` experiment script designates a file describing the sequence of transitions (_procedure_) of the dynamic query set. 
These procedure scripts are located at `ROOT/experiments/procedure_scripts`, and allow you for example to change the query that is added or removed, change the number of queries, or even implement a custom transition procedure.


## Visualizing results

We provide a python script to recreate the figures from the paper.
To execute the script, activate the `conda` environment
```
conda activate nona
```
Then, in the directory `/ROOT/scripts/visualization`, call the plotting facility:
```
python plotter.py PLOT TARGET_FOLDER [--show-in-popup]
```
where `TARGET_FOLDER` is the folder in which the output of your experiment runs is stored 
(the exact folder path is printed to the terminal after running `run.py` and will be inside `ROOT/results`). The optional flag will display the experiment plot in a popup window.
See the table in section _Running Experiments_ above, column _plot_name_, for appropriate values for the `PLOT` parameter.
