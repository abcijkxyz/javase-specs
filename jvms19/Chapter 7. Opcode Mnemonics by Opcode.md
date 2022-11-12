

Chapter 7. Opcode Mnemonics by Opcode
========================================================================================================

This chapter gives the mapping from Java Virtual Machine instruction opcodes, including the reserved opcodes ([§6.2](jvms-6.html#jvms-6.2 "6.2. Reserved Opcodes")), to the mnemonics for the instructions represented by those opcodes.

Opcode value 186 was not used prior to Java SE 7.



**Table 7.1.** 

  
| Constants | Loads | Stores |
| :-: | :-: | :-: |
| 
  
`00 (0x00)`    nop  
`01 (0x01)`    aconst\_null  
`02 (0x02)`    iconst\_m1  
`03 (0x03)`    iconst\_0  
`04 (0x04)`    iconst\_1  
`05 (0x05)`    iconst\_2  
`06 (0x06)`    iconst\_3  
`07 (0x07)`    iconst\_4  
`08 (0x08)`    iconst\_5  
`09 (0x09)`    lconst\_0  
`10 (0x0a)`    lconst\_1  
`11 (0x0b)`    fconst\_0  
`12 (0x0c)`    fconst\_1  
`13 (0x0d)`    fconst\_2  
`14 (0x0e)`    dconst\_0  
`15 (0x0f)`    dconst\_1  
`16 (0x10)`    bipush  
`17 (0x11)`    sipush  
`18 (0x12)`    ldc  
`19 (0x13)`    ldc\_w  
`20 (0x14)`    ldc2\_w  
      



 | 

  
`21 (0x15)`    iload  
`22 (0x16)`    lload  
`23 (0x17)`    fload  
`24 (0x18)`    dload  
`25 (0x19)`    aload  
`26 (0x1a)`    iload\_0  
`27 (0x1b)`    iload\_1  
`28 (0x1c)`    iload\_2  
`29 (0x1d)`    iload\_3  
`30 (0x1e)`    lload\_0  
`31 (0x1f)`    lload\_1  
`32 (0x20)`    lload\_2  
`33 (0x21)`    lload\_3  
`34 (0x22)`    fload\_0  
`35 (0x23)`    fload\_1  
`36 (0x24)`    fload\_2  
`37 (0x25)`    fload\_3  
`38 (0x26)`    dload\_0  
`39 (0x27)`    dload\_1  
`40 (0x28)`    dload\_2  
`41 (0x29)`    dload\_3  
`42 (0x2a)`    aload\_0  
`43 (0x2b)`    aload\_1  
`44 (0x2c)`    aload\_2  
`45 (0x2d)`    aload\_3  
`46 (0x2e)`    iaload  
`47 (0x2f)`    laload  
`48 (0x30)`    faload  
`49 (0x31)`    daload  
`50 (0x32)`    aaload  
`51 (0x33)`    baload  
`52 (0x34)`    caload  
`53 (0x35)`    saload  
      



 | 

  
`54 (0x36)`    istore  
`55 (0x37)`    lstore  
`56 (0x38)`    fstore  
`57 (0x39)`    dstore  
`58 (0x3a)`    astore  
`59 (0x3b)`    istore\_0  
`60 (0x3c)`    istore\_1  
`61 (0x3d)`    istore\_2  
`62 (0x3e)`    istore\_3  
`63 (0x3f)`    lstore\_0  
`64 (0x40)`    lstore\_1  
`65 (0x41)`    lstore\_2  
`66 (0x42)`    lstore\_3  
`67 (0x43)`    fstore\_0  
`68 (0x44)`    fstore\_1  
`69 (0x45)`    fstore\_2  
`70 (0x46)`    fstore\_3  
`71 (0x47)`    dstore\_0  
`72 (0x48)`    dstore\_1  
`73 (0x49)`    dstore\_2  
`74 (0x4a)`    dstore\_3  
`75 (0x4b)`    astore\_0  
`76 (0x4c)`    astore\_1  
`77 (0x4d)`    astore\_2  
`78 (0x4e)`    astore\_3  
`79 (0x4f)`    iastore  
`80 (0x50)`    lastore  
`81 (0x51)`    fastore  
`82 (0x52)`    dastore  
`83 (0x53)`    aastore  
`84 (0x54)`    bastore  
`85 (0x55)`    castore  
`86 (0x56)`    sastore  
      



 |

  



**Table 7.2.** 

  
| Stack | Math | Conversions |
| :-: | :-: | :-: |
| 
  
`87 (0x57)`    pop  
`88 (0x58)`    pop2  
`89 (0x59)`    dup  
`90 (0x5a)`    dup\_x1  
`91 (0x5b)`    dup\_x2  
`92 (0x5c)`    dup2  
`93 (0x5d)`    dup2\_x1  
`94 (0x5e)`    dup2\_x2  
`95 (0x5f)`    swap  
      



 | 

  
 `96 (0x60)`    iadd  
 `97 (0x61)`    ladd  
 `98 (0x62)`    fadd  
 `99 (0x63)`    dadd  
`100 (0x64)`    isub  
`101 (0x65)`    lsub  
`102 (0x66)`    fsub  
`103 (0x67)`    dsub  
`104 (0x68)`    imul  
`105 (0x69)`    lmul  
`106 (0x6a)`    fmul  
`107 (0x6b)`    dmul  
`108 (0x6c)`    idiv  
`109 (0x6d)`    ldiv  
`110 (0x6e)`    fdiv  
`111 (0x6f)`    ddiv  
`112 (0x70)`    irem  
`113 (0x71)`    lrem  
`114 (0x72)`    frem  
`115 (0x73)`    drem  
`116 (0x74)`    ineg  
`117 (0x75)`    lneg  
`118 (0x76)`    fneg  
`119 (0x77)`    dneg  
`120 (0x78)`    ishl  
`121 (0x79)`    lshl  
`122 (0x7a)`    ishr  
`123 (0x7b)`    lshr  
`124 (0x7c)`    iushr  
`125 (0x7d)`    lushr  
`126 (0x7e)`    iand  
`127 (0x7f)`    land  
`128 (0x80)`    ior  
`129 (0x81)`    lor  
`130 (0x82)`    ixor  
`131 (0x83)`    lxor  
`132 (0x84)`    iinc  
      



 | 

  
`133 (0x85)`    i2l  
`134 (0x86)`    i2f  
`135 (0x87)`    i2d  
`136 (0x88)`    l2i  
`137 (0x89)`    l2f  
`138 (0x8a)`    l2d  
`139 (0x8b)`    f2i  
`140 (0x8c)`    f2l  
`141 (0x8d)`    f2d  
`142 (0x8e)`    d2i  
`143 (0x8f)`    d2l  
`144 (0x90)`    d2f  
`145 (0x91)`    i2b  
`146 (0x92)`    i2c  
`147 (0x93)`    i2s  
      



 |

  



**Table 7.3.** 

<table class="table" border="0"><colgroup><col> <col></colgroup><tbody><tr><td><div class="table"><a name="d5e25409" href="javase/specs/jvms/se19/html/jvms-7.html"></a><p class="title"><b>Table&nbsp;7.4.&nbsp;</b></p><div class="table-contents"><table class="table" border="0"><colgroup><col></colgroup><thead><tr><th align="center">Comparisons</th></tr></thead><tbody><tr><td><div class="literallayout"><p><br><code class="literal">148&nbsp;(0x94)</code>&nbsp;&nbsp;&nbsp;&nbsp;lcmp<br><code class="literal">149&nbsp;(0x95)</code>&nbsp;&nbsp;&nbsp;&nbsp;fcmpl<br><code class="literal">150&nbsp;(0x96)</code>&nbsp;&nbsp;&nbsp;&nbsp;fcmpg<br><code class="literal">151&nbsp;(0x97)</code>&nbsp;&nbsp;&nbsp;&nbsp;dcmpl<br><code class="literal">152&nbsp;(0x98)</code>&nbsp;&nbsp;&nbsp;&nbsp;dcmpg<br><code class="literal">153&nbsp;(0x99)</code>&nbsp;&nbsp;&nbsp;&nbsp;ifeq<br><code class="literal">154&nbsp;(0x9a)</code>&nbsp;&nbsp;&nbsp;&nbsp;ifne<br><code class="literal">155&nbsp;(0x9b)</code>&nbsp;&nbsp;&nbsp;&nbsp;iflt<br><code class="literal">156&nbsp;(0x9c)</code>&nbsp;&nbsp;&nbsp;&nbsp;ifge<br><code class="literal">157&nbsp;(0x9d)</code>&nbsp;&nbsp;&nbsp;&nbsp;ifgt<br><code class="literal">158&nbsp;(0x9e)</code>&nbsp;&nbsp;&nbsp;&nbsp;ifle<br><code class="literal">159&nbsp;(0x9f)</code>&nbsp;&nbsp;&nbsp;&nbsp;if_icmpeq<br><code class="literal">160&nbsp;(0xa0)</code>&nbsp;&nbsp;&nbsp;&nbsp;if_icmpne<br><code class="literal">161&nbsp;(0xa1)</code>&nbsp;&nbsp;&nbsp;&nbsp;if_icmplt<br><code class="literal">162&nbsp;(0xa2)</code>&nbsp;&nbsp;&nbsp;&nbsp;if_icmpge<br><code class="literal">163&nbsp;(0xa3)</code>&nbsp;&nbsp;&nbsp;&nbsp;if_icmpgt<br><code class="literal">164&nbsp;(0xa4)</code>&nbsp;&nbsp;&nbsp;&nbsp;if_icmple<br><code class="literal">165&nbsp;(0xa5)</code>&nbsp;&nbsp;&nbsp;&nbsp;if_acmpeq<br><code class="literal">166&nbsp;(0xa6)</code>&nbsp;&nbsp;&nbsp;&nbsp;if_acmpne<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p></div></td></tr></tbody></table><table class="table" border="0"><colgroup><col></colgroup><thead><tr><th align="center">Control</th></tr></thead><tbody><tr><td><div class="literallayout"><p><br><code class="literal">167&nbsp;(0xa7)</code>&nbsp;&nbsp;&nbsp;&nbsp;goto<br><code class="literal">168&nbsp;(0xa8)</code>&nbsp;&nbsp;&nbsp;&nbsp;jsr<br><code class="literal">169&nbsp;(0xa9)</code>&nbsp;&nbsp;&nbsp;&nbsp;ret<br><code class="literal">170&nbsp;(0xaa)</code>&nbsp;&nbsp;&nbsp;&nbsp;tableswitch<br><code class="literal">171&nbsp;(0xab)</code>&nbsp;&nbsp;&nbsp;&nbsp;lookupswitch<br><code class="literal">172&nbsp;(0xac)</code>&nbsp;&nbsp;&nbsp;&nbsp;ireturn<br><code class="literal">173&nbsp;(0xad)</code>&nbsp;&nbsp;&nbsp;&nbsp;lreturn<br><code class="literal">174&nbsp;(0xae)</code>&nbsp;&nbsp;&nbsp;&nbsp;freturn<br><code class="literal">175&nbsp;(0xaf)</code>&nbsp;&nbsp;&nbsp;&nbsp;dreturn<br><code class="literal">176&nbsp;(0xb0)</code>&nbsp;&nbsp;&nbsp;&nbsp;areturn<br><code class="literal">177&nbsp;(0xb1)</code>&nbsp;&nbsp;&nbsp;&nbsp;return<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p></div></td></tr></tbody></table></div></div><br class="table-break"></td><td><div class="table"><a name="d5e25457" href="javase/specs/jvms/se19/html/jvms-7.html"></a><p class="title"><b>Table&nbsp;7.5.&nbsp;</b></p><div class="table-contents"><table class="table" border="0"><colgroup><col></colgroup><thead><tr><th align="center">References</th></tr></thead><tbody><tr><td><div class="literallayout"><p><br><code class="literal">178&nbsp;(0xb2)</code>&nbsp;&nbsp;&nbsp;&nbsp;getstatic<br><code class="literal">179&nbsp;(0xb3)</code>&nbsp;&nbsp;&nbsp;&nbsp;putstatic<br><code class="literal">180&nbsp;(0xb4)</code>&nbsp;&nbsp;&nbsp;&nbsp;getfield<br><code class="literal">181&nbsp;(0xb5)</code>&nbsp;&nbsp;&nbsp;&nbsp;putfield<br><code class="literal">182&nbsp;(0xb6)</code>&nbsp;&nbsp;&nbsp;&nbsp;invokevirtual<br><code class="literal">183&nbsp;(0xb7)</code>&nbsp;&nbsp;&nbsp;&nbsp;invokespecial<br><code class="literal">184&nbsp;(0xb8)</code>&nbsp;&nbsp;&nbsp;&nbsp;invokestatic<br><code class="literal">185&nbsp;(0xb9)</code>&nbsp;&nbsp;&nbsp;&nbsp;invokeinterface<br><code class="literal">186&nbsp;(0xba)</code>&nbsp;&nbsp;&nbsp;&nbsp;invokedynamic<br><code class="literal">187&nbsp;(0xbb)</code>&nbsp;&nbsp;&nbsp;&nbsp;new<br><code class="literal">188&nbsp;(0xbc)</code>&nbsp;&nbsp;&nbsp;&nbsp;newarray<br><code class="literal">189&nbsp;(0xbd)</code>&nbsp;&nbsp;&nbsp;&nbsp;anewarray<br><code class="literal">190&nbsp;(0xbe)</code>&nbsp;&nbsp;&nbsp;&nbsp;arraylength<br><code class="literal">191&nbsp;(0xbf)</code>&nbsp;&nbsp;&nbsp;&nbsp;athrow<br><code class="literal">192&nbsp;(0xc0)</code>&nbsp;&nbsp;&nbsp;&nbsp;checkcast<br><code class="literal">193&nbsp;(0xc1)</code>&nbsp;&nbsp;&nbsp;&nbsp;instanceof<br><code class="literal">194&nbsp;(0xc2)</code>&nbsp;&nbsp;&nbsp;&nbsp;monitorenter<br><code class="literal">195&nbsp;(0xc3)</code>&nbsp;&nbsp;&nbsp;&nbsp;monitorexit<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p></div></td></tr></tbody></table><table class="table" border="0"><colgroup><col></colgroup><thead><tr><th align="center">Extended</th></tr></thead><tbody><tr><td><div class="literallayout"><p><br><code class="literal">196&nbsp;(0xc4)</code>&nbsp;&nbsp;&nbsp;&nbsp;wide<br><code class="literal">197&nbsp;(0xc5)</code>&nbsp;&nbsp;&nbsp;&nbsp;multianewarray<br><code class="literal">198&nbsp;(0xc6)</code>&nbsp;&nbsp;&nbsp;&nbsp;ifnull<br><code class="literal">199&nbsp;(0xc7)</code>&nbsp;&nbsp;&nbsp;&nbsp;ifnonnull<br><code class="literal">200&nbsp;(0xc8)</code>&nbsp;&nbsp;&nbsp;&nbsp;goto_w<br><code class="literal">201&nbsp;(0xc9)</code>&nbsp;&nbsp;&nbsp;&nbsp;jsr_w<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p></div></td></tr></tbody></table><table class="table" border="0"><colgroup><col></colgroup><thead><tr><th align="center">Reserved</th></tr></thead><tbody><tr><td><div class="literallayout"><p><br><code class="literal">202&nbsp;(0xca)</code>&nbsp;&nbsp;&nbsp;&nbsp;breakpoint<br><code class="literal">254&nbsp;(0xfe)</code>&nbsp;&nbsp;&nbsp;&nbsp;impdep1<br><code class="literal">255&nbsp;(0xff)</code>&nbsp;&nbsp;&nbsp;&nbsp;impdep2<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p></div></td></tr></tbody></table></div></div><br class="table-break"></td></tr></tbody></table>

