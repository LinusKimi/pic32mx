# pic32mx
pic32mx wdt nmi

When I was using the NMI function of the WDT in the pic32mx sleeping, the device would not wake up properly.
Consult the information to solve the problem, the following is the solution;

1) Add the following to the end of the crt0.s file

```
	.text
	.align	2
	.weak	_nmi_handler
	.set	nomips16
	.ent	_nmi_handler
_nmi_handler:
	.frame	sp,0,$31		# vars= 0, regs= 0/0, args= 0, gp= 0
	.mask	0x00000000,0
	.fmask	0x00000000,0
	.set	noreorder
	
        mfc0    k0, _CP0_STATUS                   # retrieve STATUS
        lui     k1, ~(_CP0_STATUS_BEV_MASK >> 16) & 0xffff
        ori     k1, k1, ~_CP0_STATUS_BEV_MASK & 0xffff
        and     k0, k0, k1                        # Clear BEV
        mtc0    k0, _CP0_STATUS                   # store STATUS
        eret

	.set	macro
	.set	reorder
	.end	_nmi_handler
	.size	_nmi_handler, .-_nmi_handler
```

2) Don't use POWER_LowPowerModeEnter(XXXXX) function; Use the following code instead of this function
```
asm volatile( "di "); // disable interrupts while we set up
SYSKEY = 0x0; // osccon unlock sequence
SYSKEY = 0xAA996655; // osccon unlock sequence
SYSKEY = 0x556699AA; // osccon unlock sequence
WDTCONbits.ON = 1; // enable watchdog
WDTCONbits.WDTCLR = 1; // clear timer
OSCCONbits.SLPEN = 1; // we want sleepy time
SYSKEY = 0x0; // re-lock osccon
asm volatile( "ei "); // re-enable interrupts
asm volatile( "wait "); // go to sleep
Nop();
```

3) Have fun !

具体原因没有深追，盲猜可能是睡眠后NMI唤醒时函数堆栈有问题吧，有空再详细研究。  airlcade@163.com
