0xobi

org 100h

;----------------------------------
; Dati
;----------------------------------
message     db '0xobi'
msg_len     equ $ - message

; Alcuni colori "arcobaleno" (foreground, bg nero)
; 04 = red, 06 = brown, 0E = yellow, 0A = light green,
; 09 = light blue, 0D = magenta, 05 = purple
colors      db 04h, 06h, 0Eh, 0Ah, 09h, 0Dh, 05h
num_colors  equ $ - colors

;----------------------------------
; Codice
;----------------------------------
start:
    ; Segmento video testo a colori
    mov     ax, 0B800h
    mov     es, ax

    ; DS = CS per accedere ai dati
    mov     ax, cs
    mov     ds, ax

    ; Posizioniamo la scritta in riga 12, colonna 37 (0-based)
    mov     dh, 12          ; riga
    mov     dl, 37          ; colonna

    mov     al, 80          ; 80 colonne per riga
    mov     bl, dh          ; BL = riga
    mul     bl              ; AX = riga * 80

    xor     bh, bh
    mov     bl, dl          ; BL = colonna
    add     ax, bx          ; AX = riga*80 + colonna

    shl     ax, 1           ; *2 (carattere + attributo)
    mov     di, ax          ; DI = offset in video memory

    xor     si, si          ; SI = "shift" dei colori

main_loop:
    ; Scrivi la stringa con colori che dipendono da (indice + shift)
    mov     cx, msg_len     ; CX = contatore caratteri
    xor     bx, bx          ; BX = indice carattere (0..msg_len-1)

write_loop:
    ; Carattere
    mov     al, [message + bx]
    mov     es:[di + bx*2], al

    ; Calcolo dell'indice del colore: (bx + si) mod num_colors
    mov     dx, bx
    add     dx, si
    mov     ax, dx

mod_loop:
    cmp     ax, num_colors
    jb      mod_done
    sub     ax, num_colors
    jmp     mod_loop
mod_done:
    ; AX ora è tra 0 e num_colors-1
    mov     dl, [colors + ax]
    mov     es:[di + bx*2 + 1], dl

    inc     bx
    loop    write_loop

    ; Avanza lo "shift" dei colori per l'effetto arcobaleno animato
    inc     si
    cmp     si, num_colors
    jb      shift_ok
    xor     si, si          ; ricomincia da 0
shift_ok:

    ; Piccolo delay per vedere l'animazione
    mov     cx, 30000
delay_outer:
    nop
    loop    delay_outer

    jmp     main_loop       ; ciclo infinito (ESC non gestito)

; Fine programma (mai raggiunta)