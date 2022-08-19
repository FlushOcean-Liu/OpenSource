
# DPDK rte hash实现原理详解(dpdk-stable-18.11.11)

## hash api函数

```c
struct rte_hash * 	rte_hash_create (const struct rte_hash_parameters *params)
```

### 1.创建函数

### 2.查找  

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



### 3.添加

``` c
int rte_hash_add_key_data(const struct rte_hash *h, const void *key, void *data);  


int32_t rte_hash_add_key_with_hash_data(const struct rte_hash *h, const void *key,  
						hash_sig_t sig, void *data);  


int32_t rte_hash_add_key(const struct rte_hash *h, const void *key);  


int32_t rte_hash_add_key_with_hash(const struct rte_hash *h, const void *key, hash_sig_t sig);  

```

### 3.删除

```c
int32_t rte_hash_del_key (const struct rte_hash *h, const void *key);  
 
int32_t rte_hash_del_key_with_hash (const struct rte_hash *h, const void *key, hash_sig_t sig);  

```

## 原理简介  

1）hash表中桶采用连续数组保存，每个桶有8个元素，根据key计算出两个散列位置（主，从）；

2）查找和删除先找key主散列位置，不存在在查找从散列位置；

3）插入较复杂，先判断主key散列位置是否已被占用，如果被占用，则定位到key从散列位置；  
  从散列为也被占用，则剔除占用者，占用着重复上述插入步骤，直到不在冲突为止，冲突次  
  数达到阈值，判断hash表满。



说明：
rte_hash 在创建时，会根据参数设置多个元素，内存会创建多少个key空间，

申请空间大小=(sizeof(struct rte_hash_key) + params->key_len)* (params->entries + 1)

最后插入hash表的key会copy到初始化申请内存中，从这方面说明，创建hash表时就已经要设置hash  
表最大容量，并且为每个可以申请好空间，使用hash表时要注意key的长度尽量小。
