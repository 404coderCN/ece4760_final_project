---
layout: post
title: Glove-controlled robot hand
subtitle: an ECE4760 final project
author: Jerry Jin
categories: jekyll
banner:
  image: https://404codercn.github.io/ece4760_final_project//assets/images/banners/demo_setup.jpg
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: C code
sidebar: []
---

## Transmitter code

```c
#include <stdio.h>
#include <math.h>
#include "pico/stdlib.h"

#include "hardware/sync.h"
#include "hardware/spi.h"

// Include standard libraries
#include <stdlib.h>
#include <string.h>

// Include PICO libraries
#include "pico/stdlib.h"
#include "pico/multicore.h"

// Include hardware libraries
#include "hardware/pio.h"
#include "hardware/dma.h"
#include "hardware/adc.h"
#include "hardware/irq.h"

// Include custom library
#include "pt_cornell_rp2040_v1.h"
#include "nrf24_driver.h"
#include "pico/stdlib.h"

#include <tusb.h> // TinyUSB tud_cdc_connected()

// Get data from flex sensor 
#define FLEX_INDEX            26
#define ADC_INDEX_CHAN        0
#define FLEX_MIDDLE           27
#define ADC_MIDDLE_CHAN       1

#define FLEX_RING             28
#define ADC_RING_CHAN         2

uint8_t flex_out_index;  //flex sensor output
uint8_t flex_out_middle;  //flex sensor output
uint8_t flex_out_ring;  //flex sensor output
fn_status_t checks = 0;

//////////////////////////////////Transmiter //////////////////////////////////

// GPIO pin numbers
pin_manager_t my_pins = { 
    .sck = 2, 
    .copi = 3, 
    .cipo = 4, 
    .csn = 5, 
    .ce = 6
};

/**
* nrf_manager_t can be passed to the nrf_client_t
* initialise function, to specify the NRF24L01 
* configuration. If NULL is passed to the initialise 
* function, then the default configuration will be used.
*/
nrf_manager_t my_config = {
    // RF Channel 
    .channel = 120,

    // AW_3_BYTES, AW_4_BYTES, AW_5_BYTES
    .address_width = AW_5_BYTES,

    // dynamic payloads: DYNPD_ENABLE, DYNPD_DISABLE
    .dyn_payloads = DYNPD_ENABLE,

    // data rate: RF_DR_250KBPS, RF_DR_1MBPS, RF_DR_2MBPS
    .data_rate = RF_DR_2MBPS,

    // RF_PWR_NEG_18DBM, RF_PWR_NEG_12DBM, RF_PWR_NEG_6DBM, RF_PWR_0DBM
    .power = RF_PWR_NEG_12DBM,

    // retransmission count: ARC_NONE...ARC_15RT
    .retr_count = ARC_10RT,

    // retransmission delay: ARD_250US, ARD_500US, ARD_750US, ARD_1000US
    .retr_delay = ARD_500US 
};

// SPI baudrate
uint32_t my_baudrate = 10000000;

nrf_client_t my_nrf;

// result of packet transmission
fn_status_t success = 0;

//////////////////////////////////Transmiter ////////////////////////////////// 

// User input thread. User can change draw speed
static PT_THREAD (protothread_transmitter(struct pt *pt))
{
    PT_BEGIN(pt) ;
    while(1) {
        //////////////////////////////////Transmiter //////////////////////////////////
        my_nrf.tx_destination((uint8_t[]){0x37,0x37,0x37,0x37,0x37});          //data pipe 0
        success = my_nrf.send_packet(&flex_out_index, sizeof(flex_out_index));
        if (success)
        {               
            printf("\nPacket sent: flex_out_index: %d\n", flex_out_index);
        }

        my_nrf.tx_destination((uint8_t[]){0xC7,0xC7,0xC7,0xC7,0xC7});           //data pipe 1
        success = my_nrf.send_packet(&flex_out_middle, sizeof(flex_out_middle));
        if (success)
        {               
            printf("\nPacket sent: flex_out_middle: %d\n", flex_out_middle);
        }

        my_nrf.tx_destination((uint8_t[]){0xCA,0xC7,0xC7,0xC7,0xC7});           //data pipe 2
        success = my_nrf.send_packet(&flex_out_ring, sizeof(flex_out_ring));
        if (success)
        {               
            printf("\nPacket sent: flex_out_ring: %d\n", flex_out_ring);
        }

        //////////////////////////////////Transmiter //////////////////////////////////

    }
    PT_END(pt) ;
}

// User input thread. User can change draw speed
static PT_THREAD (protothread_adcs(struct pt *pt))
{
    PT_BEGIN(pt) ;
    while(1) {
        adc_select_input(ADC_INDEX_CHAN) ;
        flex_out_index = (uint8_t) adc_read();
        // printf("Index Flex sensor data is %d \n: ", flex_out_index); 

        adc_select_input(ADC_MIDDLE_CHAN) ;
        flex_out_middle = (uint8_t) adc_read();
        // printf("Middle Flex sensor data is %d \n: ", flex_out_middle); 

        adc_select_input(ADC_RING_CHAN) ;
        flex_out_ring = (uint8_t) adc_read();
        printf("Ring Flex sensor data is %d \n: ", flex_out_ring); 
    }
    PT_END(pt) ;
}

// Entry point for core 1
void core1_entry() {
    pt_add_thread(protothread_adcs) ;
    pt_schedule_start ;
}

int main() {
    // Initialize stdio/uart
    stdio_init_all();        

    //wait until the CDC ACM (serial port emulation) is connected
    while (!tud_cdc_connected()) 
    {
        sleep_ms(10);
    }

    //////////////////////////////////Transmiter //////////////////////////////////

    // initialise my_nrf
    nrf_driver_create_client(&my_nrf);

    // configure GPIO pins and SPI
    my_nrf.configure(&my_pins, my_baudrate);

    // not using default configuration (my_nrf.initialise(NULL)) 
    my_nrf.initialise(&my_config);

    my_nrf.dyn_payloads_enable();

    // set to Standby-I Mode
    my_nrf.standby_mode();

    ///////////////////////////////////////////////////////////////////////////////
    // ============================== ADC CONFIGURATION ==========================
    //////////////////////////////////////////////////////////////////////////////

    adc_gpio_init(FLEX_INDEX);
    adc_gpio_init(FLEX_MIDDLE);
    adc_gpio_init(FLEX_RING);
    adc_init() ;
    
    ////////////////////////////////////////////////////////////////////////
    ///////////////////////////// ROCK AND ROLL ////////////////////////////
    ////////////////////////////////////////////////////////////////////////
    // start core 1 
    multicore_reset_core1();
    multicore_launch_core1(core1_entry);

    // start core 0
    pt_add_thread(protothread_transmitter) ;
    pt_schedule_start ;
    
}
```


## Receiver
```c
#include <stdio.h>
#include <math.h>
#include "pico/stdlib.h"

#include "hardware/sync.h"
#include "hardware/spi.h"

// Include standard libraries
#include <stdlib.h>
#include <string.h>

// Include PICO libraries
#include "pico/stdlib.h"
#include "pico/multicore.h"

// Include hardware libraries
#include "hardware/pio.h"
#include "hardware/dma.h"
#include "hardware/adc.h"
#include "hardware/irq.h"

// Include custom library
#include "pt_cornell_rp2040_v1.h"
#include "nrf24_driver.h"
#include "pico/stdlib.h"

#include <tusb.h> // TinyUSB tud_cdc_connected()

#include "hardware/pwm.h"

uint8_t in_data_index = 0;
uint8_t in_data_middle = 0;
uint8_t in_data_ring = 0;

///////////////////////////////PWM Servo /////////////////////////////////////////

// PWM wrap value and clock divide value
// For a CPU rate of 125 MHz, this gives
// a PWM frequency of 1 kHz.
#define WRAPVAL  20000.0f
#define CLKDIV   125.0f

// Variable to hold PWM slice number
uint8_t slice_num_index ;
uint8_t slice_num_middle;
uint8_t slice_num_ring;
uint8_t slice_num_little;
uint8_t slice_num_thumb;

///////////////////////////////PWM Servo /////////////////////////////////////////

//////////////////////////////////Receiver //////////////////////////////////

uint8_t pipe_number = 0; 

// GPIO pin numbers
pin_manager_t my_pins = { 
    .sck = 2, 
    .copi = 3, 
    .cipo = 4, 
    .csn = 5, 
    .ce = 6
};

  /**
   * nrf_manager_t can be passed to the nrf_client_t
   * initialise function, to specify the NRF24L01 
   * configuration. If NULL is passed to the initialise 
   * function, then the default configuration will be used.
   */
  nrf_manager_t my_config = {
    // RF Channel 
    .channel = 120,

    // AW_3_BYTES, AW_4_BYTES, AW_5_BYTES
    .address_width = AW_5_BYTES,

    // dynamic payloads: DYNPD_ENABLE, DYNPD_DISABLE
    .dyn_payloads = DYNPD_ENABLE,

    // data rate: RF_DR_250KBPS, RF_DR_1MBPS, RF_DR_2MBPS
    .data_rate = RF_DR_2MBPS,

    // RF_PWR_NEG_18DBM, RF_PWR_NEG_12DBM, RF_PWR_NEG_6DBM, RF_PWR_0DBM
    .power = RF_PWR_NEG_12DBM,

    // retransmission count: ARC_NONE...ARC_15RT
    .retr_count = ARC_10RT,

    // retransmission delay: ARD_250US, ARD_500US, ARD_750US, ARD_1000US
    .retr_delay = ARD_500US 
  };

// SPI baudrate
uint32_t my_baudrate = 10000000;

nrf_client_t my_nrf;

// result of packet transmission
fn_status_t success = 0;

//////////////////////////////////Transmiter ////////////////////////////////// 

// User input thread. User can change draw speed
static PT_THREAD (protothread_serial(struct pt *pt))
{
    PT_BEGIN(pt) ;
    while(1) {
        //////////////////////////////////Transmiter //////////////////////////////////
        // read payload
        if  (my_nrf.is_packet(&pipe_number)){
            switch (pipe_number)
            {
                case DATA_PIPE_0:
                    my_nrf.read_packet(&in_data_index, sizeof(in_data_index));
                    // receiving a one byte uint8_t payload on DATA_PIPE_0
                    printf("\nPacket received:- Payload (%d) on data pipe (%d)\n", in_data_index, pipe_number);
                break; 

                case DATA_PIPE_1:
                    // read payload
                    my_nrf.read_packet(&in_data_middle, sizeof(in_data_middle));

                    // receiving a five byte string payload on DATA_PIPE_1
                    printf("\nPacket received:- Payload (%d) on data pipe (%d)\n", in_data_middle, pipe_number);
                break;

                case DATA_PIPE_2:
                    // read payload
                    my_nrf.read_packet(&in_data_ring, sizeof(in_data_ring));

                    // receiving a five byte string payload on DATA_PIPE_1
                    printf("\nPacket received:- Payload (%d) on data pipe (%d)\n", in_data_ring, pipe_number);
                break;
            }
        }

        //////////////////////////////////Transmiter //////////////////////////////////
        
    }
    PT_END(pt) ;
}

static PT_THREAD (protothread_pwm(struct pt *pt))
{
    PT_BEGIN(pt) ;
    while(1) {
        ///////////////////////////////PWM Servo /////////////////////////////////////////
        if (in_data_index > 150 ){
            pwm_set_chan_level(slice_num_index, PWM_CHAN_B, 2000);
        }
        else {
            pwm_set_chan_level(slice_num_index, PWM_CHAN_B, 1000);
        }

        if (in_data_middle > 150){
            pwm_set_chan_level(slice_num_middle, PWM_CHAN_A, 2000);
            pwm_set_chan_level(slice_num_thumb, PWM_CHAN_B, 1000);
        }
        else {
            pwm_set_chan_level(slice_num_middle, PWM_CHAN_A, 1000);
            pwm_set_chan_level(slice_num_thumb, PWM_CHAN_B, 2000);
        }

        if (in_data_ring < 150){
            pwm_set_chan_level(slice_num_ring, PWM_CHAN_A, 2000);
            pwm_set_chan_level(slice_num_little, PWM_CHAN_B, 2000);
        }
        else {
            pwm_set_chan_level(slice_num_ring, PWM_CHAN_A, 1000);
            pwm_set_chan_level(slice_num_little, PWM_CHAN_B, 1000);
        }
        
    }
    PT_END(pt) ;
}

// Entry point for core 1
void core1_entry() {
    pt_add_thread(protothread_pwm) ;
    pt_schedule_start ;
}

int main() {
    // Initialize stdio/uart
    stdio_init_all();        

    // wait until the CDC ACM (serial port emulation) is connected
    while (!tud_cdc_connected()) 
    {
        sleep_ms(10);
    }

    //////////////////////////////////Transmiter //////////////////////////////////

    // initialise my_nrf
    nrf_driver_create_client(&my_nrf);

    // configure GPIO pins and SPI
    my_nrf.configure(&my_pins, my_baudrate);

    // not using default configuration (my_nrf.initialise(NULL)) 
    my_nrf.initialise(&my_config);

    /**
    * set addresses for DATA_PIPE_0 - DATA_PIPE_3.
    * These are addresses the transmitter will send its packets to.
    */
    my_nrf.rx_destination(DATA_PIPE_0, (uint8_t[]){0x37,0x37,0x37,0x37,0x37});
    my_nrf.rx_destination(DATA_PIPE_1, (uint8_t[]){0xC7,0xC7,0xC7,0xC7,0xC7});
    my_nrf.rx_destination(DATA_PIPE_2, (uint8_t[]){0xCA,0xC7,0xC7,0xC7,0xC7});

    my_nrf.payload_size(ALL_DATA_PIPES, FIVE_BYTES);
  
    // set to RX Mode
    my_nrf.receiver_mode();
    
    ///////////////////////////////PWM Servo /////////////////////////////////////////
    gpio_set_function(7, GPIO_FUNC_PWM);
    gpio_set_function(8, GPIO_FUNC_PWM);
    gpio_set_function(10, GPIO_FUNC_PWM);
    gpio_set_function(9, GPIO_FUNC_PWM);
    gpio_set_function(11, GPIO_FUNC_PWM);


    // Find out which PWM slice is connected to GPIO 5 (it's slice 2)
    slice_num_index = pwm_gpio_to_slice_num(7);
    slice_num_middle = pwm_gpio_to_slice_num(8);
    slice_num_ring = pwm_gpio_to_slice_num(10);
    slice_num_little = pwm_gpio_to_slice_num(9);
    slice_num_thumb = pwm_gpio_to_slice_num(11);

     // This section configures the period of the PWM signals
    pwm_set_clkdiv(slice_num_index, CLKDIV) ;
    pwm_set_wrap(slice_num_index, WRAPVAL) ;

    pwm_set_clkdiv(slice_num_middle, CLKDIV) ;
    pwm_set_wrap(slice_num_middle, WRAPVAL) ;

    pwm_set_clkdiv(slice_num_ring, CLKDIV) ;
    pwm_set_wrap(slice_num_ring, WRAPVAL) ;

    pwm_set_clkdiv(slice_num_little, CLKDIV) ;
    pwm_set_wrap(slice_num_little, WRAPVAL) ;

    pwm_set_clkdiv(slice_num_thumb, CLKDIV) ;
    pwm_set_wrap(slice_num_thumb, WRAPVAL) ;

    // This sets duty cycle
    pwm_set_chan_level(slice_num_index, PWM_CHAN_B, 0);
    pwm_set_chan_level(slice_num_middle, PWM_CHAN_A, 0);
    pwm_set_chan_level(slice_num_ring, PWM_CHAN_A, 0);
    pwm_set_chan_level(slice_num_little, PWM_CHAN_B, 0);
    pwm_set_chan_level(slice_num_thumb, PWM_CHAN_B, 0);

    // Start the channel
    pwm_set_enabled(slice_num_index, true);
    pwm_set_enabled(slice_num_middle, true);
    pwm_set_enabled(slice_num_ring, true);
    pwm_set_enabled(slice_num_little, true);
    pwm_set_enabled(slice_num_thumb, true);

    ////////////////////////////////////////////////////////////////////////
    ///////////////////////////// ROCK AND ROLL ////////////////////////////
    ////////////////////////////////////////////////////////////////////////
    // start core 1 
    multicore_reset_core1();
    multicore_launch_core1(core1_entry);

    // start core 0
    pt_add_thread(protothread_serial) ;
    pt_schedule_start ;
}

```