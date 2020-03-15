---
title: "NAS Parallel Benchmark - NPB"
categories:
  - scientist
tags:
  - NAS Parallel Benchmark
  - NPB
  - MPI
last_modified_at: 2020-02-13T02:13:00+09:00
toc: true
published: true
share: false
---
## Introduction

NAS

## Benchmarks

### Kernel

|Benchmark|Name derived from|Description|Remarks|
|---------|-----------------|-----------|-------|
|IS|**I**nteger **S**ort|Sort small integers|rm|
|EP|**E**mbarrassingly **P**arallel|ds|rm|
|CG|**C**onjugate **G**radient|ds|rm|
|MG|**M**ulti**G**rid|ds|rm|
|FT|Fast **F**ourier **T**ransform|ds|rm|

### Pseudo applications

|Benchmark|Name derived from|Description|Remarks|
|---------|-----------------|-----------|-------|
|BT|**B**lock **T**ridiagonal|ds|rm|
|SP|**S**calar **P**entadiagonal|ds|rm|
|LU|**L**ower-**U**pper symmetric (Gauss-Seidel)|ds|rm|

### NPB-MZ

|Benchmark|Name derived from|Description|Remarks|
|---------|-----------------|-----------|-------|
|BT-MZ|**B**lock **T**ridiagonal Multi Zone|ds|rm|
|SP-MZ|**S**calar **P**entadiagonal Multi Zone|ds|rm|
|LU-MZ|**L**ower-**U**pper symmetric Multi Zone|ds|rm|

### For unstructured computation, parallel I/O, data movement

|Benchmark|Name derived from|Description|Remarks|
|---------|-----------------|-----------|-------|
|UA|**U**nstructured **A**daptive|ds|rm|
|BT-IO|**B**lock **T**ridiagonal IO|ds|rm|
|DC|**D**ata **C**ube operator|
|DT|**D**ata **T**raffic|

### GridNPB

|Benchmark|Name derived from|
|---------|-----------------|
|ED|**E**mbarassingly **D**istributed|
|HC|**H**elical **C**hain|
|VP|**V**isualization **P**ipeline|
|MB|**M**ixed **B**ag|
