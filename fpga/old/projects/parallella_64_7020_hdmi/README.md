# FPGA PROJECT 

parallella_64_7020_hdmi - Interface to Epiphany E64 on Parallella-I board, with HDMI.

## Toolchain

Xilinx ISE/EDK/PlanAhead version 14.7

## Target Devices

* Zynq xc7z020clg400-1
* Epiphany E64G401

## Dependencies

* hdl sources in ../../hdl
* edk project in ../../edk/parallella_7020_hdmi
* external edk library ../../externals/fpgahdl_xilinx
* constraints in ../../../boards/parallella-I/constraints

## Build instructions

1.  If you have just checked-out this repository, you will also have to populate the external submodule in "fpga/externals/fpgahdl_xilinx" by running the script "get_fpgahdl_xilinx" in the /fpga/externals directory.  This step only has to be done once for each external library.

2.  If this is the first time you are trying to build a project that uses the parallella_7020_hdmi edk project, you will also have to point it to the external library pulled in step 1.  Unfortunately the Xilinx edk does not allow relative paths for libraries, so there is an extra step required here for each edk project that uses external libs:

	1.  Open the edk project "fpga/edk/parallella_7020_hdmi/system.xmp" using xps.

	1.  XPS will complain about a missing module, and present you with a dialog box to learn the new location.  Browse to "fpga/externals/fpgahdl_xilinx/cf_lib/edk/pcores/axi_hdmi_tx_16b_v1_00_a" in your local copy of the repository, and choose that directory.  From this information xps will be able to find all other modules it needs, and it will write the absolute path to these libraries into your copy of the xps project.

	1.  Close XPS.  The project file will automatically be saved.

	1.  This step has to be done for each edk project you want to use, e.g. parallella_7010_hdmi or parallella_7020_hdmi.

3. Be sure to update the version.v file in this directory with a new version number, and type or platform as needed.

4.  Launch the Xilinx PlanAhead tool

5.  Open the project fpga/projects/parallella_7020_hdmi/parallella_7020_hdmi.ppr.

6.  From the planAhead GUI, in the Flow Navigator on the left side, select "Generate Bitstream"

7.  Answer "Yes" to "OK to launch synthesis and implementation?"

```
    Linux hints:
      $ . /opt/Xilinx/14.7/ISE_DS/settings[32|64].[c]sh
      $ xps &
      $ planAhead &
```

## Creating derivative projects

To create a new project using this one as a base, open the project in planAhead and do "Save Project as...," specifying your new directory and project name.  The current source files may be left in place by leaving "Import all files to the new project" unchecked.

In the new project directory, create a new copy of version.v and update it with your settings.  After opening the new planAhead project you will have to remove the old 'version.v" and add the new local copy.  Select the file in the "sources" window, then in the "source file properties" windows set the type to "Verilog Header" and check the "Global" box, then press "apply."  The project is now ready to build.

## Output

The generated bitstream file will be found in the directory
*parallella_7020_hdmi.runs/impl_1*
To convert this to a raw binary file and copy it to the ../bitstreams directory, run the script 'getbits'.

##  Critical Warnings

The following 3 critical warnings are produced during the build, and should be ignored.  These constraints are auto-generated by xps so can not be fixed as far as I understand.  The locations are correct and agree with the schematic.
```
 [Constraints 18-5] Cannot loc instance 'processing_system7_0_PS_PORB_pin_IBUF' at site C7, Site location is not valid [../../edk/parallella_7020_hdmi/implementation/system_processing_system7_0_wrapper.ncf:160]

 [Constraints 18-5] Cannot loc instance 'processing_system7_0_PS_SRSTB_pin_IBUF' at site B10, Site location is not valid [../../edk/parallella_7020_hdmi/implementation/system_processing_system7_0_wrapper.ncf:161]

 [Constraints 18-5] Cannot loc instance 'processing_system7_0_PS_CLK_pin_IBUF' at site E7, Site location is not valid [../../edk/parallella_7020_hdmi/implementation/system_processing_system7_0_wrapper.ncf:162]
```

Any other critical warnings or timing errors not described here are significant and should be investigated.

## Timing errors

If FEATURE_CCLK_DIV is defined and the CCLK is set above 460MHz, there will be a timing failure reported because the BUFG buffer used to clock the OSERDESE2 that generates the core clock for the epiphany has a switching limit of 464MHz:

```
[Par 450] At least one timing constraint is impossible to meet because 
component switching limit violations have been detected for a constrained 
component. A timing constraint summary below shows the failing constraints 
(preceded with an Asterisk (*)). Please use the Timing Analyzer (GUI) or 
TRCE (command line) with the Mapped NCD and PCF files to evaluate the 
component switching limit violations in more detail...
```
and from the TRACE report:
```
--------------------------------------------------------------------------------
Slack: -0.489ns (period - min period limit)
  Period: 1.666ns
  Min period limit: 2.155ns (464.037MHz) (Tbcper_I(Fmax))
  Physical resource: parallella/ewrapper_link_top/io_clock_gen/clkout1_buf/I0
  Logical resource: parallella/ewrapper_link_top/io_clock_gen/clkout1_buf/I0
  Location pin: BUFGCTRL_X0Y3.I0
  Clock network: parallella/ewrapper_link_top/io_clock_gen/clkout0
--------------------------------------------------------------------------------
```

What I don't understand is why this is reported as an error for the FEATURE_CCLK_DIV case but not for the basic case, even though the basic case has the same BUFG macro on the same clock signal.  We have verified proper operation of the BUFG->OBUFDS outputs over 700MHz with the Zynq devices, so we're confident that this will work at least up to 600MHz as these projects implement it.  If that bothers you feel free to un-define the FEATURE_CCLK_DIV macro and limit yourself to running at the full clock rate.

## Notes

* TIMING: Note that the FPGA-side eLink receiver has a timing constraint of only 150MHz = 300Mb/s DDR.  The FPGA-transmit interface passes timing at 300MHz = 600Mb/s.  The SDK will set the epiphany-fpga clock speed to 150MHz.

## TODO / Wishlist

* Create makefile-based build flow.
* Extract technology-dependent blocks from inner modules, move to top level.
* Replace RTL serializer/deserializer logic with Zynq IOSERDES modules for better timing.


