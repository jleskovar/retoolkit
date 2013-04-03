retoolkit
=========

Scripting-based reverse engineering toolkit for x86 Windows.

## Introduction

Retoolkit is a powerful tool for performing real-time process analysis and modification. It encourages an agile/prototype-based approach to reverse engineering.

## Features

 - Ruby-scripting interface
 - Complete .NET library available from scripting interface
 - Intuitive and powerful, re-entrant hooking system
 - Ability to create and invoke x86 functions within ruby code
 - Ability to invoke arbitrary functions within the process memory space
 - Memory wrapper for process memory I/O
 - Can be used as either a stand-alone ruby interpreter, or injected into processes using supplied injector program.

## Screenshots

![Retoolkit](web/mboxhook_sml.png?raw=true)

## Requirements

 - Visual C++ 2008 redist
 - .NET 3.5 framework or greater
 - Windows XP or later

## Hooking example

Here, we're hooking the MessageBoxA API call of the parent process, which resides in the user32.dll module. We then use the Win32API library to call into this function, to test that our hook works.

    require 'retools'
    require 'pp'
    include Retools

    Hooker.hook('user32', 'MessageBoxA') { |h,r,f|
      # StackArgs is a memory wrapper used for easy
      # access to arguments on the stack at the hook-point
      stack = StackArgs[r]
      hwnd = stack[1].dword
      text = stack[2].ascii
      cap  = stack[3].ascii
      style = stack[4].dword
      
      stack[2].ptr = "hooked #{text}"
      
      true
    }

    mb = Win32API.new('user32', 'MessageBoxA', 'LPPL', 'L')

    $mytext = 'my text'
    $mycap = 'my caption'
    mb.call(0, $mytext, $mycap, 0)


The block passed to the Hooker.hook module function is the callback function for this hook. Execution passes through the block whenever the hooked function is invoked. The block takes in three arguments, the hook handle (h), a context structure (r), and EFLAGS register (f). The hook handle may be used to obtain information about the hook. The context structure contains the state of the CPU registers up to the point where the hook is installed. It can be used to read and write to and from registers. The EFLAGS argument is simply an integer containing the value of EFLAGS up to the hook-point. It can be read from and written to.

The hook block finally returns true to indicate that execution should resume from just after the hook-point. Returning false will simply x86-ret instead of resuming execution.

### x86 example

The following example makes use of the [metasm][metasm] ruby library for generating x86 code

    require 'retools'
    require 'asm'
    require 'pp'
    include Retools

    # create a new assembly function
    # here we're specifying our function follows (c)decl
    # convention, and takes in two integer ('l') parameters
    # all assembly functions implicitely use the value of EAX
    # as the return value
    myfunc = AsmFunc.new('c', 'll', <<ASM)
      mov eax, [esp+4]  // eax <- arg 1
      mov ecx, [esp+8]  // ecx <- arg 2
      add eax, ecx
      ret
    ASM

    # call our function
    p myfunc.call(32, 10) #=> 42


## Related projects

The following projects are used in Retoolkit:

 - [AsmJit](http://code.google.com/p/asmjit/)
 - [Metasm][metasm]
 - [IronRuby](http://www.ironruby.net/)
 - [libdisasm](http://bastard.sourceforge.net/libdisasm.html)
 - [ScintillaNET](http://scintillanet.codeplex.com/)

 [metasm]: http://code.google.com/p/metasm/
