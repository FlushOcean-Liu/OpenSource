# dpdk 内存池使用及实现原理

## 1.mempool api函数

### 1.1.创建函数

struct rte_mempool *
rte_mempool_create(const char *name, unsigned n, unsigned elt_size,  
                   unsigned cache_size, unsigned private_data_size,  
                   rte_mempool_ctor_t *mp_init, void *mp_init_arg,  
                   rte_mempool_obj_cb_t *obj_init, void *obj_init_arg,  
                   int socket_id, unsigned flags);  

* name 内存池名字
* n    内存池分配内存单元数量
* elt_size 每个内存单元大小
* cache_size
