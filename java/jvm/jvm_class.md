### Class文件结构
class文件是一组以8位字节为基础单位的二进制流，各数据项目严格按照顺序紧凑排列，中间没有分割符。如果数据项占用8位字节以上空间，则按照高位在前方式存储。也就是大端方式。

###### class文件格式

| 类型        | 名称           | 数量  |
| ------------- |-------------| -----|
|u4|magic|1|
|u2|minor_version|1|
|u2|major_version|1|
|u2|constant_pool_count|1|
|cp_info|constant_pool|constant_pool_count - 1|
|u2|access_flags|1|
|u2|this_class|1|
|u2|super_class|1|
|u2|interfaces_count|1|
|u2|interfaces|interfaces_count|
|u2|fields_count|1|
|field_info|fields|fields_count|
|u2|methods_count|1|
|method_info|methods|methods_count|
|u2|attributes_count|1|
|attribute_info|attributes|attributes_count|

