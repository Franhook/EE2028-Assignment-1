/*
* iir.s
*
*  Created on: 29/7/2025
*      Author: Ni Qingqing
*/
   .syntax unified
	.cpu cortex-m4
	.fpu softvfp
	.thumb

		.global iir

@ Start of executable code
.section .text

.equ N_MAX, 10 @Define N_MAX = 10
.lcomm x_store, 4*N_MAX
.lcomm y_store, 4*N_MAX

@ EE2028 Assignment 1, Sem 1, AY 2025/26
@ (c) ECE NUS, 2025

@ Write Student 1’s Name here: Matthew Goh Jin Han
@ Write Student 2’s Name here: Jeremy Lim Zhen Xing

@ You could create a look-up table of registers here:

@ int iir(int N, int* b, int* a, int x_n)

@ r0  = accumulator y_n (pre-scale), also return register
@ r1  = holds b0 initially, later b[j+1], y[j], and divisor 100
@ r2  = loop counter j (0..N-1), later reused as shift index
@ r3  = used for offsets (4*(j+1), 4*j) and a[j+1]
@ r4  = N (filter order)
@ r5  = base pointer to b coefficients
@ r6  = base pointer to a coefficients
@ r7  = current input x_n
@ r8  = base address of x_store
@ r9  = base address of y_store
@ r10 = a0, constant divisor for all calculations
@ r11 = used for offsets, y_store[j], and shift data
@ r12 = used for x_store[j] and offset_prev in shift loop

@ write your program from here:

iir:
    push {r4-r12, lr}       @ save callee-saved + temp registers

    mov r4, r0              @ r4 = N
    mov r5, r1              @ r5 = b base
    mov r6, r2              @ r6 = a base
    mov r7, r3              @ r7 = x_n

    ldr r8, =x_store        @ base of x_store
    ldr r9, =y_store        @ base of y_store

    ldr r10, [r6]           @ a0
    ldr r1, [r5]            @ b0

    mul r0, r7, r1          @ y_n = x_n * b0
    sdiv r0, r0, r10        @ divide by a0

    movs r2, #0             @ loop counter j = 0

loop_j:
    cmp r2, r4
    bge loop_j_end

    add r3, r2, #1
    lsl r3, r3, #2          @ offset = 4*(j+1)
    ldr r1, [r5, r3]        @ b[j+1]
    ldr r3, [r6, r3]        @ a[j+1]

    lsl r11, r2, #2         @ offset = 4*j
    ldr r12, [r8, r11]      @ x_store[j]
    ldr r11, [r9, r11]      @ y_store[j]

    mul r1, r1, r12         @ b*x
    mul r3, r3, r11         @ a*y
    subs r1, r1, r3         @ b*x - a*y
    sdiv r1, r1, r10        @ divide by a0
    add r0, r0, r1          @ accumulate

    adds r2, r2, #1
    b loop_j
loop_j_end:

    mov r2, r4
    subs r2, r2, #1         @ j = N-1

shift_loop:
    cmp r2, #0
    ble shift_end

    lsl r3, r2, #2          @ offset = 4*j
    sub r12, r2, #1
    lsl r12, r12, #2        @ offset_prev = 4*(j-1)

    ldr r11, [r8, r12]
    str r11, [r8, r3]       @ x[j] = x[j-1]
    ldr r11, [r9, r12]
    str r11, [r9, r3]       @ y[j] = y[j-1]

    subs r2, r2, #1
    b shift_loop
shift_end:

    str r7, [r8]            @ newest x_n
    str r0, [r9]            @ newest y_n (pre-scale, not yet divided)

    movs r1, #100
    sdiv r0, r0, r1         @ scale y_n

    pop {r4-r12, lr}
    bx lr

