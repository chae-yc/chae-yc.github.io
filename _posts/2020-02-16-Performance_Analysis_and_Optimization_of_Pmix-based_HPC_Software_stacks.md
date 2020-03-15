---
title: "Parallel process management - MPI"
categories:
  - scientist
  - paper_reading
tags:
  - hpc
  - pmix
last_modified_at: 2020-02-16T18:58:00+09:00
toc: true
published: true
share: false
---

## 1 Introduction

> The objective of this work is to identify and address **performance critical areas** of existing PMI implementation.
Our vehicle for this work is the PMI Exascale(PMIx) standard as the state-of-the-art  PMI that was designed with exascale in mind.

> PMIx provides advanced capabilities to enable efficient bootstrapping of applications on emerging system.

PMIx supports than PMI-1, PMI-2:

1. extended job level information allowing for the exposure of RM knoledge
2. on-demand mode of data exchange.

HPC stacks that rely on PMIx and include:

- PMIx Reference Implementation (PRI)
- RunTime Environment (RTE) implementation; Slurm,  IBM LSF/JSM, Open MPI ORTE
- HPC communication libraries; UCX, IBM PAMI
- parallel programming environments; MPICH, Open MPI/SHMEM, Mellanox HPCX, IBM Spectrum MPI

Focused efforts ton three distinct performance-critical  areas

- improving PRI intranode communication with new locking and thread-safety schemes
- optimizing the amount of information communicated during an application startup due to more efficient data representation in PRI and UCX libraries
- proposing a highly efficient all-gatherv inspired by HPC collective communication design that significantly improves the performance and scalability of the PMIx data exchange.

## 2 Background

### 2.1 Process Management Interface

#### Launch of a parallel application

##### Runtime Deployment

> The deployment of a distributed RTE on a subset of HPC system nodes allocated to a job. In most of the existing software stacks, this step assumes the creation of a single RTE instance per compute node in the allocation. These instances are later used to create and manage parallel application processes.

분산 RTE를 배포 작업은 HPC 시스템의 하위 노드들에 작업을 할당한다. 현재 대부분의 소프트웨어 스택에서, 이 과정은 모든 compute node에 단일 RTE instance를 생성하는 것을 가정한다. 이 instance들은 병렬 프로세스들을 생성하고 관리하는 데에 사용된다.

##### Launch of application processes

> The creation of processes that form a parallel application by the corresponding RTE instances.

병렬 프로세스들은 서로 일치하는 RTE instance들에 의해서 생성된다.

##### Predefined environment discovery

> In this stage, each process is provided with access to job-level information describing the portion of that application’s environment known by the RTE.
>
> For example, some common information retrieved on this step includes:
>
> - A unique integer identifier (rank) of a process in the job
> - A total amount of processes in the application
> - A list of processes sharing the same node

이 과정에서는, 각 프로세스들이 job-level information에 대한 접근권한을 제공하게 된다.
job-level information은 application 환경의 일부를 설명한다.
information들은 RTE에 의해서 알 수 있다.

이 작업에서 알게 되는 정보들의 예는,

- 작업에서 해당 프로세스의 unique integer identifier(rank)
- 프로세스의 갯수
- 같은 노드를 공유하는 프로세스의 목록

##### Dynamic environment discovery

> In this phase, application processes are exchanging application-specific data that is required for the execution but not provided on the previous step. This step mainly deals with communication addressing information that is dynamically assigned to each process by the interconnect fabric.

이 과정에서는, application 프로세스들이 이전 단계에서 제공되지 않았지만 실행에 필요한 특정 데이터를 교환한다. 이 단계는 주로 서로 연결된 fabric에 의해 각 프로세스에 동적으로 할당되는 통신에 사용되는 주소 정보 (communication addressing information)를 처리한다.

병렬 프로그래밍 모델을 구현하는 고급 소프트웨어 스택은 가능한 많은 정보를 사전 정의 된 환경 검색 단계(Predefined environment discovery phase)로 이동시키고 있다. 현재 실행 환경을 확인하고 그에 맞춰진 주소 정보 얻는 방식인 instant-on initiative가 개발되고 있지만 아직 다른 공급자들이 많이 채택하지 않았기 때문에 이 논문에서는 동적 환경 검색(Dynamic environment discory)을 포함한 소프트웨어 스택을 사용한다.

Three primary compnents in HPC software stack

- Parallel programming paradigm implementation
- PMI implementation
- RTE software (RTE library, Resource Manager)

> Parallel job processes are communicating with the RTE through an API defined by PMI. The PMI information is organized in the form of a Key-Value Database (KVDb) that is being accessed through *Put* and *Get* primitives.

병렬 프로세스들은 PMI에서 정의한 API를 통해서 RTE와 통신한다. PMI의 정보들은 는 Put과 Get을 사용하여 접근되는 Key-Value Database에서 구성된다.

> Different versions of PMI also provide a global (Fence) and optionally a local (Commit) KVDb synchronization primitives that are required to ensure that the information provided with Put operations is Get-accessible.

다른 버전의 PMI는 정보에 액세스 하도록 Put과 별도로 전역용(Fence) 그리고 (선택적으로) 로컬용(Commit) KVDb 동기화 함수들도(primitives)도 제공한다.

### 2.2 PMI generations

PMI-1 = part of the MPICH project

- KVDb as the form of data exchange
- Put, Get, Fence (aka PMI_Barrier), Commit primitives

Fence operation의 근거(rationale)는 RTE implementation에서 기존의 집단 교환 알고리즘(collective exchange algorighms)을 활용하여 데이터 분배 단계(data distribution step)를 최적화 할 수 있도록 하는 것이다. PMI-1이 제공하는 유일한 *job-level* information(작업수준 정보)은 프로세스의 rank와 갯수다.

PMI-2 는 PMI-1의 한계에 집중하여 개발되었다.

- Query functionality: RM에서 제공한 키들의 집합 그리고 특별한 primitive들에 의한 접근에 대한 개념이 추가되었다. 예를 들어 SMP system의 node-local rank들 목록. 이 기능은 일부 데이터를 predefined environment으로 이동해서 데이터가 차지하는 부분의 크기를 줄인다.
- KVDb information scope: 펜스 작업 중 교환되는 데이터 양을 줄이기 위해 전역 및 노드 수준 범위의 개념이 도입되었습니다. 노드 로컬 구성 요소에 액세스하기 위해 특수 API 루틴이 정의되었습니다.
- *Get* operation: 타겟의 rank를 특정하는 선택적 매개 변수를 사용하여 구현되었다.

PMIx 인터페이스는 이전 호환성을 유지하는 이전 API의 후속 버전으로 설계되었다.

- 단일 Get 작업을 통해 KVDb에 통합 접근(consolidated access).
작업, 노드 및 프로세스 레벨 정보들이 구분되어 있지 않다.
- Extended KVDb information scopes: process-local, node-local, remote-only and global.
- Extended set of datatypes for key values(PMI-1,2 에서는 string only 였음)
- *Fence* and *Get*의 Non-blocking version. [ref](https://dl.acm.org/doi/10.1145/2642769.2642780)
- On-demand data exchange (aka Direct Business Card Exchange)

• 드문 통신 그래프가있는 응용 프로그램을 대상으로하는 주문형 데이터 교환 (일명 직접 명함 교환 [6]).
• 런타임 환경과의 통합을 단순화하는 서버 측 인터페이스의 정의.


