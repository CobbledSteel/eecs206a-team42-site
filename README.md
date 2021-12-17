Motivation
==========

A fruit fly can compute workloads including trajectory planning,
visual/inertial odometry (VIO), classification, and closed-loop control,
all while only consuming 120 nW [@scheffer2021physical]. A
state-of-the-art VIO ASIC consumes 2 mW [@suleiman2019navion]---over
10,000$\times$ more power. With such a stark difference, biology
suggests vast optimization opportunities for autonomous systems.
However, closing the gap becomes increasingly challenging; with the end
of Moore's Law and Dennard Scaling, it is no longer feasible to rely on
process technology improvements and general purpose processors (GPPs) to
improve power efficiency and performance [@hennessy2019new]. As a
result, over the past decade there has been a proliferation of academic
research groups, startups, and industrial R&D labs developing domain
specific accelerators (DSAs) to eke out remaining performance
improvements. Although DSAs can offer orders of magnitude improved
performance over GPPs on isolated robotics benchmarks, factors such as
diminishing returns of acceleration due to Amdahl's Law, poor functional
unit utilization, contention over shared resources in heterogeneous
systems, and unsupported computational kernels limit acceleration at a
system-level. These issues are exacerbated in autonomous robotics
systems, which face strict power, latency, and quality of service (QoS)
constraints. These challenges present opportunities to improve robotics
DSA performance by making use of hardware-software co-design techniques
coupled with the full-stack evaluation of robotics SoC designs.

There are numerous projects using specialized hardware to accelerate a
variety of robotics tasks
[@fpgasurvey; @li2019fpga; @chretien2016gpu; @liang2018gpu; @murray2016microarchitecture; @lian2018dadu; @suleiman2019navion].
However, integrating and evaluating the performance of such hardware
accelerators at a system level remains challenging. While the latency
and throughput characteristics of robotics DSAs can be evaluated using
data traces generated by model-based simulations, such methodologies to
not capture system level feedback loops between hardware acceleration
and robot behavior. For example, prior work has shown that in
hardware-in-the-loop (HIL) setups, scaling clock frequency and
allocating more compute units can directly impact quality-of-flight
metrics in a quadrotor, such as flight velocity
[@boroujerdian2018mavbench]. Conversely, higher velocity can negatively
impact pose estimation accuracy [@delmerico2019we], impacting trajectory
planning and optimal control workloads which make use of iterative
algorithms with variable runtimes. Therefore, a closed-loop test setup
is needed for evaluating hardware accelerators for robotics. For designs
implemented on programmable hardware such as GPUs and FPGAs, closed-loop
evaluation is possible using hardware-in-the-loop (HIL) setups. However,
this cannot be done with ASIC designs unless the chip is manufactured,
which is a very expensive and time consuming task. Because of this, in
order perform pre-silicon performance evaluation of a robotics SoC it is
necessary to co-simulate the architectural behavior of hardware running
a full robotics software stack together with a robotics environment
modeling system dynamics and sensor data.

Project Description
===================

In this project we will develop co-simulation infrastructure to enable
the design space exploration of robotics SoCs. As a driving application,
we will look at autonomous quadrotor systems. This is because UAVs make
for an interesting design point due to the interactions between latency,
power, and weight constraints [@hadidi-designspace], as well as the fact
that the SoCs used onboard UAVs are comparable in scale to those
previously designed at the ADEPT Lab at UC Berkeley.

In our project we plan on simulating a drone's hardware and software
stack, along with a physical design to be used as a reference
implementation. While we have not selected a physical drone at this
point, we plan on implementing a design that implements the computers,
actuators, and sensors as depicted in
Figure [1](#fig:dronelectronics){reference-type="ref"
reference="fig:dronelectronics"}. In this configuration, we plan to have
both the flight controller and companion computer on-board the drone, as
this design point has more interesting constraints for the companion
computer SoC. A possible candidate drone for this project is the
[ASPLOS21-Drone](https://github.com/ramyadhadidi/ASPLOS21-Drone), an
open source drone released with detailed assembly documentation
[@hadidi-designspace]. In addition to the hardware, a preliminary draft
of the software stack we plan on running on both the flight controller
and companion computer is depicted in
Figure [3](#fig:dronesoftware){reference-type="ref"
reference="fig:dronesoftware"}.

![Electronics top level diagram for the proposed
UAV.[]{label="fig:dronelectronics"}](img/Hardware Architecture.png){#fig:dronelectronics
width="\linewidth"}

Secondly, a key component of our work will be developing the
co-simulation infrastructure for our UAV. Our work will build upon two
existing simulators. For simulating UAV dynamics and visual rendering we
plan on using [AirSim](https://microsoft.github.io/AirSim/), a simulator
based on Unreal Engine developed by Microsoft [@shah2018airsim]. For
cycle-accurate SoC simulation, we plan on using
[FireSim](https://fires.im), an FPGA-accelerated RTL simulator developed
at the ADEPT Lab at UC Berkeley [@firesim]. A top level diagram of our
planned infrastructure is depicted in
Figure [5](#fig:simarchitecture){reference-type="ref"
reference="fig:simarchitecture"}, with components that we expect to make
major modifications to highlighted in red. These components mainly
consist of the target-to-host bridges found in FireSim, which are
responsible for the communication and synchronization between the host
CPU managing the RTL simulation, and the target FPGA accelerating the
simulation. Our modifications would need to synchronize the clock cycles
elapsed in the RTL simulation with the amount of time simulated in
AirSim, as well as to schedule the data transfers between AirSim and the
SoC I/O modeled by FireSim. Before moving robotics software to the
FireSim simulations, we will evaluate the RISC-V ports in a QEMU session
as depicted in Figure [4](#fig:qemu){reference-type="ref"
reference="fig:qemu"}.

The final component of our project will involve generating SoC instances
on which we will evaluate our software stack. For our project, we will
focus on evaluating custom hardware for an on-board companion computer.
This is because the flight controller can be implemented using a
low-power microcontroller, and provides no benefit from being
accelerated as the frequency of the flight controller loop is bounded by
the physical properties of an UAV rather than by the available compute
capabilities [@hadidi-designspace]. On the other hand, accelerating high
level control tasks in a HIL setup has been shown to improve
quality-of-flight metrics in quadrotors, such as mission time and
maximum velocity [@boroujerdian2018mavbench]. Because these high level
control tasks run on the companion computer, we identified this unit for
our design-space exploration. Developing new custom hardware
accelerators is out of the scope of this project. However, we still plan
on evaluating configurations of existing hardware, including the
in-order Rocket CPU [@asanovic2016rocket], the out-of-order superscalar
BOOM CPU [@zhao2020sonicboom], and Gemmini, a systolic array hardware
generator [@genc2021gemmini]. We plan on generating hardware designs
using these components using Chipyard, an SoC generator developed by the
ADEPT Lab at UC Berkeley [@chipyard]. While discovering an optimal SoC
configuration is out of the scope of this project, we plan on using the
designs to evaluate the co-simulation infrastructure.

The project will incorporate the sensing and actuation through the use
of the ASPLOS21-Drone, which will act as a physical reference design for
the co-simulation infrastructure. However, sensing and actuation will
also be explored through the simulated environment. Similarly, high
level control and planning algorithms will be deployed on both the
physical and simulated drones.

![An example software stack for both the flight controller and the
companion
computer.[]{label="fig:dronesoftware"}](img/Flight Control.png "fig:"){#fig:dronesoftware
width="\linewidth"}\
![An example software stack for both the flight controller and the
companion
computer.[]{label="fig:dronesoftware"}](img/High Level Control.png "fig:"){#fig:dronesoftware
width="\linewidth"}

![Top level architecture for evaluating ROS workloads on the RISC-V
software stack.[]{label="fig:qemu"}](img/AirSim-QEMU.png){#fig:qemu
width="0.8\linewidth"}

![Top level architecture for the proposed co-simulation
architecture.[]{label="fig:simarchitecture"}](img/AirSim-FireSim.png){#fig:simarchitecture
width="\linewidth"}

Related Work
============

Co-Simulation of Custom SoC Hardware
------------------------------------

The co-simulation infrastructure presented in this paper builds upon
prior projects, integrating various functionality to support a full
stack robotics co-simulation infrastructure. To begin, this project
heavily depends on the RTL simulation and synchronization functionality
functionality provided by FireSim [@firesim]. FireSim already has
built-in synchronization support, used to provide deterministic behavior
both between FPGA targets, and FPGA targets and hosts. However, the
functionality that this project needs to build on top of FireSim is the
ability to have deterministic transactions that are initiated by the
host, as typically FireSim bridges are configured to be deterministic
with respect to transactions initiated by a target device. Additionally,
while co-simulation projects have been built using FireSim, such as the
Fromajo project, they differ in scope from this co-simulator. Fromajo is
used to validate FireSim simulations against the Dromajo [@dromajo]
architectural simulator [@zhao2020sonicboom]. Fromajo differs from this
co-simulation infrastructure, however, as it is meant to be a platform
for verification, and compares two instruction traces rather than
integrating two simulators to support closed-loop feedback.

Simulation-Based Design Space Exploration of UAV Hardware
---------------------------------------------------------

Several projects have used simulation methods to evaluate the impact of
custom hardware on the flight performance of UAVs. One significant work
presents MAVBench [@boroujerdian2018mavbench], a closed-loop
benchmarking suite based on AirSim. MAVBench profiled several UAV
workloads such as scanning, package delivery, and 3D mapping in a HIL
environment, running flight controller code on a Pixhawk board, and
running high level control code on an NVIDIA Jetson TX2. While the
benchmark did not explore custom robotics architectures, the authors
determined that hardware accleration could affect quality-of-flight
metrics such as maximum drone velocity, and total mission time. The
hardware acceleration explored included sweeps of the SoCs' clock speed,
as well as the number of cores allocated for robotics workloads.

Closed-Loop Simulation of Custom Robotics Hardware and Systems
--------------------------------------------------------------

Another work that is relevant to this project is a prior co-simulation
infrastructure developed at Linköping University [@acevedo2016fpga].
This project functions as a HIL setup, co-simulating an FPGA running
robotics workloads with the Wolfram SystemModeler simulation environment
[@rozhdestvensky2020description]. An FPGA and host computer are
connected using a serial interface for synchronization and data
transfer. This project differs from prior FPGA prototyping attempts as
it synchronizes FPGA cycles to match SystemModeler's update rate,
whereas prototyping projects run all systems directly in real-time.
However, this project lacks several features compared to the proposed
co-simulation infrastructure. First, rather than using a true
cycle-exact ASIC simulation, the HIL co-simulator synchronizes against
an FPGA implementation, which has different performance characteristics
compared to an ASIC [@firesim]. Secondly, the HIL co-simulator currently
only supports low-level hardware accelerators instead of an entire SoC
supporting a full Linux stack. Having full-stack support is important
for supporting and integrating projects that make use of the modern
open-source robotics ecosystem. Finally, this paper's co-simulation
infrastructure intends to support the ROS framework, allowing for a more
standardized approach for integrating robotic software components.

Finally, there have been prior attempts at co-simulating robotics
simulations on top of the Gazebo/ROS ecosystem. One such project,
CORNET, presents middleware that integrates a Gazebo simulation with a
multiple UAV flight controllers [@acharya2020cornet]. As in this
project, CORNET uses a custom Gazebo plugin to perform synchronization
with external simulators. However, CORNET is intended to provide
co-simulation between Gazebo and a network simulator instead of
cycle-exact hardware simulation, and so it has vastly different timing
and performance requirements compared to this co-simulation
infrastructure.

Based on this review, there have been many projects that support
elements of the infrastructure needed for closed-loop robotics ASIC
co-simulation. However, this project is novel as it integrates all these
aspects into one system.

Tasks, Milestones, and Assessment
=================================

This project will include a broad range of tasks, and relies heavily on
infrastructure development. Accounting for this, we do not plan on
accomplishing every task, given that there might be unexpected issues
related with third-party components. We divided the tasks into Base,
Target, and Reach, where we plan to complete base tasks by mid November,
Target tasks by the project deadline, and Reach tasks if time permits.
As this is a continuing research project, we plan on continuing this
infrastructure development after the semester ends.\
We will assess the success of this project both on the milestones met,
but also by the documentation and analysis of areas of improvement in
the robotics, open source hardware, and electronic design automation
communities that we encounter while working on this project.

Physical UAV Prototyping
------------------------

-   **(Base) Obtain FAA licenses and register drone:** Needed to legally
    pilot drones for recreational/research purposes. Can be filed
    online.

-   **(Base) Assemble ASPLOS21-Drone:** Purchase the parts listed in the
    BOM and follow the assembly instructions as in the ASPLOS21-Drone
    BuildGuide. Ensure that the drone functions using manual controls.

-   **(Base) Deploy flight controller:** Deploy ArduPilot onto the drone
    hardware, and verify that it can perform takeoff/landing as well as
    waypoint tracking.

-   **(Target) Develop basic high level control in ROS:** Deploy
    algorithms including mapping, localization, perception, and
    trajectory planning.

-   **(Reach) Evaluate UAV performance:** Verify that the system
    displays expected functionality, and note potential improvements.

-   **(Reach) Optimize high level control in ROS:** Make improvements to
    algorithms and scheduling to improve system-level performance.

Porting ROS libraries to RISC-V
-------------------------------

-   **(Base) Port core ROS middleware:** Ensure that core ROS libraries
    are functional when compiled for RISC-V, demonstrating functionality
    of a ROS master as well as `roscpp` or `rospy`.

-   **(Target) Port integration-level libraries:** Ensure that standard
    or commonly used libraries such as `sensor_msgs`, `geometry_msgs`
    and `tf2` function properly.

-   **(Reach) Port application-level libraries:** Build and verify the
    functionality of libraries such as MoveIt, gmapping, and OpenCV.

Developing Co-simulation Infrastructure
---------------------------------------

-   **(Base) Interface with AirSim from QEMU session:** Transmit
    waypoints to AirSim from a RISC-V QEMU session, and receive sensor
    data through the AirSim APIs.

-   **(Target) Integrate ROS in QEMU with AirSim:** Run ROS code ported
    to RISC-V running high-level control, deploying setpoints to and
    reading sensor data from AirSim.

-   **(Target) Interface with AirSim from FireSim:** Transmit waypoints
    to and receive sensor data from AirSim from a simulated SoC within
    FireSim.

-   **(Reach) Integrate ROS on FireSim with AirSim:** Run ROS code on
    FireSim, communicating with AirSim.

-   **(Reach) Implement lockstep time synchronization between AirSim and
    FireSim:** Create a synchronizer bridge between FireSim and Airsim,
    using custom hardware to ensure lockstep synchronization between
    AirSim frames and FireSim cycles.

-   **(Reach) Implement deterministic data synchronization between
    Airsim and FireSim:** Implement a system for scheduling and
    releasing data transfers at deterministic time intervals between
    AirSim and FireSim, stalling simulation in case of unexpected
    network delays.

Generating Robotics SoC Designs in Chipyard
-------------------------------------------

-   **(Base) Single Rocket Core:** Generate hardware using a single
    Rocket in-order CPU.

-   **(Target) Multi-core Rocket:** Generate hardware with 4-8 Rocket
    cores.

-   **(Target) Single BOOM Core:** Generate hardware using a BOOM
    out-of-order superscalar CPU.

-   **(Reach) Heterogeneous Rocket/BOOM SoC:** Generate design with both
    high performance BOOM cores and efficient Rocket cores.

Documenting Challenges
----------------------

-   **Software Challenges:** Did any of the software/algorithms not work
    as expected? Are there any potential improvements?

-   **Software Infrastructure Challenges:** Are there any missing
    libraries or tools that prevent porting some software libraries to
    RISC-V? Are there deficiencies with simulators impacting integration
    for co-simulation?

-   **Hardware Challenges:** Do existing configurations face significant
    bottlenecks for the given workloads?

-   **Hardware Infrastructure Challenges:** Are there missing
    features/IP that impact the ability to port applications to RISC-V?
    Are there limitations of FPGA-accelerated simulations that impact
    co-simulation performance?

-   **Unexpected Issues:** Any other legal/social/mechanical/etc.
    concerns?

Team Member Roles
=================

Dima Nikiforov
--------------

Dima will be in charge of tasks involving porting software libraries to
RISC-V, developing FireSim to support co-simulation, and generating
hardware designs, given their experience working with similar
infrastructures and environments at the ADEPT Lab.

Chris Dong
----------

Chris will be developing the software infrastructure via ROS and AirSim,
setting up AirSim in AWS server and running built-in simple flight
controller, along with developing and testing high level algorithms both
in simulation and on the real drone.

Collaboration
-------------

While we plan on collaborating throughout the project, we will make sure
to only do drone hardware prototyping and testing when both group
members are present in order to follow lab safety protocols. We will
also collaborate heavily to ensure that we can successfully integrate
the infrastructure components that we develop.

Bill of Materials
=================

Use of Lab Resources
--------------------

We do not plan on using any of the EECS 206A lab resources for this
project.

Other Robotic Platforms
-----------------------

We plan on using the ASPLOS21-Drone to perform physical prototyping for
this project.

![image](img/206A.pdf){width="\linewidth"}

Items for Purchase
------------------

This project will involve purchasing components for physical
prototyping, as well as paying for the use of AWS infrastructure for
software development and running GPU and FPGA accelerated simulations.
The bill of materials for the drone is shown in
Figure [\[fig:dronebudget\]](#fig:dronebudget){reference-type="ref"
reference="fig:dronebudget"}. Additionally, we plan on using the
following AWS EC2 instances using on-demand pricing: `c5.4xlarge`
(Managing FireSim simulations, general software development),
`g4dn.2xlarge` (Running GPU-accelerated drone simulations using AirSim),
and `f1.2xlarge` (Running FPGA-accelerated RTL simulations in FireSim.)
Funding for purchasing components will be provided by grants through the
ADEPT Lab.

References
============
[1] L. K. Scheffer, “The physical design of biological systems-insights from
the fly brain,” in Proceedings of the 2021 International Symposium on
Physical Design, 2021, pp. 101–108.

[2] A. Suleiman, Z. Zhang, L. Carlone, S. Karaman, and V. Sze, “Navion:
A 2-mw fully integrated real-time visual-inertial odometry accelerator
for autonomous navigation of nano drones,” IEEE Journal of Solid-State
Circuits, vol. 54, no. 4, pp. 1106–1119, 2019.

[3] J. L. Hennessy and D. A. Patterson, “A new golden age for computer
architecture,” Communications of the ACM, vol. 62, no. 2, pp. 48–60,
2019.

[4] Z. Wan, B. Yu, T. Y. Li, J. Tang, Y. Zhu, Y. Wang, A. Raychowdhury,
and S. Liu, “A survey of fpga-based robotic computing,” 2021.

[5] R. Li, X. Huang, S. Tian, R. Hu, D. He, and Q. Gu, “Fpga-based
design and implementation of real-time robot motion planning,” in 2019
9th International Conference on Information Science and Technology
(ICIST). IEEE, 2019, pp. 216–221.

[6] B. Chr ́etien, A. Escande, and A. Kheddar, “Gpu robot motion planning
using semi-infinite nonlinear programming,” IEEE Transactions on
Parallel and Distributed Systems, vol. 27, no. 10, pp. 2926–2939, 2016.

[7] J. Liang, V. Makoviychuk, A. Handa, N. Chentanez, M. Macklin, and
D. Fox, “Gpu-accelerated robotic simulation for distributed reinforce-
ment learning,” in Conference on Robot Learning. PMLR, 2018, pp.
270–282.

[8] S. Murray, W. Floyd-Jones, Y. Qi, G. Konidaris, and D. J. Sorin, “The
microarchitecture of a real-time robot motion planning accelerator,” in
2016 49th Annual IEEE/ACM International Symposium on Microarchi-
tecture (MICRO). IEEE, 2016, pp. 1–12.

[9] S. Lian, Y. Han, X. Chen, Y. Wang, and H. Xiao, “Dadu-p: A scalable
accelerator for robot motion planning in a dynamic environment,” in
2018 55th ACM/ESDA/IEEE Design Automation Conference (DAC).
IEEE, 2018, pp. 1–6

