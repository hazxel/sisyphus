## OpenMP

#### pragma

The `#pragma <xxx>` usage is ignored by the preprocessor and passed to the compiler. clang and gcc has their own implementation of it. 

- GCC will replace them by funtions like `GOMP_<xxx>`
- Clang will replace this by functions like `__kmpc_<xxx>`

## MPI

#### MPI_Init

任何MPI程序都应该首先调用该函数（程序中第一个调用的MPI函数必须是这个函数）

#### MPI_Finalize

任何MPI程序结束时，都需要调用该函数

#### MPI_Comm_rank

该函数是获得当前进程的进程标识，如进程0在执行该函数时，可以获得返回值0。可以看出该函数接口有两个参数，前者为进程所在的通信域，后者为返回的进程号。通信域可以理解为给进程分组，比如有0-5这六个进程。可以通过定义通信域，来将比如[0,1,5]这三个进程分为一组，这样就可以针对该组进行“组”操作，比如规约之类的操作。

#### MPI_Comm_size

该函数是获取该通信域内的总进程数，如果通信域为MP_COMM_WORLD，即获取总进程数，使用方法和MPI_COMM_RANK相近。

#### MPI_Send

该函数为发送函数，用于进程间发送消息

传入的参数：

- buf: 数据的起始地址，比如你要传递一个数组A，长度是5，则buf为数组A的首地址。
- count: 长度，从首地址之后count个变量。
- datatype: 变量类型，注意该位置的变量类型是MPI预定义的变量类型，比如需要传递的是C++的int型，则在此处需要传入的参数是MPI_INT，其余同理。
- dest:接收方的进程号，即被传递信息进程的进程号。
- tag: 信息标志，同为整型变量，发送和接收需要tag一致，这将可以区分同一目的地的不同消息。比如进程0给进程1分别发送了数据A和数据B，tag可分别定义成0和1，这样在进程1接收时同样设置tag0和1去接收，避免接收混乱。

#### MPI_Recv

该函数为MPI的接收函数，需要和MPI_SEND成对出现。参数和MPI_Send大体相同，不同的是source这一参数，这一参数标明从哪个进程接收消息。最后多一个用于返回状态信息的参数status。

## TBB

## PPL

## OpenCL

## OpenACC

## SYCL

## Eigen



## pcap (package capture)

### open & close connection

- `pcap_open_live`
- `pcap_open_offline`
- `pcap_close`: close the connection, free related resources



### collect and process packet

- `pcap_dispatch`: return when process more than *cnt*
- `pcap_loop`: 



