
# DPDK rte hash实现原理详解(dpdk-stable-18.11.11)

## 1.hash api函数


### 1.1.创建函数

```c
struct rte_hash * 	rte_hash_create (const struct rte_hash_parameters *params)
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






