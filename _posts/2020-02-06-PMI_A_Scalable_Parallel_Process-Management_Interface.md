---
title: "논문읽기 - PMI: A Scalable Parallel Process-Management
Interface for Extreme-Scale Systems"
categories:
  - scientist
  - paper_reading
tags:
  - hpc
  - mpi
  - pmi
  - mpich
  - openmpi
last_modified_at: 2020-02-05T17:58:00+09:00
toc: true
published: true
share: false
---

본 글은
**PMI: A Scalable Parallel Process-Management Interface for Extreme-Scale Systems -Pavan BalajiDarius BuntinasDavid GoodellWilliam GroppJayesh KrishnaEwing LuskRajeev Thakur. (EuroMPI 2010: Recent Advances in the Message Passing Interface) pp 31-41**
를 읽고 개인적으로 정리한 글입니다.
문제가 발생될 시 포스트를 삭제하도록 하겠습니다.

This post is a personalized summary of
**PMI: A Scalable Parallel Process-Management Interface for Extreme-Scale Systems -Pavan BalajiDarius BuntinasDavid GoodellWilliam GroppJayesh KrishnaEwing LuskRajeev Thakur. (EuroMPI 2010: Recent Advances in the Message Passing Interface) pp 31-41**
I will delete when any problems occured.

## Abscract

MPICH2 uses PMI (PMI1, PMI2)
PMI allows different process managers to interact with the MPI library in standardized way.

## 1. Introduction

Although the growing scale of HPC systems requires close interaction between the parallel programming library (such as MPI) and the process manager, an appropriate separation between these two components is necessary.

scalable design of the process-management infrastructure is critical for

- launching
- management of parallel applications
- debugging utilities, management tools
  - A process-management system must,
    - start and stop processes in a scalable way
    - provide mechanisms for the processes in a parallel job to exchange the information to establish communication

In this paper, PMI is introduced as a generic process-managementg interface for parallel applications.

PMI1 is widely used following areas

- in MPI implementation for the programming library side
  - MPICH2
  - MVAPICH2, Intel MPI, SiCortex MPI, Microsoft MPI are derived from PMI1.
- in process-management frameworks for the process-manager side
  - MPICH2's internal process managers(Hydra, MPD, SMPD, Gforker, Remshell)
  - Slurm, OSC mpiexec, OSUmpirun

## 2. Requirements of a Process-Management Interface

### 2.1 Decoupling the Process Manager and Process-Management Interface

Process management compirses tree primary components

1. Parallel programming library (such as MPI)
2. PMI library
3. process manager

process manager is a logically centralized process (but often a distributed set of process in practice) that manages

1. process launcing
  - starting/stopping processes
  - providing the environment information to each process
  - stdin/out/err forwarding
  - propagating signals
2. information exchange between processes in a parallel application

Several process managers already provide such capabilities.
(e.g. PBS, SUN Grid Engine, SSH)

Following figure is an illustration of different MPI libraries, PMI libraries, and process managers.

![fig1](/assets/images/2020-02-06-PMI_A_Scalable_Parallel_Process-Management_Interface/interaction.png)

PMI library provides the PMI API. 
The PMI library might depend on the system itself.
In some cases, such as IBM Blue Gene/L(BG/L) may use system-specific features to provide PMI service.
In other cases, such as processes on a typical commodity cluster, the PMI library can communicate with the process manager over a communication path (e.g. TCP).

While the PMI library can be implemented in any way, PMI-1 and PMI-2 uses predefined *wire protocol* which exchange data through the socket interface.

**Advantage**: Any application using PMI API with predefined PMI *wire protocol* is compatible with any PMI process manager that accepts the wire protocol.

PMI API and PMI wire protocol are separate entities.
For example, BG/L provides PMI API but doesn't use the sockets-based wire protocol.

### 2.2 Overview of the First-Generation PMI (PMI-1)

skip

### 2.3 PMI Requirements for the Process Manager

Two primary requirements for the process manager.

1. separation of features is needed to enable layering on a "native" process manager with the lowest possible overhead.
  - For example, an interface that requires asynchronous processing of data or interrupts to manage data might cause additional overhead for applications even when they are not interacting with the PMI services. (데이터를 비동기 처리하거나 데이터를 관리하기 위해서 interrupt를 요구하는 interface는 PMI 서비스 작업을 하지 않을 때에도 overhaed를 유발 할 수 있다.)
2. scalable data-interchange approach for the key-value system is needed
  - Consider, each process in a parallel job starts, <u>creates a contact id</u> and <u>available to the other processes in parallel job</u>
  - Simple way: provide the data to central server. This approach is not scalable.    
  - The solution in PMI provides a collective abstraction using efficient collective algorithms to provide more scalable behavior.
    
- Simple way(Single Central Server)
  - If all processes add/extract a (key, value) pair data into a database, time complexity is O(p);
- PMI solution
  - processes put data into a key-value space (KVS).
  - then collectively perform a fence operation.
  - fence operation, which is collective over all processes, provides an excellent opportunity for the implementation to distribute the data supplied by the put operations in a scalable manner.
  - Following completion of the fence, all processes can perform a get operation against the KVS.