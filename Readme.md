# Introduction

A software and hardware co-design, this project is a study into whether a 2-layer aritificial neural network (ANN) is sufficient for recognition of numerical digits of the MNIST dataset. It is a fork of this neural network (NN) [project](https://github.com/vipinkmenon/neuralNetwork) which employs 4-layers. The analysis was validated in hardware with a Zedboard. The full report is [here](EE8223_F23_Final_Project_Report_v2.pdf). There are three main aspects of this project. 

1. Fixed point representation: No need for floating point representation because range of numbers is small. Most NNs use normalized inputs between -1 and 1 to represent both positive and negative numbers therefore fixed point is preferred as many 100s of neurons can be implemented in low-end FPGAs like Zynq with this format. 

2. Activation function (AF): Sigmoid is preferred over Rectilinear Activation (ReLu) due to the boundedness (between 0 and 1) of the former. When y = x if x > 0, x can be large where its digital representation becomes problematic if the width of the data bus is immutable.

3. Flexibility, scalability and portability: No Xilinx specific IP cores are used to provide a platform independent solution. Everything is built from Verilog. Even though IP cores are more efficient, its not scalable for changing sizes of ROM for AF where a new IP core will have to be generated. 

# Low-Level Details

1. Weights memory
    - Implemeted as a ROM or RAM based on if the user wants to user wants to use pretrained values or not.
    - If chosing pretrainted values, it is implemented as a ROM where the weight file is read as a binary and initialized to a 2-D memory
    - If not using pretrained values then RAM is used to send weight values to a 2-D memory on every clock cycle (cc)
    - Reading is common among both implementations but read sequentially incurring 1 cc read latency by inferring a blockRAM
    - Therefore input is delayed by 1 cc in order for the multiplication process
    - implemented as a shared bus where all the neurons are hooked up to the same bus
2. Inputs
    - Regardless of the amount of neurons there is only one input bus, so each input to a neuron arrives sequentially. 
    - delayed by 1 cc in order to be mulitiplied by the corresponding weight
3. Multiplication and addition (MAC) operation
    - 2's complement is used therefore **signed** type is enabled
    - Overflow and underflow conditions are monitored
4. Bias
    - Needed when the last weight value is used. 
    - checked for overflow and underflow conditions
5. Activation Function
    - Using Generate statement to selectively instantiate one of the AFs chosen
    - 2 separate verilog modules used for Sigmoid or ReLu.
    - precalculated values are used for Sigmoid and initialized as a ROM where only the upper bits of inputs are used. 
    - The higher the depth of Sigmoid ROM the more resources. The input is 32 bits long but cannot have a ROM of size $2^{32}$ so only upper bits are used.
    - Sigmoid memory is implemented as distributed RAM (using LUTs and Flip Flops) rather than block RAM because the size of sigmoid memory is quite small compared to weights file (32 or 64 bit deep). Smallest blockRAM is 18 Kb. 
6. Pipeline delays
    - 3 pipeline delays from input to output
    
## hardmax 

[Softmax](https://en.wikipedia.org/wiki/Softmax_function) is an exponential divided by the sum of exponentials from all output neurons, a formula that is difficult to implement in hardware. Two options for workarounds:

1. send ouputs of neurons to processing system (PS) and perform softmax there
2. use a hardware compatible version, in this case, called 'Hardmax'.  

Option 2 is selected and module called maxFinder.v is used to find the highest value from the 10 output neurons without normalizing it. Uses 10 cc to find out the highest. 

# Results

The results, which are not exhaustive, are shown in the picture below. 
<div align="center">
    <img src="9_Hardware_validation/HWValidation_pic1.jpg" alt="Cant find image" />
</div>
<p align="center"><em>Figure 1: The output </em></p>
