# Decoding-native-handler-hooks

so you need a pretty small assembly function to do this,
1. some bit shifting magic
# Code:
bit_mix proc
	sub    rsp,18h
	mov    QWORD PTR [rsp+10h],rcx
	mov    rax,QWORD PTR [rsp+10h]
	mov    QWORD PTR [rsp],rax
	mov    rax,QWORD PTR [rsp]
	mov    ax,WORD PTR [rax]
	mov    WORD PTR [rsp+0eh],ax
	mov    rax,QWORD PTR [rsp]
	mov    ax,WORD PTR [rax+2]
	mov    WORD PTR [rsp+0ch],ax
	mov    rax,QWORD PTR [rsp]
	mov    ax,WORD PTR [rax+4]
	mov    WORD PTR [rsp+0ah],ax
	mov    rax,QWORD PTR [rsp]
	mov    ax,WORD PTR [rax+6]
	mov    WORD PTR [rsp+8],ax
	mov    ax,WORD PTR [rsp+0ah]
	mov    rcx,QWORD PTR [rsp]
	mov    WORD PTR [rcx],ax
	mov    ax,WORD PTR [rsp+0eh]
	mov    rcx,QWORD PTR [rsp]
	mov    WORD PTR [rcx+4],ax
	mov    ax,WORD PTR [rsp+8]
	mov    rcx,QWORD PTR [rsp]
	mov    WORD PTR [rcx+2],ax
	mov    ax,WORD PTR [rsp+0ch]
	mov    rcx,QWORD PTR [rsp]
	mov    WORD PTR [rcx+6],ax
	add    rsp,18h
	ret
bit_mix endp

2. the actual logic (you need to build this at runtime, the constants are just poc from my dump)
# Code:
decode_native_handler proc
    sub     rsp, 8h
    mov     [rsp], rcx
    lea     rcx, [rsp]
    call    bit_mix
 
    mov     rax, 07fffd42b4508h ; adhesive + 0x32b4508
    mov     rcx, 07fffd3e0a670h ; adhesive + 0x2e0a670
    xor     rcx, rax
    mov		rdx, 07fffd42b4500h ; adhesive + 0x32b4500
    xor     rdx, rcx
    mov		rax, [rsp]
    xor     rax, rdx
    not     rax
    shld    rax, rax, 20h
    mov		rcx, 07fffd42b44f8h ; adhesive + 0x32b44f8
    xor     rax, rcx
    add     rsp, 8h
    ret
decode_native_handler endp

example of use:
# Code:
extern "C" std::uintptr_t decode_native_handler(std::uint64_t stub_rdx);
 
auto handler_trampoline = decode_native_handler(0x2C1F598FE721CF78); // -> 0x7ff7ccf50b80
disassembly of 0x7ff7ccf50b80
Code:
0:000> u 0x7ff7ccf50b80
00007ff7`ccf50b80 ff2500000000    jmp     qword ptr [00007ff7`ccf50b86]
 
0:000> dqs 00007ff7`ccf50b86
00007ff7`ccf50b86  00007ff7`d0ac4f99 FiveM_b3407_GTAProcess+0x3a84f99
