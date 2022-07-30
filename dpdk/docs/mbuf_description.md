# mbuf详解

## headroom和tailroom作用
网卡收包后需要对报文进行解析，这个解析一般时一次性的，后续报文继续流动，在其他模块、进程中  
存在使用前面预先解析的结果，这时如果重新解析会导致重复处理，延迟收包速度。

此时mbuf的headroom与tailroom就派上用场，每个mubf的headroom大小与tailroom的大小在创建的时候就已经确定，
后续可以将解析报文关键信息存储到headroom或tailroom中，其他模块、进程在获取到mubf后，通过增加相应的偏移就能
获取到已经解析过字段值。

## 巨帧数据组装
自定义mbuf数据发送时，如果组装巨帧，那么需要发送巨帧按照mbuf链式组合，

第一个mbuf要包含数据头部信息如以太层，ip层，传输层，mbuf链后续单元只需承载数据部分即可，无需头部信息和headrom。

示例：
```c

#define CONFIG_MBUF_SIZE   2048

/* 构造巨帧数据发生 */
int send_mbuf_data(uint8_t *data, int data_len, int ring_id)
{
    int i;
    
    int num_seg      = 0;
    int copy_len     = 0;
    int left_len     = 0;
    int data_offset  = 0;
    int copy_seg_len = 0;
    struct rte_mbuf *head = NULL;
    struct rte_mbuf *mbuf = NULL;
    struct rte_mbuf *pre_mbuf = NULL;

    
    int mdata_size=CONFIG_MBUF_SIZE - sizeof(struct ether_hdr);
    if(data_len<=mdata_size){
        num_seg  = 1;
        copy_len = data_len;
    }else{
        copy_len     = mdata_size;
        copy_seg_len = CONFIG_MBUF_SIZE+RTE_PKTMBUF_HEADROOM;
        num_seg=(data_len-mdata_size)/(CONFIG_MBUF_SIZE+RTE_PKTMBUF_HEADROOM);
        if((data_len-mdata_size)%(CONFIG_MBUF_SIZE+RTE_PKTMBUF_HEADROOM)!=0){
            num_seg++;  /* last mbuf */
        }
        num_seg++; /* first mbuf */
    }

    /* head */
    head=rte_mbuf_raw_alloc(tx_pktmbuf_poo);   /* tx_pktmbuf_pool是初始化mbuf内存池 */
    if( NULL == head ){
        return 0;
    }
    head->data_len = sizeof(struct ether_hdr) + copy_len;
    head->nb_segs  = num_seg;
    head->vlan_tci = 0;
    head->vlan_tci_outer = 0;
    
    head->pkt_len  = sizeof(struct ether_hdr) + copy_len;
    head->data_off = RTE_PKTMBUF_HEADROOM;
    
    _tx_updating_mac(head); /* 构造以太层数据，如果需要构造ip，传输层类似，本示例只有以太层，以太层后面就是data数据*/
    memcpy((void *)((uint8_t *)head->buf_addr+head->data_off+sizeof(struct ether_hdr)), 
                    (void*)data, copy_len);

    data_offset = copy_len;
    left_len    = data_len-copy_len;

    pre_mbuf=head;
    /* middle segment mbuf, first is head, start from second */
    for( i = 1; i < num_seg; i++ ){
        mbuf = rte_mbuf_raw_alloc(tx_pktmbuf_pool[0]);
        if( NULL == mbuf ){
            break;
        }
        
        mbuf->nb_segs  = num_seg-i;
        mbuf->data_off = 0;
        if(left_len<copy_seg_len){
            memcpy((void *)mbuf->buf_addr, (void*)&data[data_offset], left_len);
            mbuf->data_len = left_len;
        }else{
            memcpy((void *)mbuf->buf_addr, (void*)&data[data_offset], copy_seg_len);
            mbuf->data_len = copy_seg_len;
            data_offset += copy_seg_len;
            left_len    -= copy_seg_len;
        }

        head->pkt_len  += mbuf->data_len;
        pre_mbuf->next =  mbuf;
        pre_mbuf       =  mbuf;
    }

    _tx_enqueue_mbuf(head, ring_id);

    return 1;
}


```
