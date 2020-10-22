
<h1>AD719x Driver Integrate Guide</h1>
</br>

| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;   |  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; |
| :---- | :---- |
| Project | AD719x Driver Module |
| Document name | AD719x_Driver_IntegrateGuide |
| Version  | 1.0 |
| Status | Draft |
| Auther | Gu Jun |
| Date   | 2020-08-15 |

</br>

<span id="API function overview"> **Table of Contents**</span>
<!-- TOC -->

- [1. Introduction](#1-introduction)
- [2. Integration into Rainbow](#2-integration-into-rainbow)
    - [2.1. Add Source Files](#21-add-source-files)
    - [2.2. Module Enable](#22-module-enable)
    - [2.3. Specific Definitions](#23-specific-definitions)
        - [2.3.1. ADC chip definition](#231-adc-chip-definition)
        - [2.3.2. Pin definition](#232-pin-definition)
        - [2.3.3. SPI Communication definition](#233-spi-communication-definition)
    - [2.4. RTOS Support](#24-rtos-support)
    - [2.5. Data Ready Interrupt](#25-data-ready-interrupt)

<!-- /TOC -->

<span id="API function overview"> **History of Changes**</span>

| Version | Date  | Status | Notes | Author |
| ---- | ---- | ---- | ---- | ---- |
1.0 | 15.Aug.2020 | Draft | Initial Version  | Gu Jun |


</br>

# 1. Introduction
<a id="markdown-introduction" name="introduction"></a>
This document describes how to reuse and integrate the AD719x driver module into a new project.
</br>

# 2. Integration into Rainbow
<a id="markdown-integration-into-rainbow" name="integration-into-rainbow"></a>
This section describes the integration of the AD719x driver module into software system based on Rainbow.

## 2.1. Add Source Files
<a id="markdown-add-source-files" name="add-source-files"></a>
These five AD719x source files should be added in the project. 
- dev
    - AD719x.c
    - AD719x.h
- drv
    - ExtADC_AD719X.c
    - ExtADC_AD719X.h
    - ExtADC_IO.h

The AD719x.c source deal with the ADC chip configuration and operation without knowing the detailed IO specification.  It is seperated from ExtADC_AD719X.c just for conveniently supporting "<b>AD7190</b>" and "<b>AD7195</b>" etc. 
The logical relation of these source files is decribled as follows:

``` dot
digraph G {

  subgraph cluster_0 {
    style=filled;
    color=lightgrey;
    node [style=filled,color=white];
    ExtADC_AD719X ExtADC_IO;
    label = "drv";
  }

  subgraph cluster_1 {
    node [style=filled];
    AD719x;
    label = "dev";
    color=lightgrey;
  }
  Application -> ExtADC_AD719X;
  ExtADC_AD719X -> AD719x;
  AD719x -> ExtADC_IO;
  ExtADC_IO -> Chip;

  Application [shape=Mdiamond];
  Chip [shape=Msquare];
}
```
Normally, if there is a new AD719x serial chip to be suported, only AD719x.c file in "dev" directory needs to be updated, without modifying the ExtADC_AD719X.c file in "drv" directory.
In this module repository, it is just for demonstration to put the source files into two directories "dev" and "drv" seperately. A new project could add these files any way for convenience, as long as the directory path containing the header file is listed in the 'include directories' of the project.

## 2.2. Module Enable
<a id="markdown-module-enable" name="module-enable"></a>
The AD719x driver belongs to External ADC function in Rainbow system. So it is nesessary to define the RB_CONFIG_USE_EXTADC micro with RB_CONFIG_YES in "<b>RB_Config_Use.h</b>" file as follows: 
``` c
#define RB_CONFIG_USE_EXTADC                RB_CONFIG_YES
```
Then the AD719x driver could be enabled in "<b>RB_Config_Board.h</b>". And meanwhile the other external ADC drivers should be disabled.
``` c
#if defined(RB_CONFIG_USE_EXTADC) && (RB_CONFIG_USE_EXTADC == RB_CONFIG_YES)
	//! Enable External ADC drivers to be used.
	#define RB_CONFIG_EXTADC_USE_SIMULATOR			RB_CONFIG_NO
	#define RB_CONFIG_EXTADC_USE_AD719X 			RB_CONFIG_YES
#endif
```

## 2.3. Specific Definitions
<a id="markdown-specific-definitions" name="specific-definitions"></a>

```c
 #if defined(RB_CONFIG_EXTADC_USE_AD719X) && \
            (RB_CONFIG_EXTADC_USE_AD719X == RB_CONFIG_YES)
          


    d
 
 	       


	 
	#endif
```
### ADC chip definition
<a id="markdown-adc-chip-definition" name="adc-chip-definition"></a>
Now AD7190 an AD7195 are supported, so the ADC chip definiion as follows will let the driver know which ADC chip it deals with.
```c
#define RB_CONFIG_EXTADC_AD719X_TYPE		    7190
```
Or

```c
#define RB_CONFIG_EXTADC_AD719X_TYPE		    7195
```

### Pin definition
<a id="markdown-pin-definition" name="pin-definition"></a>
```c
// SCLK PIN
#define RB_CONFIG_EXTADC_AD719X_SCLK_PORT       RB_PORT_3
#define RB_CONFIG_EXTADC_AD719X_SCLK_PIN        RB_PORT_BIT_15
// DOUT PIN
#define RB_CONFIG_EXTADC_AD719X_DOUT_PORT       RB_PORT_3
#define RB_CONFIG_EXTADC_AD719X_DOUT_PIN        RB_PORT_BIT_17
// DIN PIN
#define RB_CONFIG_EXTADC_AD719X_DIN_PORT        RB_PORT_3
#define RB_CONFIG_EXTADC_AD719X_DIN_PIN         RB_PORT_BIT_16
// CS PIN
#define RB_CONFIG_EXTADC_AD719X_CS_PORT         RB_PORT_3
#define RB_CONFIG_EXTADC_AD719X_CS_PIN          RB_PORT_BIT_18
// SYNC PIN
#define RB_CONFIG_EXTADC_AD719X_SYNC_PORT       RB_PORT_3
#define RB_CONFIG_EXTADC_AD719X_SYNC_PIN        RB_PORT_BIT_19
// RDY PIN
#define RB_CONFIG_EXTADC_AD719X_RDY_PORT        RB_PORT_3
#define RB_CONFIG_EXTADC_AD719X_RDY_PIN         RB_PORT_BIT_12
```

### SPI Communication definition
<a id="markdown-spi-communication-definition" name="spi-communication-definition"></a>
```c
#define RB_EXTADC_AD719X_SPI_TYPE_EMULATE       0
#define RB_EXTADC_AD719X_SPI_TYPE_MASTER        1
#define RB_EXTADC_AD719X_SPI_TYPE_INTERRUPT     2
#define RB_EXTADC_AD719X_SPI_TYPE               RB_EXTADC_AD719X_SPI_TYPE_EMULATE

#if (RB_EXTADC_AD719X_SPI_TYPE != RB_EXTADC_AD719X_SPI_TYPE_EMULATE)
#define RB_CONFIG_EXTADC_AD719X_SSP_INDEX		0
#endif
```

### 2.6. RTOS Support
<a id="markdown-rtos-support" name="rtos-support"></a>
```c
#define RB_EXTADC_AD719X_RTOS_SUPPORT           1
```

### 2.5. Data Ready Interrupt
<a id="markdown-data-ready-interrupt" name="data-ready-interrupt"></a>
```c
#define RB_EXTADC_AD719X_USE_TIMER             3
```

### Rainbow ExtADC Channel
```c
#define RB_CONFIG_EXTADC_CHANNEL_TABLE \
/*  Logical Channel Name    Phys. Channel Number    Phys. Driver    */	\
RB_EXTADC_ITEM(RB_CONFIG_EXTADC_AD719X_WEIGHT,  0,  RB_EXTADC_AD719X_Driver)
```

``` mermaid
gantt
    dateFormat DD-MM-YYY
    axisFormat %m/%y

    title Example
    section example section
    activity :active, 01-02-2019, 03-08-2019
```

$\sqrt{3x-1}+(1+x)^2$


``` plantuml
Bob -> Alice : hello
```

