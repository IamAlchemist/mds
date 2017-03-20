### Error

```
Unable to simultaneously satisfy constraints.
    Probably at least one of the constraints in the following list is one you don't want. Try this: (1) look at each constraint and try to figure out which you don't expect; (2) find the code that added the unwanted constraint or constraints and fix it. (Note: If you're seeing NSAutoresizingMaskLayoutConstraints that you don't understand, refer to the documentation for the UIView property translatesAutoresizingMaskIntoConstraints)
(
    "<NSLayoutConstraint:0x7fc82d3e18a0 H:[UIView:0x7fc82aba1210(768)]>",
    "<NSLayoutConstraint:0x7fc82d6369e0 H:[UIView:0x7fc82aba1210]-(0)-|   (Names: '|':UIView:0x7fc82d6b9f80 )>",
    "<NSLayoutConstraint:0x7fc82d636a30 H:|-(0)-[UIView:0x7fc82aba1210]   (Names: '|':UIView:0x7fc82d6b9f80 )>",
    "<NSLayoutConstraint:0x7fc82d3e7fd0 'UIView-Encapsulated-Layout-Width' H:[UIView:0x7fc82d6b9f80(50)]>"
)

Will attempt to recover by breaking constraint
<NSLayoutConstraint:0x7fc82d3e18a0 H:[UIView:0x7fc82aba1210(768)]>

Make a symbolic breakpoint at UIViewAlertForUnsatisfiableConstraints to catch this in the debugger.
The methods in the UIConstraintBasedLayoutDebugging category on UIView listed in <UIKit/UIView.h> may also be helpful.
```

### Set AutoLayout breakpoint

Make a symbolic breakpoint at `UIViewAlertForUnsatisfiableConstraints` to catch this in the debugger.

you will see things like this:

```
UIKit`UIViewAlertForUnsatisfiableConstraints:
->  0x1131cc4a2 <+0>:   pushq  %rbp
    0x1131cc4a3 <+1>:   movq   %rsp, %rbp
    0x1131cc4a6 <+4>:   pushq  %r15
    0x1131cc4a8 <+6>:   pushq  %r14
    0x1131cc4aa <+8>:   pushq  %rbx
    0x1131cc4ab <+9>:   pushq  %rax
    0x1131cc4ac <+10>:  movq   %rsi, %r14
    0x1131cc4af <+13>:  movq   %rdi, %r15
    0x1131cc4b2 <+16>:  cmpq   $-0x1, 0x778796(%rip)     ; _UIConstraintBasedLayoutPlaySoundOnUnsatisfiable.result + 7
    0x1131cc4ba <+24>:  jne    0x1131cc529               ; <+135>
    0x1131cc4bc <+26>:  cmpb   $0x0, 0x778785(%rip)      ; _UIConstraintBasedLayoutPlaySoundWhenEngaged.__once + 7
    0x1131cc4c3 <+33>:  je     0x1131cc4ed               ; <+75>
    0x1131cc4c5 <+35>:  movq   0x73cd8c(%rip), %rdi      ; (void *)0x0000000113925248: UIDevice
    0x1131cc4cc <+42>:  movq   0x7108e5(%rip), %rsi      ; "currentDevice"
    0x1131cc4d3 <+49>:  movq   0x4d7aee(%rip), %rbx      ; (void *)0x0000000114598000: objc_msgSend
    0x1131cc4da <+56>:  callq  *%rbx
    0x1131cc4dc <+58>:  movq   0x71e375(%rip), %rsi      ; "_playSystemSound:"
    0x1131cc4e3 <+65>:  movl   $0x3ee, %edx
    0x1131cc4e8 <+70>:  movq   %rax, %rdi
    0x1131cc4eb <+73>:  callq  *%rbx
    0x1131cc4ed <+75>:  cmpq   $-0x1, 0x77877b(%rip)     ; _UIConstraintBasedLayoutLogUnsatisfiable.result + 7
    0x1131cc4f5 <+83>:  jne    0x1131cc541               ; <+159>
    0x1131cc4f7 <+85>:  cmpb   $0x0, 0x77876a(%rip)      ; _UIConstraintBasedLayoutVisualizeMutuallyExclusiveConstraints.__once + 7
    0x1131cc4fe <+92>:  je     0x1131cc51e               ; <+124>
    0x1131cc500 <+94>:  leaq   0x53ba89(%rip), %rdi      ; @"Unable to simultaneously satisfy constraints.\n\tProbably at least one of the constraints in the following list is one you don't want. Try this: (1) look at each constraint and try to figure out which you don't expect; (2) find the code that added the unwanted constraint or constraints and fix it. (Note: If you're seeing NSAutoresizingMaskLayoutConstraints that you don't understand, refer to the documentation for the UIView property translatesAutoresizingMaskIntoConstraints) \n%@\n\nWill attempt to recover by breaking constraint \n%@\n\nMake a symbolic breakpoint at UIViewAlertForUnsatisfiableConstraints to catch this in the debugger.\nThe methods in the UIConstraintBasedLayoutDebugging category on UIView listed in <UIKit/UIView.h> may also be helpful."
    0x1131cc507 <+101>: xorl   %eax, %eax
    0x1131cc509 <+103>: movq   %r14, %rsi
    0x1131cc50c <+106>: movq   %r15, %rdx
    0x1131cc50f <+109>: addq   $0x8, %rsp
    0x1131cc513 <+113>: popq   %rbx
    0x1131cc514 <+114>: popq   %r14
    0x1131cc516 <+116>: popq   %r15
    0x1131cc518 <+118>: popq   %rbp
    0x1131cc519 <+119>: jmp    0x11331930c               ; symbol stub for: NSLog
    0x1131cc51e <+124>: addq   $0x8, %rsp
    0x1131cc522 <+128>: popq   %rbx
    0x1131cc523 <+129>: popq   %r14
    0x1131cc525 <+131>: popq   %r15
    0x1131cc527 <+133>: popq   %rbp
    0x1131cc528 <+134>: retq
    0x1131cc529 <+135>: leaq   0x778720(%rip), %rdi      ; _UIConstraintBasedLayoutPlaySoundOnUnsatisfiable.__once
    0x1131cc530 <+142>: leaq   0x4fdb59(%rip), %rsi      ; __block_literal_global68
    0x1131cc537 <+149>: callq  0x11331a2a8               ; symbol stub for: dispatch_once
    0x1131cc53c <+154>: jmp    0x1131cc4bc               ; <+26>
    0x1131cc541 <+159>: leaq   0x778728(%rip), %rdi      ; _UIConstraintBasedLayoutLogUnsatisfiable.__once
    0x1131cc548 <+166>: leaq   0x4fdbc1(%rip), %rsi      ; __block_literal_global76
    0x1131cc54f <+173>: callq  0x11331a2a8               ; symbol stub for: dispatch_once
    0x1131cc554 <+178>: jmp    0x1131cc4f7               ; <+85>
```

### Printing views from memory addresses

`Will attempt to recover by breaking constraint <NSLayoutConstraint:0x7fc82d3e18a0 H:[UIView:0x7fc82aba1210(768)]>`

`po 0x7fc82aba1210`

If this doesn't provide enough info as the below sample output shows (not very helpful).

```
(lldb) po 0x7fc82aba1210
<UIView: 0x7fc82aba1210; frame = (0 0; 768 359); autoresize = RM+BM; layer = <CALayer: 0x7fc82d338960>>
```

Maybe printing it's recursiveDescription will help out and in my case gives me a better idea of which component I'm actually trying to look at.

```
(lldb) po [0x7fc82aba1210 recursiveDescription]
<UIView: 0x7fc82aba1210; frame = (0 0; 768 359); autoresize = RM+BM; layer = <CALayer: 0x7fc82d338960>>
   | <MYAPPButton: 0x7fc82d61c800; baseClass = UIButton; frame = (0 0; 768 359); opaque = NO; autoresize = RM+BM; layer = <CALayer: 0x7fc82ab96570>>
   |    | <UIImageView: 0x7fc82d3f7b10; frame = (0 0; 0 0); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x7fc82d6e7020>>
   | <UIImageView: 0x7fc82d54a1c0; frame = (314 110; 140 140); autoresize = RM+BM; userInteractionEnabled = NO; layer = <CALayer: 0x7fc82d52def0>>
```

Or if that's not enough context, then we can look at the view's superview and potentially even walk up the view's tree, printing out a different levels trying to understand which component is having trouble with auto layout.

```
po [[0x7fc82aba1210 superview] recursiveDescription]
```
