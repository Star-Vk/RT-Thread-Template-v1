# RT-Thread CubeMX������ֲ
> ��ƪ�����������CubeMX�°汾��ֲRT-Thread nano�汾���ֵ�������н����������ϸ�ĸ���CubeMX�½����������

- CubeMX�汾��6.1.2
- RT-Thread�汾��3.1.5

## һ��������ͨ��STM32����
�ڱ��ڿ�ʼǰ���ȸ��ݹٷ��ĵ����[׼������](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-nano/nano-port-cube/an0041-nano-port-cube?id=%e5%87%86%e5%a4%87%e5%b7%a5%e4%bd%9c)

tips���°�����е���죬������࣬���ﲻ��׸��

![����䶯1](./image/׼������.png)
![����䶯2](./image/׼������fromUrl.png)

һ���յ�STM32������CubeMX������Ҫ�����ý�֤��ʱ��

��������

![��������](./image/����ʱ�Ӿ���.png)

���ȿ���ϵͳʱ��

![����ϵͳʱ��](./image/����ϵͳʱ��.png)

ϵͳʱ������

![ϵͳʱ������](./image/ʱ������.png)

## ������ֲRT-Thread

�����׼��������CubeMX�Ͱ�RT-Thread׼�����ˣ�����ֻ��Ҫ��CubeMX��ӵ�������

![��ѡRT-Thread](./image/��ֲRT-Thread.png)
![����RT-Thread](./image/����RT-Thread.png)

�ص�������ж�
![�ر��ж�](./image/�ر��ж�.png)

���ｨ��Ѵ���6����Ϊ���Դ��ڣ����Ҹ���board.c��Ӧ��
![������](./image/���ô���.png)

## �������ɹ���
��"Project Manager"ѡ��£����ù������ơ�����·�������ɵ�IDE������
![��������](./image/��������.png)
![��������](./image/��������.png)


Ȼ������Ͻǡ�GENERATE CODE����������Ŀ
![�򿪹���](./image/�򿪹���.png)

## �ġ����빤�̡��������
> �°汾�Ĺ��̻���һЩ���룬Ҫ�����ù��̵���ȥ��д�������ڵ��������ϰ汾����ֲ���̲�һ��

### 4.1 ����board.c����
![getchar](./image/getchar.png)

����ԭ�������°汾��RT-Thread��ֲҪ�����б�дboard.c����ļ�
![board](./image/board�ļ�.png)

��ᷢ�֡�#error��������������������ط���Ҫ������д���룬����˵���õ��Դ��ڡ����Դ������ݴ����
����㲻��д�������ֱ�Ӹ������´��뵽����board.c�ļ�

```c
/*
 * Copyright (c) 2006-2019, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2021-05-24                  the first version
 */

#include <rthw.h>
#include <rtthread.h>

#include "main.h"

#if defined(RT_USING_USER_MAIN) && defined(RT_USING_HEAP)
/*
 * Please modify RT_HEAP_SIZE if you enable RT_USING_HEAP
 * the RT_HEAP_SIZE max value = (sram size - ZI size), 1024 means 1024 bytes
 */
#define RT_HEAP_SIZE (15*1024)
static rt_uint8_t rt_heap[RT_HEAP_SIZE];

RT_WEAK void *rt_heap_begin_get(void)
{
    return rt_heap;
}

RT_WEAK void *rt_heap_end_get(void)
{
    return rt_heap + RT_HEAP_SIZE;
}
#endif

void SysTick_Handler(void)
{
    rt_interrupt_enter();
    
    rt_tick_increase();

    rt_interrupt_leave();
}

/**
 * This function will initial your board.
 */
void rt_hw_board_init(void)
{
    extern void SystemClock_Config(void);
    
    HAL_Init();
    SystemClock_Config();
    SystemCoreClockUpdate();
    /* 
     * 1: OS Tick Configuration
     * Enable the hardware timer and call the rt_os_tick_callback function
     * periodically with the frequency RT_TICK_PER_SECOND. 
     */
    HAL_SYSTICK_Config(HAL_RCC_GetHCLKFreq()/RT_TICK_PER_SECOND);

    /* Call components board initial (use INIT_BOARD_EXPORT()) */
#ifdef RT_USING_COMPONENTS_INIT
    rt_components_board_init();
#endif

#if defined(RT_USING_USER_MAIN) && defined(RT_USING_HEAP)
    rt_system_heap_init(rt_heap_begin_get(), rt_heap_end_get());
#endif
}

#ifdef RT_USING_CONSOLE

static UART_HandleTypeDef UartHandle;
static int uart_init(void)
{
    /* TODO: Please modify the UART port number according to your needs */
    /* ���Դ������� */
    UartHandle.Instance = USART6;
    UartHandle.Init.BaudRate = 115200;
    UartHandle.Init.WordLength = UART_WORDLENGTH_8B;
    UartHandle.Init.StopBits = UART_STOPBITS_1;
    UartHandle.Init.Parity = UART_PARITY_NONE;
    UartHandle.Init.Mode = UART_MODE_TX_RX;
    UartHandle.Init.HwFlowCtl = UART_HWCONTROL_NONE;
    UartHandle.Init.OverSampling = UART_OVERSAMPLING_16;

    if (HAL_UART_Init(&UartHandle) != HAL_OK)
    {
        while (1);
    }
    return 0;
}
INIT_BOARD_EXPORT(uart_init);

void rt_hw_console_output(const char *str)
{
    rt_size_t i = 0, size = 0;
    char a = '\r';

    __HAL_UNLOCK(&UartHandle);

    size = rt_strlen(str);

    for (i = 0; i < size; i++)
    {
        if (*(str + i) == '\n')
        {
            HAL_UART_Transmit(&UartHandle, (uint8_t *)&a, 1, 1);
        }
        HAL_UART_Transmit(&UartHandle, (uint8_t *)(str + i), 1, 1);
    }
}
#endif

#ifdef RT_USING_FINSH
char rt_hw_console_getchar(void)
{
    /* Note: the initial value of ch must < 0 */
    int ch = -1;

    if (__HAL_UART_GET_FLAG(&UartHandle, UART_FLAG_RXNE) != RESET)
    {
        ch = UartHandle.Instance->DR & 0xff;
    }
    else
    {
        rt_thread_mdelay(10);
    }
    return ch;
}
#endif
```

### 4.2 finsh_config����
![finsh_config����](./image/finsh_config.png)
����ctrl+F Ȼ���޸ĳɡ�Current Porject��
����rtconfig.h��ֱ���ҵ�rtconfig.h����ļ�
tips�����û�У��ҵ��������룬�Ҽ���
Ȼ������ע��ȡ����
![���dowith_finsh_config](./image/dowith_finsh_config.png)

### 4.3 �߳�ʹ�ñ��������Ҳ����̴߳���������
�����������ͷ�ļ��������뻹�Ǳ����Ҳ����߳���غ���
��4.2 rtconfig.h���ļ���
ȡ��ע��
![���dowith_USEHEAPING](./image/USE_HEAPING.png)
