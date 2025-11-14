# Modifications Introduced

This repository contains an extended version of the FLoRaSat framework with a
new **energy-efficiency module (ECO-SAT)** designed to compute energy
consumption of LoRa IoT devices in Direct-to-Satellite (DTS) scenarios.  
The modifications focus primarily on the `LoRaRadio.cc` module, enabling
fine-grained tracking of Tx, Rx, Idle, and Sleep states based on LoRa PHY
parameters.

These extensions were developed to support the research described in the
accompanying article on energy optimization in satellite-based LoRaWAN systems.

## Summary of Modifications in OMNeT++ / INET

Below is a concise summary of the main changes introduced into the original
OMNeT++ LoRa radio.

### **1. Partial rewrite of the reception logic**
INET’s built-in reception handling was bypassed:

```sh
// FlatRadioBase::startReception(timer, part);
```

and replaced by a custom LoRa-specific routine: 

```sh
auto isReceptionSuccessful =
    medium->getReceptionDecision(this, signal->getListening(), transmission, part)
    ->isReceptionSuccessful();
```

This allows explicit control over reception start, success/failure, and duration
of each radio state.

### **2. Insertion of an explicit LoRa PHY preamble (LoRaPhyPreamble)**

The preamble is now added at the beginning of each transmitted packet:

```sh
auto preamble = makeShared<LoRaPhyPreamble>();
packet->insertAtFront(preamble);
```

and removed during decapsulation:

```sh
auto preamble = packet->popAtFront<LoRaPhyPreamble>();
```

This exposes real LoRa PHY parameters (SF, BW, CR, Tx power) to the energy
consumer.

### **3. New signal for tracking dropped packets**

A new signal was created to capture failed receptions that the original INET
radio does not expose:

```sh
simsignal_t LoRaRadio::droppedPacket =
    cComponent::registerSignal("droppedPacket");
emit(LoRaRadio::droppedPacket, 0);
```

### **4. Explicit handling of transmission power**

Transmission power is now computed and stored manually to support the new
energy model:

```sh
setCurrentTxPower(10 * log10(tag->getPower().get() * 1000.0));
```

A SignalPowerReq tag is also forced into each packet:

```sh
auto signalPowerReq = packet->addTagIfAbsent<SignalPowerReq>();
signalPowerReq->setPower(tag->getPower());
```


---------------------------------------------------------------------------------------------------



# Installing the FLoRaSat Framework

These instructions are for setting up the FLoRaSat framework for use with OMNeT++.

## Prerequisites

1.  **OMNeT++:** You must have `OMNeT++` version `6.0.3` installed.
2.  **INET Framework:** In the OMNeT++ IDE, go to the menu **Help >> Install Simulation Models...** and install the `INET` framework, version `4.3.9`, into your workspace.

## Installation Steps (Terminal)

1.  Open a terminal and navigate to your OMNeT++ workspace directory. This is typically where you store your simulation projects.

    ```sh
    # Navigate to your workspace (adjust the path if necessary)
    cd ~/omnetpp-6.0.3/workspace
    ```

2.  Clone the FLoRaSat repository from GitLab.

    ```sh
    git clone [https://gitlab.inria.fr/jfraire/florasat.git](https://gitlab.inria.fr/jfraire/florasat.git)
    ```

3.  Navigate into the newly cloned directory.

    ```sh
    cd florasat
    ```

4.  Check out a specific, known-working commit.
    > **Note:** The `master` branch of FLoRaSat may have compatibility issues with the required versions of OMNeT++ and INET. The following commit is known to be stable with this setup.

    ```sh
    git checkout 989866236a3ded66dbac60a132609fb859ef98aa
    ```

5.  (Optional) Create a new branch from this commit to save your work.

    ```sh
    # Creates a new branch named 'stable-build'
    git checkout -b stable-build
    ```

## Configuring in the OMNeT++ IDE

1.  Open or return to the OMNeT++ IDE.
2.  Go to the menu **File >> Open Projects from File System...**.
3.  Select the `florasat` project directory you just cloned and add it to the workspace.
4.  In the Project Explorer, right-click on the `florasat` project and go to **Properties**.
5.  In the Properties window, navigate to **Project References** and ensure that only `inet` (version 4.3.9) is selected.
6.  Finally, right-click the `florasat` project again and select **Build Project**.
