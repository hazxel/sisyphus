# Peripheral Component Interconnect Express (PCIe)

PCIe is a high-speed interface standard used for connecting various components within a computer system. It facilitates communication between the CPU, GPUs, storage devices, network cards, and other peripherals.



# CXL (Compute Express Link)

CXL is an emerging technology that extends memory coherency to the PCIe root complex. It allows host CPUs and device accelerators to access memory with coherency using a load/store interface. 解决主存和设备内存（显存）的割裂问题，实现统一编址，保证缓存一致性，降低延迟。



# RDMA**(Remote Direct Memory Access)**:

RDMA is a technology that allows direct memory access from one computer to another over a network without involving the CPU. 可以理解为分布式存储计算下的 shared disk. (存算一体，计算也可以下推)



# DPDK**(Data Plane Development Kit)**:

DPDK is an open-source framework that provides fast packet processing for network applications.