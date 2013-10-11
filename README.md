# Patch for r38025
This branch is for OpenWrt trunk r38025.  
And thers patch only supports 16M flash / 32M SDRAM HLK-RM04 boards.

## GPIO Definition

	RIN 	<==> reset
	GPIO0   <==> BTN_0
	I2C_SD	<==> gpio1(input)
	I2C_SCK <==> gpio2(output)