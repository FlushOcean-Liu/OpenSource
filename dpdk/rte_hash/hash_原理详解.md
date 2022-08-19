
# DPDK rte hash实现原理详解(dpdk-stable-18.11.11)

## 1.hash api函数


### 1.1.创建函数

```c
struct rte_hash * 	rte_hash_create (const struct rte_hash_parameters *params)
```
  
创建hash表初始化配置中的entries不能随意配置，创建hash表要根据此长度申请内存空间，要配置为实际单个key的字节数；  

如果配置参数key_len与hash查找时输入key长度不一致，会导致查找失败，这点需要注意，比对key时需要根据key_len进行比对；  

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

**桶buckets结构说明:**
* 1）buckets数量根据用户配置元素数量的2的指数倍，比如输入64，buckets就是64个，输出65，buckets时128；  
* 2）根据key计算sig，根据sig计算 主散列和从散列位置；  
* 3）buckets每个元素存放[ flag（是否使用标志），sig（根据key计算的sig）, key_index(存储在key_store中索引位置)]；  
     共八组；  
* 4）查找删除，比较简单，找到对应桶，根据桶中sig相同的项，找到key_store的index，根据index比对key_store中的key是否相同，
     查找的返回，删除的释放key_store索引到空闲队列中，清空桶中数据；
* 5）比较复杂的时添加hash元素，正常根据主从位置找到空闲桶元素，插入是没有问题，难点在于主从散列位置都没有空闲位置，
     此时冲突处理需要剔除已有元素，剔除元素要重复插入方式去找空闲桶位置，如此类似，直到冲突达到阈值任务hash已满。  






**key_store结构说明：**
* 1）key_store是数组，初始化创建时根据用户配置元素数量，给key_store配置多少个数组元素；
* 2）key_store中数组索引空闲没有被使用的要加入空闲队列中，添加hash元素需要从队列中申请空闲key_store索引，  
     删除hash元素时，需要将使用key_store索引加入空闲队列中；
* 3）key_store数组内容包含key的内存空间，需要将key复制到内存空间中，value的指针，指向value所在内存地址；




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


## 3.hash添加元素，主从都无空闲位置处理方式




