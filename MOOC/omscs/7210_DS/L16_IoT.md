# Lesson 16 Edge Computing & IoT (Internet of Things)

## 16.1 Introduction

**New tiers** in distributed computing infrastructure

- **Edge computing**
- **IoT**
- **Impact on design tradeoffs** from characteristics of new tiers

## 16.2 Tiers in Computing

![](images/2021-05-01-23-14-31.png)
![](images/2021-05-01-23-16-22.png)

## 16.3 Why Edge Computing?

![](images/2021-05-01-23-17-34.png)
![](images/2021-05-01-23-18-17.png)
![](images/2021-05-01-23-20-02.png)

## 16.4 Closing the Latency/Bandwidth Gap

![](images/2021-05-01-23-21-19.png)
![](images/2021-05-01-23-23-59.png)
![](images/2021-05-01-23-21-49.png)

- Use compute at the network edge to deliver latency, bandwith, privacy, ....
- Edge Computing, Mobile/Multi-access Edge Computing(MEC)

## 16.5 Is Edge Computing New?

![](images/2021-05-01-23-28-00.png)
![](images/2021-05-01-23-28-23.png)
![](images/2021-05-01-23-29-13.png)
![](images/2021-05-01-23-29-50.png)

## 16.6 Edge Computing Drivers

#### Fundamental Driver

- Speed of light
- Energy (growth of data)
- Regulatory, data sovereignty

##### "Killer" Apps:

- 5G
- New Video
- AR/VR/XR
- IoT and automation
- Cognitive tasks
- Enterprise/private LTE
- ...

![](images/2021-05-01-23-32-29.png)

#### Bandwith as a Driver

- Backhaul connectivity
- Asymmetric upload/download speeds

![](images/2021-05-01-23-33-39.png)
![](images/2021-05-01-23-37-51.png)

## 16.7 Distributed Edge Computing

#### Edge != Cloud

- Scale, geo-distribution
- "Chatty" protocols not appropriate
- The edge is not elastic
- Mobility, device churn, reliability
- Variability/heterogeneity
- Localization, contextualization
- Decentralization
- Lack of physical security

## 16.8 IoT and Distributed Transactions

#### IoT Based Edge Services

- Cloud + edge/gateways + smart devices
- Sensing and actuation
  ![](images/2021-05-01-23-45-25.png)

#### Intrusion Detection Application

![](images/2021-05-01-23-45-53.png)

#### Failure Example

![](images/2021-05-01-23-47-25.png)

#### Dependencies

![](images/2021-05-01-23-49-09.png)

## 16.9 Transactuations

- **High-level programming abstraction and model**
  - `perform(applicationLogic, [sensorList, timeWindow, sensingPolicy],[actuatingPolicy]`
- `onSuccess()` and `onFailure()` methods
- **Atoic durability** of actuation
- Avoid **concurrency bugs**

#### Transactuations Invariants

- **Sensing Invariant**
  - Transactuation **executes only when staleness of its sensor reads is bounded**, as per specified sensing policy
- **Sensing Policy**
  - How much staleness is acceptable?
  - How many failed sensors is acceptable?
- **Sensing Example**

  - At least one co2 sensor can be read within last 5 mins

- **Actuation Invariant**
  - When a transactuation commits its app states, enough actuation have succeeded as per specified actuation policy
- **Actuation Policy**
  - How many failed actuation is acceptable?
- **Actuation Example**
  - At least one alarm should successfully turn on

#### Execution Model

![](images/2021-05-02-00-09-33.png)

## 16.10 Evaluation of Transactuations

![](images/2021-05-02-00-11-51.png)

#### Evaluation: Impact on Programmability

![](images/2021-05-02-00-12-36.png)

#### Evaluation: Impact on Failure-Free Execution

![](images/2021-05-02-00-12-59.png)

## 16.11 Lesson 16 Summary

- **New Infrastructure tiers** in distributed computing => **impact on rethinking design** of distributed systems and concepts
- **Edge computing**
- **Transactuations:** Distributed Transactions for **IoT** and **the Edge**
