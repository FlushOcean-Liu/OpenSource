# dpdk hash 函数使用说明

```c

#define MAXELEMENT_TEST 5

struct rte_test_value{
	int key;
	int value;
};

void dpdk_rte_hash_example(void)
{
    int ret;
    int i;
    struct rte_hash_parameters hash_param={
        .name               = "rte_hash_example",
        .entries            = 1000,
        .key_len            = sizeof(int),
        .hash_func          = rte_jhash,
        .hash_func_init_val = 0,
        .scoket_id          = 0,
        
    };

    struct rte_hash *hash_table=rte_hash_create(&hash_param);
    if(!hash_table){
        printf("create hash table failed\n");
        exit(-1);
    }

    struct rte_test_value  *tval[MAX_ELEMENT_TEST];
    for(i=0;i<MAX_ELEMENT_TEST;i++){
        tval[i]=(struct rte_test_value *)malloc(sizeof(struct rte_test_value));
        if(!tval[i]){
            printf("malloc failed \n");
            exit(-1);
        }
        tval[i]->key = i;
        tval[i]->value=i*2;

        ret=rte_hash_add_key_data(hash_table, &tval[i]->key, &tval[i]->value);
        if(ret<0){
            printf("add hash value failed \n");
        }
    }
    
    for(i=0;i<MAX_ELEMENT_TEST;i++){
        int *p=NULL;
        ret=rte_hash_lookup_data(hash_table, &i, (void **)&p);
        if(ret>=0 && p){
            printf("get hash value p=%d\n",*p);
        }else{
            printf("lookup failed ret=%d\n",ret);
        }
    }

}

```

陷阱一：
经常有小伙伴在创建hash表,不注意配置key_len与实际查询插入key 长度保持一致，导致hash表查询失败：
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




