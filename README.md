# FPGA Convolution Accelerator
This project implements a hardware accelerator for convolution operations used primarily in computer vision tasks, such as image processing, using FPGA. The accelerator is designed to be highly parallel, optimizing the computation of convolution operations on 3×3 kernels over image patches using multiple MAC (Multiply-Accumulate) units. It handles fixed-point arithmetic, sliding window generation, kernel selection, and activation functions (like ReLU) for efficient image feature extraction. New features are being added in a continuing basis. The implementation is the next step from my MIPS Pipelined Processor Project.

## Table of Contents
1. Overview
2. Design Modules
    * Memory Loader
    * Sliding Window Generator
    * Kernel Selector
    * Parallel MAC Units
    * Activation & Output Writer
3. How It Works
4. Top-Level Integration
5. Resources and Requirements
6. Future Work

## Overview
This FPGA-based Convolution Accelerator is designed to optimize the computation of convolutions for computer vision tasks. Convolution operations are at the heart of many image processing algorithms, such as edge detection, blur, and feature extraction in Convolutional Neural Networks (CNNs).

The design is broken down into several key components that work in tandem to accelerate the process:

![High-Level Overview](img/high_level.png)

1. Memory Loader: Efficiently loads pixel data from memory (e.g., DDR or BRAM).
2. Sliding Window Generator: Extracts 3x3 pixel windows from the input image.
3. Kernel Selector: Allows the selection of different convolution kernels.
4. Parallel MAC Units: Perform the actual convolution by multiplying pixels by the corresponding kernel coefficients.
5. Activation and Output Writer: Applies activation functions like ReLU and writes the results back to memory.

## Design Modules
### Memory Loader
The Memory Loader fetches data from memory (e.g., DDR or BRAM) and feeds pixel blocks to the rest of the pipeline. It is optimized to perform burst reads to increase throughput and reduce latency when accessing large datasets.
```
module memory_loader_burst #(
    parameter BURST_SIZE = 16
) (
    input wire clk,
    input wire rst,
    input wire read_enable,
    input wire [31:0] base_addr,
    output reg [7:0] pixel_out [BURST_SIZE-1:0],
    input wire [7:0] mem_data [BURST_SIZE-1:0],
    output reg mem_read
);
```

## Sliding Window Generator
The Sliding Window Generator is responsible for extracting 3×3 pixel windows from the input data stream. It uses line buffers to efficiently shift and store pixel values, ensuring that the convolution operation can be performed on every 3×3 region of the image.

```
module sliding_window_generator #(
    parameter WIDTH = 256
) (
    input wire clk,
    input wire rst,
    input wire [7:0] pixel_in,
    output reg [7:0] window [0:8]
);
```

## Kernel Selector
The Kernel Selector module allows for dynamic selection of different convolution kernels. This is useful for tasks such as edge detection, blurring, and other image processing tasks that require different filters.
```
module kernel_selector (
    input wire clk,
    input wire rst,
    input wire [3:0] kernel_id,
    output reg [7:0] kernel_out [0:8]
);
```

## Parallel MAC Units
The Parallel MAC Units perform the convolution by multiplying the corresponding pixels of the window with the values in the convolution kernel, followed by the accumulation of these products.

```
module parallel_mac #(
    parameter NUM_MACS = 4
) (
    input wire clk,
    input wire rst,
    input wire [7:0] windows [NUM_MACS-1:0][0:8],
    input wire [7:0] kernel [0:8],
    output reg [15:0] results [NUM_MACS-1:0]
);
```

## Activation & Output Writer
The Activation and Output Writer module applies an activation function (ReLU) to the output of the convolution and writes the result back to memory. ReLU activation is commonly used to introduce non-linearity in the feature maps.

```
module relu_and_writer (
    input wire clk,
    input wire rst,
    input wire [15:0] conv_result,
    output reg [7:0] final_result
);
```

## How It Works
The FPGA Convolution Accelerator processes an image in the following sequence:
1. Memory Loader fetches pixels from memory in burst reads to reduce latency.
2. The Sliding Window Generator extracts 3×3 windows of pixels from the image.
3. The Kernel Selector selects the appropriate convolution kernel, which could be used for operations like edge detection, blurring, or any other kernel-based image operation.
4. The Parallel MAC Units execute the actual convolution operation by performing multiply-accumulate (MAC) for each pixel window, producing a convolved output.
5. The result is passed through an Activation Function (ReLU) to introduce non-linearity, followed by writing the results back to memory.

This design is optimized for parallelism, where multiple MAC operations are performed simultaneously, increasing throughput and reducing computation time.

## Top-Level Integration
The Top-Level Module integrates all of the described modules into a cohesive system, allowing the FPGA to handle the full end-to-end convolution process. It coordinates the flow of data from memory, through the computation units, and back to memory after processing.

```
module convolution_accelerator (
    input wire clk,
    input wire rst
);

// Connect all modules together
wire [7:0] pixel_data [3:0]; 
wire [7:0] windows [3:0][0:8]; 
wire [15:0] conv_results [3:0]; 

memory_loader_burst mem_loader (.clk(clk), .rst(rst), .pixel_out(pixel_data));
sliding_window_generator swg (.clk(clk), .rst(rst), .pixel_in(pixel_data[0]), .window(windows[0]));
kernel_selector ks (.clk(clk), .rst(rst), .kernel_out(kernel));
parallel_mac mac (.clk(clk), .rst(rst), .windows(windows), .kernel(kernel), .results(conv_results));

endmodule
```

This module coordinates the reading of data, the convolution operation, and writing the result to memory.

## Resources and Requirements
### Hardware:
* FPGA device (e.g., Xilinx or Intel FPGA)
* Memory (DDR/BRAM) for storing input image and output data
* Clock and reset signal management
### Software:
* Verilog (HDL for hardware description)
* FPGA synthesis and implementation tools (e.g., Vivado for Xilinx, Quartus for Intel)
### Simulation Tools:
* ModelSim or Vivado Simulator to simulate the design before actual hardware implementation

## Future Work
* Optimization: Explore further optimizations in memory access patterns (e.g., double buffering, tiling) to improve performance.
* Support for Larger Kernels: Extend the design to support larger kernels such as 5x5 or 7x7.
* Integration with Neural Networks: Incorporate this design as a building block for accelerating Convolutional Neural Networks (CNNs) on FPGA.
* FPGA Hardware Scaling: Consider scaling the accelerator to support higher resolutions or more complex image processing tasks.

## Conclusion
This project represents a foundational step in accelerating computer vision tasks using FPGA. It's designed to be modular and can be extended with additional features, optimizations, or other image processing techniques as needed.