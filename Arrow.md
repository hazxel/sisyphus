# Apache Arrow
Arrow是Apache基金会近几年最活跃的项目之一，它基于内存列式格式衍生出了完善的内存计算生态，是当前内存列式数据格式事实上的标准。Arrow的内存模型可以帮助编译器自动地实现向量化，且在传输时没有序列化/反序列化成本，实现了CPU和IO的效率提升。众多我们熟悉的数据处理系统如Spark，Impala，Kudu，Clickhouse，Pandas等项目中都有Arrow的身影。

Apache Arrow定义了一个各语言通用（language-agnostic）的列式内存格式（columnar memory format），支持扁平（flat）和多层级的（hierarchical）数据。简而言之，Arrow定义了各种数据类型（如简单的int、float和复合的struct, union，list, map等）的数组在内存中如何用一段连续内存表示，且这种表示是各语言通用的。

与其他大多数的Apache项目不同，Arrow项目本身不是一个组件而是一个标准化协议，类似于Protobuf和Thrift，Arrow本身定义了一个跨语言的表示数据的方法，同时为各种主流语言提供了原生的处理Arrow数据的实现，当前支持的语言包括：C++、Python、Rust、Java、Go等。除了官方实现外，开发者还可以根据协议编写非官方的实现，比如Rust的arrow2。Arrow本身不像是“轮子”，更像是用来造轮子的“轮毂”，在底层支撑各个数据处理项目。



## 列存

列存（columnar，column based）是与行存（row based）相对应的一种存储方式，是将同一列的数据存储在一起。
优势：

- 更少的IO：可以只读取需要的列。
- 更好的压缩效果：同类型的数据存储连续存储对某些压缩机制更友好，比如Run Length Encoding，Delta Encoding，Dictionary Encoding等。
- 更高的计算效率：列式存储对cpu缓存和向量化更友好

劣势：
- 单行读取，插入，更新和删除操作都更慢。涉及到整行的操作，每行数据都要访问 m 次

由此可见，列存格式的优势在于行数多，列数少的只读（分析）操作，数据库领域称这种场景为OLAP，A代表Analysis，与之相对的，行存的优势场景在OLTP，T代表Transaction。



### API

###### read parquet

1. parquet::arrow::FileReader
2. parquet::arrow::OpenFile
3. parquet::arrow::FileReader::GetRecordBatchReader >> parquet::arrow::RowGroupRecordBatchReader
4. arrow::RecordBatchReader::ReadAll 

arrow工程中，parquet::arrow::FileReaderImpl 继承自parquet::arrow::FileReader，持有 parquet::ParquetFileReader：

> parquet::arrow::OpenFile 调用 FileReaderBuilder::Build 调用 FileReader::Make 来创建并返回 FileReaderImpl 

> FileReader::GetRecordBatchReader 写的非常奇怪，实现了两种重载，一个参数是`shared_ptr<RecordBatchReader>*`，另一个参数是 `unique_ptr<RecordBatchReader>*`，反正不会跑到这里先不管了。。

parquet-cpp工程中，parquet::arrow::FileReader 持有 FileReader::Impl 持有 parquet::ParquetFileReader （旧实现）





### Zero-copy memory sharing

##### C Interface

```c
struct ArrowSchema { // Array type(schema) description
 const char* format; // can be nested format struct/map, or int/string/date/... 
 const char* name;
 const char* metadata;
 int64_t flags;
 int64_t n_children;
 struct ArrowSchema** children; // array of children
 struct ArrowSchema* dictionary;
 void (*release)(struct ArrowSchema*); // Release callback (dtor), 为 NULL 代表已释放
 void* private_data; 					// Opaque producer-specific data
};

struct ArrowArray { // Array data description
 int64_t length;   // row number (length of array)
 int64_t null_count; // empty count (for optimization)
 int64_t offset;
 int64_t n_buffers;
 int64_t n_children;
 const void** buffers;			// array of buffers
 struct ArrowArray** children;	// array of children
 struct ArrowArray* dictionary;
 void (*release)(struct ArrowArray*); 	// Release callback (dtor), 为 NULL 代表已释放
 void* private_data; 					// Opaque producer-specific data
};
```



##### C++ Interface

###### Buffer

Object containing a pointer to a piece of contiguous memory with a particular size. The Buffer base class does not own its memory, but **subclasses often own their memory**(`PoolBuffer`, etc.), and manage the memory in RAII way.

```c++
class ARROW_EXPORT Buffer {
protected:
 bool is_mutable_;
 bool is_cpu_;
 const uint8_t* data_;
 int64_t size_;
 int64_t capacity_;
 DeviceAllocationType device_type_;	// CPU/CUDA/... depends on memory_manager_
 std::shared_ptr<Buffer> parent_; 		// null by default, but may be set
private:
 // private so that subclasses are forced to call SetMemoryManager()
 // default to CPUMemoryManager which uses default_system_memory_pool (C library malloc)
 std::shared_ptr<MemoryManager> memory_manager_; 
}
```

###### ArrayData

This data structure is a self-contained representation of the memory and metadata inside an Arrow array data structure

```c++
struct ARROW_EXPORT ArrayData {
 std::shared_ptr<DataType> type;	// similar to C interface's "format"
 int64_t length = 0;				// row count
 mutable std::atomic<int64_t> null_count{0};
 // The logical start point into the physical buffers (in values, not bytes).
 // Note that, for child data, this must be *added* to the child data's own offset.
 int64_t offset = 0;
 std::vector<std::shared_ptr<Buffer>> buffers; // first buffer holds null bitmap (in LSB numbering)
 std::vector<std::shared_ptr<ArrayData>> child_data;

 // The dictionary for this Array, if any. Only used for dictionary type
 std::shared_ptr<ArrayData> dictionary;
}
```

###### Array (abstraction for "column")

Immutable data array with some logical type and some length. Any memory is owned by the underlying Buffer instance.

```c++
class ARROW_EXPORT Array {
protected:
 std::shared_ptr<ArrayData> data_;
  // required if data_.null_count > 0, points to the first buffer
 const uint8_t* null_bitmap_data_ = NULLPTR; 
}
```

###### RecordBatch

Use RecordBatch to store both schema and array, the base class:

```c++
class ARROW_EXPORT RecordBatch {
 public:
 static std::shared_ptr<RecordBatch> Make(
   std::shared_ptr<Schema> schema,
   int64_t num_rows,
   std::vector<std::shared_ptr<Array>> columns);
   
 static std::shared_ptr<RecordBatch> Make(
   std::shared_ptr<Schema> schema, int64_t num_rows,
   std::vector<std::shared_ptr<ArrayData>> columns);
   
 protected:
 RecordBatch(const std::shared_ptr<Schema>& schema, int64_t num_rows);

 std::shared_ptr<Schema> schema_;
 int64_t num_rows_;
};
```
a default, basic, non-lazy in-memory record batch impl:

```c++
class SimpleRecordBatch : public RecordBatch {
 public:
 SimpleRecordBatch(std::shared_ptr<Schema> schema, int64_t num_rows,
  				std::vector<std::shared_ptr<Array>> columns)
   : RecordBatch(std::move(schema), num_rows), boxed_columns_(std::move(columns));

 SimpleRecordBatch(const std::shared_ptr<Schema>& schema, int64_t num_rows,
          std::vector<std::shared_ptr<ArrayData>> columns)
   : RecordBatch(std::move(schema), num_rows), columns_(std::move(columns));

 private:
 std::vector<std::shared_ptr<ArrayData>> columns_;

 // Caching boxed array data, lazy boxing
 mutable std::vector<std::shared_ptr<Array>> boxed_columns_;
};
```