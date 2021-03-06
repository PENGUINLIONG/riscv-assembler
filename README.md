[![Build Status](https://travis-ci.org/kcelebi/riscv-assembler.svg?branch=main)](https://travis-ci.org/kcelebi/riscv-assembler)
[![Gitter](https://badges.gitter.im/riscv-assembler/community.svg)](https://gitter.im/riscv-assembler/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

# riscv-assembler Documentation
RISC-V Assembly code assembler package.

This package contains tools and functions that can convert **RISC-V Assembly code to machine code**. The whole process is implemented using Python purely for understandability, less so for efficiency in computation. These tools can be used to **convert given lines of code or [whole files](#convert) to machine code**. For conversion, output file types are binary, text files, and printing to console. The supported instruction types are **R, I, S, SB, U, and UJ**. Almost all standard instructions are supported, most pseudo instructions are also supported (see [helper functions](#helper-functions) about adding pseudo/missing instructions).

Feel free to open an issue or contact me at [kayacelebi17@gmail.com](mailto:kayacelebi17@gmail.com?subject=[GitHub]%20riscv-assembler) with any questions/inquiries.

# Table of Contents

Use the links below to jump to sections of the documentation:

- [Installation](#installation)
- [Usage](#usage)
    - [AssemblyConverter](#assemblyconverter)
        - [Convert](#convert)
        - [Helper Functions](#helper-functions)
    - [Toolkit](#toolkit)
        - [Instruction Format Functions](#instruction-format-functions)
        - [Debugger Functions](#debugger-functions)
- [What is this?](#what-is-this)
- [How it Works](#how-it-works)
- [Future Plans](#future-plans)

# Installation

The package can be installed using pip:

`$ pip install riscv-assembler`

If issues arise try:

`$ python3 -m pip install riscv-assembler`

It is possible that the `bitstring` dependency might not install correctly. If this occurs, you can simply pip install it separately:

`$ pip install bitstring`

No other actions necessary.

# Usage

The package works through an `AssemblyConverter` class and a `Toolkit` class. The `AssemblyConverter` class contains functions to convert whole files and projects from Assembly code to machine code. The `Toolkit` class contains tools to convert individual lines, apply specific instruction format types, calculate branching immediates, and many other [functions](#toolkit).

## AssemblyConverter
Let's check out how to use `AssemblyConverter`. First, we need to import it into our Python file:

`from riscv_assembler.convert import AssemblyConvert`

We can now instantiate an object. The constructor is initialized as so:

    AssemblyConverter(output_type = "b", nibble = False, hexMode = False)
    
`output_type` refers to whether a converted file should be outputted to a binary file ("b"), a text file ("t"), or to console ("p"). The three of these options can be used in any sort of combination. Here are acceptable usages:
    
    cnv = AssemblyConverter(output_type = "btp") #binary and text and printing
    cnv = AssemblyConverter(output_type = "pbt") #works same as above ^
    cnv = AssemblyConverter(output_type = "b") #just binary
    cnv = AssemblyConverter(output_type = "t") #just text
    cnv = AssemblyConverter(output_type = "p") #just printing
    cnv = AssemblyConverter() #binary by default

Jump to [convert](#convert) for more details on outputs for `AssemblyConverter`.

`nibble` refers to whether (for text file and console outputs) the 32-bit binary numbers should be split in nibbles or kept as an unbroken string. An example output for `nibble = True` would be:

    1101    0110    0000   0000 ...

An example for the default `nibble = False` would be:

    11010110000000...

`hexMode` gives the option (for text file and console outputs) to output in hexadecimal form instead of binary. By default, the outputs are in binary. Leading zeros are included to properly represent a 32-bit instruction.

### Convert
With this object we can apply our most powerful function : `convert()`. This function takes in a file name (with .s extension) from the local directory and converts it to the output file of your choice, specified by the object construction. Let's convert the file `simple.s`:

    cnv.convert("simple.s")

Any empty files, fully commented files, or files without `.s` extension will not be accepted. The function creates a directory by truncating the extension of the file along with two subdirectories called `bin` and `txt` for the respective output files (for `simple.s` the directory would be `simple`). Output files will be stored there for usage and/or printed to console.

### Helper Functions

Here are a few functions that might be useful:

#### getOutputType()

This function simply prints the output type that has been initially selected. Example usage:

    cnv = AssemblyConverter("bt") #initially write to binary and text file
    output_type = cnv.getOutputType()
    
    print(output_type)

This will print to console:
    
    bt

#### setOutputType()

This function allows the option to change the output type after initialization. Example usage:

    cnv = AssemblyConverter("bt") #initially write to binary and text file
    cnv.setOutputType("p") #now only print to console

#### instructionExists()

This function returns a boolean for whether a provided instruction exists in the system. Example usage:

    cnv = AssemblyConverter()
    cnv.instructionExists("add") #yields true
    cnv.instructionExists("hello world") #yields false
    
## Toolkit

Let's check out how to use `Toolkit`. `Toolkit` contains functions that can be used to interpret specific lines, calculate jumps, and other debugging tools. First, let's import the class and functions:

    from riscv_assembler.utils import *

**Some of these functions rely on the class, some do not**. First, let's look at those that rely on the `Toolkit` class. Let's instantiate an object.

    tk = Toolkit()

No parameters required. Let's now look at some functions.

### Instruction Format Functions

This package also offers instruction format-specific functions for individual lines of assembly. The instruction types supported are R, I, S, SB, U, and UJ. The machine code is returned as a bitstring that can be printed to console. Check out the examples below for each instruction type:

#### R_type()

This functions converts individual lines of assembly with R type instructions to machine code. The function usage is:

`R_type(operation, rs1, rs2, rd)`

Here is an example of translating `add x0 s0 s1`

    from riscv_assembler.convert import AssemblyConverter
    
    cnv = AssemblyConverter()   
    instr = cnv.R_type("add","x0","s0","s1") #convert the instruction
    
    print(instr) 

Note that the registers are being written as strings. The package maps them correctly to their respective binary values (ex. `s0` maps to `x8`).

#### I_type()

This functions converts individual lines of assembly with I type instructions to machine code. The function usage is:

`I_type(operation, rs1, imm, rd)`

Here is an example of translating `addi x0 x0 32`

    from riscv_assembler.convert import AssemblyConverter
    
    cnv = AssemblyConverter() #output to binary   
    instr = cnv.I_type("addi","x0","32","x0") #convert the instruction
    
    print(instr)

Note that the immediate is a string, not just a number. This was implemented this way for seamless integration with the convert() function, there is an easy workaround for using it on its own. 

#### S_type()

This functions converts individual lines of assembly with S type instructions to machine code. The function usage is:

`S_type(operation, rs1, rs2, imm)`

Here is an example of translating `sw x0 0(sp)`

    from riscv_assembler.convert import AssemblyConverter
    
    cnv = AssemblyConverter() #output to binary   
    instr = cnv.S_type("sw","x0","sp","0") #convert the instruction
    
    print(instr)
    
#### SB_type()

This functions converts individual lines of assembly with SB type instructions to machine code. The function usage is:

`SB_type(operation, rs1, rs2, imm)`

Here is an example of translating `beq x0 x1 loop`:

    from riscv_assembler.convert import AssemblyConverter
    
    cnv = AssemblyConverter() #output to binary   
    instr = cnv.SB_type("beq","x0","x1","loop") #convert the instruction
    
    print(instr)

Note that the jump is written as a string, the appropriate instruction jump is calculated by the package.

#### U_type()

This functions converts individual lines of assembly with U type instructions to machine code. The function usage is:

`U_type(operation, imm, rd)`

Here is an example of converting `lui x0 10`:

    from riscv_assembler.convert import AssemblyConverter
    
    cnv = AssemblyConverter() #output to binary   
    instr = cnv.U_type("lui","x0","10") #convert the instruction
    
    print(instr)
    
#### UJ_type()

This functions converts individual lines of assembly with UJ type instructions to machine code. The function usage is:

`UJ_type(operation, imm, rd)`

Here is an example of converting `jal a0 func`:

    from riscv_assembler.convert import AssemblyConverter
    
    cnv = AssemblyConverter() #output to binary   
    instr = cnv.UJ_type("jal","func","a0") #convert the instruction
    
    print(instr)
    
### Debugger Functions

#### hex()

**Note that this is an overrided verison of the default hex() function**. This function converts an inputted string of binary to 32-bits in hexadecimal. There is an option to maintain leading zeros or not. The function usage is:

    from riscv_assembler.utils import *
    
    tk = Toolkit()
    
    string  = '00000000000000000000111100110011'
    
    print(tk.hex(string,leading_zero=True)) #outputs '0x00000F33'
    print(tk.hex(string,leading_zero=False)) #outputs '0xF33'

#### nibbleForm()

This function takes an inputted string of unbroken binary and outputs it in nibbles separated by an inputted delimeter. The function usage is:

    from riscv_assembler.utils import *
 
    string  = '00000000000000000000111100110011'

    print(nibbleForm(string, delim = '\t')) #outputs separed by tabs '0000    0000    0000    0000    0000    1111    0011    0011'
    
    print(nibbleForm(string, delim = 'qqq') #outputs '0000qqq0000qqq0000qqq0000qqq0000qqq1111qqq0011qqq0011'

#### calcJump()

This function takes in a filename, a name of a function to jump to, and the line number of a branch statement to calculate the branch jump immediate. The line number should **not** just be the line number in your text editor; ignore empty lines, comments, and lines that indicate the start of a function (such as `loop:`). **Consider the first valid line as line 0**. The function usage is:

    from riscv_assembler.utils import *
    
    tk = Tookit()
    
    print( tk.calcJump(filename = "myfile.s", x = 'loop', line_num = 42) #outputs an immediate according to how
                                                                         #many instructions need to be jumped to get to the function
    
<!-- ### addPseudo()-->
For functions that do not exist in the system that are used often, feel free to contact me to have it implemented. Most functions are readily available, however, with a growing library it is possible that a pseudo instruction or two slipped past. I intend on implementing functions that allow for the user to bypass any shortfalls like this (see [future plans](#future-plans) )

# What is this?

Your computer relies on streams of 1's and 0's to do all of its operations. This is the lowest level of code, called machine code, that can be used on a computer. Of course, machine code is quite annoying/impossible to interpret on the fly, so we have other higher levels of code, such as Python, Java, and C, that allow us to give instructions that get translated to machine code. The second-lowest level of code is called Assembly code, and it is mostly used to program Operating Systems and basic hardware functions of a computer.

Writing Assembly code to test hardware is quite annoying because you have to deal with machine code and the long 32 or 64-bit binary numbers. This package provides tools to help make tests and debug Assembly code that might be used to test/create hardware. This package also serves an educational purpose for those learning about Computer Architecture and RISC-V.

In our computer, there are different ways that we can format our hardware to interpret the 1's and 0's. Intel has a format called x86 that has been used in computers for many years. It has its own kind of formatting and way of interpreting Assembly code. More recently, there has been an emergence in popularity of a different format called RISC-V, owned by a company called ARM. These chips are mostly used in phones, and have been incredibly successful with the iPhone. ARM chips recently became even more popular with Apple's new M1 Macbook Pro which uses an ARM architecture to structure its hardware. This new computer has been known to absolutely wipe out its competitors in processing power and capability. 
# How it Works

This package works by parsing through given `.s` files, filtering through comments and empty space to find instructions, categorizing them by their instruction type, then constructing the machine code. These tools have been tested quite thoroughly and work well for individual files. Files grouped in projects have not been implemented yet, but should be [soon](#future-plans).

This package was designed with the intention of helping with CPU architecture engineering (see [future plans](#future-plans) ). Machine code is difficult to interpret which can make it difficult to create efficient tests by hand. Simple software like this can help generate testing material and help debug CPU systems. 

# Future Plans

I created this package purely to provide an extra set of tools for a difficult job and also to display my understanding of Computer Architecture systems in RISC-V. This package was first released when I was a 2nd-year student in college, and I intend to keep improving it overtime in a professional manner. I have many ideas for new implementations which I will update this README.md with. For any questions or inquiries, please feel free to contact me at [kayacelebi17@gmail.com](mailto:kayacelebi17@gmail.com?subject=[GitHub]%20riscv-assembler).

### Project Files

One of the bigger problems I intend to tackle is dealing with multiple files in a project. Currently, this package can only handle files that exist on their own. This implementation will be coming soon. 

### Adding Custom Instructions
A function that I intend to implement is `addPseudo()` which gives users the opportunity to add their own custom functions or fill in functions that might be missing from the package data. 

### Machine Code Decoder
The next logical project to tackle would be to create the reverse system: decoding machine code to create `.s` files. This is much more ambitious and can be achieved down the line. 
