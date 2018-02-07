---
title: Send packets defined by ourselves with DPDK
date: 2018-01-25 21:30:23
categories: DPDK
---

[TOC]

# How to send and receive dpdk packets?

> We can do this by flowing three steps.

* Firstly, we should construct a dpdk-type packet struct.
* Then we can code by using dpdk-api to send them.
* At last we shoud also make a receiving program to get these packets.

## Construct a packet

### DPDK net head struct
* As we all know, we should using dpdk struct. For example, if we want to make a UDP packet, we should using Ether head,Ip head,and UDP head structs, which could be found in lib/librte_ehter/ and lib/librte_net/ folders.

```c
/**
 * Ethernet header: Contains the destination address, source address
 * and frame type.
 */
struct ether_hdr {
	struct ether_addr d_addr; /**< Destination address. */
	struct ether_addr s_addr; /**< Source address. */
	uint16_t ether_type;      /**< Frame type. */
} __attribute__((__packed__));

/**
 * IPv4 Header
 */
struct ipv4_hdr {
	uint8_t  version_ihl;		/**< version and header length */
	uint8_t  type_of_service;	/**< type of service */
	uint16_t total_length;		/**< length of packet */
	uint16_t packet_id;		/**< packet ID */
	uint16_t fragment_offset;	/**< fragmentation offset */
	uint8_t  time_to_live;		/**< time to live */
	uint8_t  next_proto_id;		/**< protocol ID */
	uint16_t hdr_checksum;		/**< header checksum */
	uint32_t src_addr;		/**< source address */
	uint32_t dst_addr;		/**< destination address */
} __attribute__((__packed__));

/**
 * UDP Header
 */
struct udp_hdr {
	uint16_t src_port;    /**< UDP source port. */
	uint16_t dst_port;    /**< UDP destination port. */
	uint16_t dgram_len;   /**< UDP datagram length */
	uint16_t dgram_cksum; /**< UDP datagram checksum */
} __attribute__((__packed__));

```
* If we can using these structs correctly, it's easy to get a packet

### Using ether head struct.

* Generate a ether head as an example.

```c
// we can using flowing codes to define a ether packet.
struct ether_hdr *eth_hdr;
struct ether_addr s_addr = {{0x14,0x02,0xEC,0x89,0x8D,0x24}};
struct ether_addr d_addr = {{0x14,0x02,0xEC,0x89,0xED,0x54}};
uint16_t ether_type = 0x0a00; 	
eth_hdr->d_addr = d_addr;
eth_hdr->s_addr = s_addr;
eth_hdr->ether_type = ether_type;

```



##  Make a send program.

### First we should know the mechanism of dpdk pkt sending.

* Initlize the runtime enviroment.
* Apply a mem-pool.
* Initlize the NIC ports.  To get r/s queues, and locate memory to them.
* Define m_buf, and apply for mem from mem-pool.
* Write our pkt into m_buf.
* Move m_buf to tx queue.
* Send the pkt by using dpdk-api.

### Now write this program

```c
/*-
This program is to send a Ether packet to another server.
Author : Hox Zheng
Date : 2017年 12月 31日 星期日 15:21:04 CST
 */
#include <stdint.h>
#include <inttypes.h>
#include <rte_eal.h>
#include <rte_ethdev.h>
#include <rte_cycles.h>
#include <rte_lcore.h>
#include <rte_mbuf.h>
#include <rte_ether.h>
#include <rte_ip.h>
#include <rte_udp.h>
#include <pthread.h>
#include<string.h>

#define RX_RING_SIZE 128
#define TX_RING_SIZE 512
#define NUM_MBUFS 8191
#define MBUF_CACHE_SIZE 250
#define BURST_SIZE 12

//这里用skleten 默认配置
static const struct rte_eth_conf port_conf_default = {
	.rxmode = { .max_rx_pkt_len = ETHER_MAX_LEN }
};


/*
 *这个是简单的端口初始化 
 *我在这里简单的端口0 初始化了一个 接收队列和一个发送队列
 *并且打印了一条被初始化的端口的MAC地址信息
 */
static inline int
port_init(struct rte_mempool *mbuf_pool)
{
	struct rte_eth_conf port_conf = port_conf_default;
	const uint16_t rx_rings = 1, tx_rings = 1;
	int retval;
	uint16_t q;


	/*配置端口0,给他分配一个接收队列和一个发送队列*/
	retval = rte_eth_dev_configure(0, rx_rings, tx_rings, &port_conf);
	if (retval != 0)
		return retval;

	/* Allocate and set up 1 RX queue per Ethernet port. */
	for (q = 0; q < rx_rings; q++) {
		retval = rte_eth_rx_queue_setup(0, q, RX_RING_SIZE,
				rte_eth_dev_socket_id(0), NULL, mbuf_pool);
		if (retval < 0)
			return retval;
	}

	/* Allocate and set up 1 TX queue per Ethernet port. */
	for (q = 0; q < tx_rings; q++) {
		retval = rte_eth_tx_queue_setup(0, q, TX_RING_SIZE,
				rte_eth_dev_socket_id(0), NULL);
		if (retval < 0)
			return retval;
	}

	/* Start the Ethernet port. */
	retval = rte_eth_dev_start(0);
	if (retval < 0)
		return retval;

	return 0;
}

/*
我在main 函数里面 调用了输出化端口0的函数 
然后定义了以太网的头，以及报文。
申请了 m_pool
然后有在mempool里面 分配了两个m_buf
每个buf就是一个包
最后把他俩一次性发出去
 */
int main(int argc, char *argv[])
{
	struct rte_mempool *mbuf_pool;

	/*进行总的初始话*/
	int ret = rte_eal_init(argc, argv);
	if (ret < 0)
		rte_exit(EXIT_FAILURE, "initlize fail!");

	//I don't clearly know this two lines
	argc -= ret;
	argv += ret;

	/* Creates a new mempool in memory to hold the mbufs. */
	//分配内存池
	mbuf_pool = rte_pktmbuf_pool_create("MBUF_POOL", NUM_MBUFS,
		MBUF_CACHE_SIZE, 0, RTE_MBUF_DEFAULT_BUF_SIZE, rte_socket_id());

	//如果创建失败
	if (mbuf_pool == NULL)
		rte_exit(EXIT_FAILURE, "Cannot create mbuf pool\n");

	/* Initialize all ports. */
	//初始话端口设备 顺便给他们分配  队列
		if (port_init(mbuf_pool) != 0)
			rte_exit(EXIT_FAILURE, "Cannot init port %"PRIu8 "\n",
					0);
	//定义报文信息
	struct Message {
		char data[10];
	};
	struct ether_hdr *eth_hdr;
	struct Message obj = {{'H','e','l','l','o','2','0','1','8'}};
	struct Message *msg;
	//自己定义的包头
	struct ether_addr s_addr = {{0x14,0x02,0xEC,0x89,0x8D,0x24}};
	struct ether_addr d_addr = {{0x14,0x02,0xEC,0x89,0xED,0x54}};
	uint16_t ether_type = 0x0a00; 	
	

	//对每个buf ， 给他们添加包
	
	struct rte_mbuf * pkt[BURST_SIZE];
	int i;
	for(i=0;i<BURST_SIZE;i++) {
		pkt[i] = rte_pktmbuf_alloc(mbuf_pool);
		eth_hdr = rte_pktmbuf_mtod(pkt[i],struct ether_hdr*);
		eth_hdr->d_addr = d_addr;
		eth_hdr->s_addr = s_addr;
		eth_hdr->ether_type = ether_type;
		msg = (struct Message*) (rte_pktmbuf_mtod(pkt[i],char*) + sizeof(struct ether_hdr));
		*msg = obj;
		int pkt_size = sizeof(struct Message) + sizeof(struct ether_hdr);
		pkt[i]->data_len = pkt_size;
		pkt[i]->pkt_len = pkt_size;
	}

	uint16_t nb_tx = rte_eth_tx_burst(0,0,pkt,BURST_SIZE);
	printf("发送成功%d个包\n",nb_tx);
	//发送完成，答应发送了多少个
	
	for(i=0;i<BURST_SIZE;i++)
		rte_pktmbuf_free(pkt[i]);

	return 0;
}

```



## Make a receive program

### At the same case, we should know the machanism of receiving.

* Initlize the runtime enviroment.
* Apply a mem-pool.
* Initlize the NIC ports.  To get r/s queues, and locate memory to them.
* Define m_buf, and apply for mem from mem-pool.
* Get pkt from NIC port to m_buf.
* Analyze the m_buf, and get the pkt;
* Print message in the pkt.

### Now we get this program

```c
/*
接收一个包并且打印出信息
Author : Hox Zheng
Date : 2017年 12月 31日 星期日 15:21:04 CST
 */
#include <stdint.h>
#include <inttypes.h>
#include <rte_eal.h>
#include <rte_ethdev.h>
#include <rte_cycles.h>
#include <rte_lcore.h>
#include <rte_mbuf.h>
#include <rte_ether.h>
#include <rte_ip.h>
#include <rte_udp.h>
#include <pthread.h>

#define RX_RING_SIZE 128
#define TX_RING_SIZE 512

#define NUM_MBUFS 8191
#define MBUF_CACHE_SIZE 250
#define BURST_SIZE 32

//这里用skleten 默认配置
static const struct rte_eth_conf port_conf_default = {
	.rxmode = { .max_rx_pkt_len = ETHER_MAX_LEN }
};


/*
 *这个是简单的端口初始化 
 *我在这里简单的端口0 初始化了一个 接收队列和一个发送队列
 */
static inline int
port_init(struct rte_mempool *mbuf_pool)
{
	struct rte_eth_conf port_conf = port_conf_default;
	const uint16_t rx_rings = 1, tx_rings = 1;
	int retval;
	uint16_t q;


	/*配置端口0,给他分配一个接收队列和一个发送队列*/
	retval = rte_eth_dev_configure(0, rx_rings, tx_rings, &port_conf);
	if (retval != 0)
		return retval;

	/* Allocate and set up 1 RX queue per Ethernet port. */
	for (q = 0; q < rx_rings; q++) {
		retval = rte_eth_rx_queue_setup(0, q, RX_RING_SIZE,
				rte_eth_dev_socket_id(0), NULL, mbuf_pool);
		if (retval < 0)
			return retval;
	}

	/* Allocate and set up 1 TX queue per Ethernet port. */
	for (q = 0; q < tx_rings; q++) {
		retval = rte_eth_tx_queue_setup(0, q, TX_RING_SIZE,
				rte_eth_dev_socket_id(0), NULL);
		if (retval < 0)
			return retval;
	}

	/* Start the Ethernet port. */
	retval = rte_eth_dev_start(0);
	if (retval < 0)
		return retval;

	return 0;
}

/*
我在main 函数里面 调用了输出化端口0的函数 
申请了 m_pool
定义m_buf用来取接受队列中的包
 */
int main(int argc, char *argv[])
{
	struct rte_mempool *mbuf_pool;

	/*进行总的初始话*/
	int ret = rte_eal_init(argc, argv);
	if (ret < 0)
		rte_exit(EXIT_FAILURE, "initlize fail!");

	//I don't clearly know this two lines
	argc -= ret;
	argv += ret;

	/* Creates a new mempool in memory to hold the mbufs. */
	//分配内存池
	mbuf_pool = rte_pktmbuf_pool_create("MBUF_POOL", NUM_MBUFS,
		MBUF_CACHE_SIZE, 0, RTE_MBUF_DEFAULT_BUF_SIZE, rte_socket_id());

	//如果创建失败
	if (mbuf_pool == NULL)
		rte_exit(EXIT_FAILURE, "Cannot create mbuf pool\n");

	/* Initialize all ports. */
	//初始话端口设备 顺便给他们分配  队列
		if (port_init(mbuf_pool) != 0)
			rte_exit(EXIT_FAILURE, "Cannot init port %"PRIu8 "\n",
					0);
	//定义m_buf，用来接受接受对列中的包
	printf("初始话就绪，正在持续接受数据包.......\n");

	for(;;)
	{

		struct rte_mbuf * pkt[BURST_SIZE];
		int i;
		//定义m_buf 并且分配内存
		for(i=0;i<BURST_SIZE;i++) {
			pkt[i] = rte_pktmbuf_alloc(mbuf_pool);
		}
	
		//从接受队列中取出包
		uint16_t nb_rx = rte_eth_rx_burst(0, 0,pkt,BURST_SIZE);
		if(nb_rx == 0)
		{
			//如果没有接受到就跳过
			continue;
		}
		char * msg;
		struct ether_hdr * eth_hdr;
		//打印信息
		for(i=0;i<nb_rx;i++)
		{
			eth_hdr = rte_pktmbuf_mtod(pkt[i],struct ether_hdr*);
			//打印接受到的包的地址信息
			eth_hdr = rte_pktmbuf_mtod(pkt[i],struct ether_hdr*);
			printf("收到包 来自MAC: %02" PRIx8 " %02" PRIx8 " %02" PRIx8
				   " %02" PRIx8 " %02" PRIx8 " %02" PRIx8 " : ",
				eth_hdr->s_addr.addr_bytes[0],eth_hdr->s_addr.addr_bytes[1],
				eth_hdr->s_addr.addr_bytes[2],eth_hdr->s_addr.addr_bytes[3],
				eth_hdr->s_addr.addr_bytes[4],eth_hdr->s_addr.addr_bytes[5]);
			msg = ((rte_pktmbuf_mtod(pkt[i],char*)) + sizeof(struct ether_hdr));
			int j;
			for(j=0;j<10;j++)
				printf("%c",msg[j]);
			printf("\n");
			rte_pktmbuf_free(pkt[0]);
		}

	}
	
	return 0;
}

```

## Conclusion

>  I am studing dpdk, and this is just a example program by using eth head struct. I think it's not a difficult thing to do the same effect on IP/UDP pkt head struct.
>
> Actually, the idea of this program is to get a general understanding of dpdk send&receive machanism.
>
> Let's all, bye~

