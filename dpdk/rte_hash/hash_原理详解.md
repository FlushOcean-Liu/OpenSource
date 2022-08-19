
# DPDK rte hash实现原理详解(dpdk-stable-18.11.11)

## 1.hash api函数


### 1.1.创建函数

```c
struct rte_hash * 	rte_hash_create (const struct rte_hash_parameters *params)
```
  
创建hash表初始化配置中的entries不能随意配置，创建hash表要根据此长度申请内存空间，要配置为实际单个key的字节数；
```c
struct rte_hash_parameters hash_param={
        .name               = "rte_hash_example",
        .entries            = 1000,
        .key_len            = sizeof(int),
        .hash_func          = rte_jhash,
        .hash_func_init_val = 0,
        .scoket_id          = 0,
};
```


### 1.2.查找  

```c
int rte_hash_lookup_data(const struct rte_hash *h, const void *key, void **data);  

int rte_hash_lookup_with_hash_data(const struct rte_hash *h, const void *key,  
					hash_sig_t sig, void **data);

int32_t rte_hash_lookup(const struct rte_hash *h, const void *key);  

int32_t rte_hash_lookup_with_hash(const struct rte_hash *h,  
				const void *key, hash_sig_t sig);  


int rte_hash_lookup_bulk_data(const struct rte_hash *h, const void **keys,  
		      uint32_t num_keys, uint64_t *hit_mask, void *data[]);  

int rte_hash_lookup_bulk(const struct rte_hash *h, const void **keys,  
		      uint32_t num_keys, int32_t *positions);  

int32_trte_hash_iterate(const struct rte_hash *h, const void **key, void **data, uint32_t *next);  

``` 



### 1.3.添加

``` c
int rte_hash_add_key_data(const struct rte_hash *h, const void *key, void *data);  


int32_t rte_hash_add_key_with_hash_data(const struct rte_hash *h, const void *key,  
						hash_sig_t sig, void *data);  


int32_t rte_hash_add_key(const struct rte_hash *h, const void *key);  


int32_t rte_hash_add_key_with_hash(const struct rte_hash *h, const void *key, hash_sig_t sig);  

```

### 1.4.删除

```c
int32_t rte_hash_del_key (const struct rte_hash *h, const void *key);  
 
int32_t rte_hash_del_key_with_hash (const struct rte_hash *h, const void *key, hash_sig_t sig);  

```

## 2.原理简介  

要理解hash组织方式，先确定两个数组，桶buckets和key_store数组的管理方式；

桶buckets结构图：






key_store结构图：





**hash表查找步骤：** 
* 1）根据key计算sig，根据sig计算出两个散列值主散列，从散列；  
* 2）根据主散列定位buckets索引位置，查找内容中sig是否存在，不存在则查找从散列位置；  
* 3）步骤2）如果找到sig，则根据key_index定位到key_store内容，比对key值；  
* 4）根据步骤3）比对key结果确定是否查找到对应元素。  


**hash表删除步骤：**  
* 1）删除前要先查找，按照查找步骤，查找到对应key；  
* 2）删除操作要把key_store的index索引入空闲队列；  
* 3）buckets中对应位置sig，flag，index置空。  

**添加处理冲突较复杂，这里先描述正常在主从散列找到空闲位置进行插入情况：**  
* 1）先在主散列桶位置找到空闲位置，若无空闲，则取从散列桶查找空闲位置；  
* 2）若步骤1）找到空闲位置，则从key_store空闲队列获取空闲index，桶位置sig，index，flag赋值；  
* 3）根据步骤2）中的index，找到key_store元素，复制key内容到key_store的key空间，value指针指向对应value内存地址；  


