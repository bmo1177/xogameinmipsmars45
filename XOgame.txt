.data
    board: .asciiz "123456789"  # Initial board with numbers 1-9
    prompt: .asciiz "Enter your move (1-9): "
    player1: .asciiz "Player 1 (X)"
    player2: .asciiz "Player 2 (O)"
    win_msg: .asciiz "Player "
    win_msg2: .asciiz " wins!\n"
    draw_msg: .asciiz "The game is a draw!\n"
    invalid_move_msg: .asciiz "Invalid move. Try again.\n"
    newline: .asciiz "\n"
    space: .asciiz " "

.text
.globl main

main:
    li $t2, 0          # Current player (0 for Player 1, 1 for Player 2)
    li $t5, 0          # Total move counter
    li $s0, 0          # X counter

game_loop:
    # Print the board
    jal print_board

    # Check if 5 Xs are on the board
    beq $s0, 5, draw

    # Determine current player
    beq $t2, 0, player1_prompt
    la $a0, player2
    j player_prompt

player1_prompt:
    la $a0, player1

player_prompt:
    # Check if board is full before prompting
    beq $t5, 9, draw

    # Print player prompt
    li $v0, 4
    syscall

    la $a0, prompt
    li $v0, 4
    syscall

    # Get player input
    li $v0, 5
    syscall
    move $t3, $v0

    # Validate input
    blt $t3, 1, invalid_input
    bgt $t3, 9, invalid_input

    # Adjust index to zero-based
    addi $t3, $t3, -1

    # Check if space is already occupied
    la $t1, board
    add $t1, $t1, $t3
    lb $t4, 0($t1)
    beq $t4, 'X', invalid_input
    beq $t4, 'O', invalid_input

    # Update the board
    beq $t2, 0, update_x
    li $t4, 'O'
    j update_board

update_x:
    li $t4, 'X'
    # Increment X counter only when placing an X
    addi $s0, $s0, 1

update_board:
    sb $t4, 0($t1)

    # Increment move counter
    addi $t5, $t5, 1

    # Check for win
    jal check_win
    beq $v0, 1, win

    # Check if board is full
    beq $t5, 9, draw

    # Switch player
    xori $t2, $t2, 1
    j game_loop

invalid_input:
    # Check if board is full
    beq $t5, 9, draw

    # Print invalid move message
    la $a0, invalid_move_msg
    li $v0, 4
    syscall
    j game_loop

win:
    # Print win message
    jal print_board   # Show final board state
    
    la $a0, win_msg
    li $v0, 4
    syscall

    # Determine which player won
    beq $t2, 0, print_player1_win
    la $a0, player2
    j print_win_msg

print_player1_win:
    la $a0, player1

print_win_msg:
    li $v0, 4
    syscall

    la $a0, win_msg2
    li $v0, 4
    syscall

    j exit

draw:
    # Print draw message
    jal print_board   # Show final board state
    
    la $a0, draw_msg
    li $v0, 4
    syscall

exit:
    # Exit the program
    li $v0, 10
    syscall

print_board:
    # Save return address
    move $t9, $ra

    # Print the board
    la $t1, board
    li $t0, 0

print_loop:
    lb $a0, 0($t1)
    li $v0, 11
    syscall

    addi $t0, $t0, 1
    beq $t0, 3, print_newline
    beq $t0, 6, print_newline
    beq $t0, 9, print_newline

    la $a0, space
    li $v0, 4
    syscall

    j next_cell

print_newline:
    la $a0, newline
    li $v0, 4
    syscall

next_cell:
    addi $t1, $t1, 1
    bne $t0, 9, print_loop

    # Restore return address
    move $ra, $t9
    jr $ra

check_win:
    la $t1, board
    li $v0, 0     # Return 0 (no win) by default

    # Check rows
    li $t0, 0
check_row:
    mul $t6, $t0, 3
    add $t7, $t1, $t6
    
    lb $t3, 0($t7)
    lb $t4, 1($t7)
    lb $t5, 2($t7)
    
    beq $t3, $t4, check_row_first_two
    j next_row
check_row_first_two:
    beq $t4, $t5, check_row_all_match
    j next_row
check_row_all_match:
    # Ensure not using initial numbers
    beq $t3, '1', next_row
    beq $t3, '2', next_row
    beq $t3, '3', next_row
    beq $t3, '4', next_row
    beq $t3, '5', next_row
    beq $t3, '6', next_row
    beq $t3, '7', next_row
    beq $t3, '8', next_row
    beq $t3, '9', next_row
    
    li $v0, 1
    jr $ra

next_row:
    addi $t0, $t0, 1
    bne $t0, 3, check_row

    # Check columns
    li $t0, 0
check_col:
    add $t7, $t1, $t0
    
    lb $t3, 0($t7)
    lb $t4, 3($t7)
    lb $t5, 6($t7)
    
    beq $t3, $t4, check_col_first_two
    j next_col
check_col_first_two:
    beq $t4, $t5, check_col_all_match
    j next_col
check_col_all_match:
    # Ensure not using initial numbers
    beq $t3, '1', next_col
    beq $t3, '2', next_col
    beq $t3, '3', next_col
    beq $t3, '4', next_col
    beq $t3, '5', next_col
    beq $t3, '6', next_col
    beq $t3, '7', next_col
    beq $t3, '8', next_col
    beq $t3, '9', next_col
    
    li $v0, 1
    jr $ra

next_col:
    addi $t0, $t0, 1
    bne $t0, 3, check_col

    # Check diagonals
    lb $t3, 0($t1)
    lb $t4, 4($t1)
    lb $t5, 8($t1)
    
    beq $t3, $t4, check_diag1_first_two
    j check_diag2
check_diag1_first_two:
    beq $t4, $t5, check_diag1_match
    j check_diag2
check_diag1_match:
    # Ensure not using initial numbers
    beq $t3, '1', check_diag2
    beq $t3, '2', check_diag2
    beq $t3, '3', check_diag2
    beq $t3, '4', check_diag2
    beq $t3, '5', check_diag2
    beq $t3, '6', check_diag2
    beq $t3, '7', check_diag2
    beq $t3, '8', check_diag2
    beq $t3, '9', check_diag2
    
    li $v0, 1
    jr $ra

check_diag2:
    lb $t3, 2($t1)
    lb $t4, 4($t1)
    lb $t5, 6($t1)
    
    beq $t3, $t4, check_diag2_first_two
    j no_win
check_diag2_first_two:
    beq $t4, $t5, check_diag2_match
    j no_win
check_diag2_match:
    # Ensure not using initial numbers
    beq $t3, '1', no_win
    beq $t3, '2', no_win
    beq $t3, '3', no_win
    beq $t3, '4', no_win
    beq $t3, '5', no_win
    beq $t3, '6', no_win
    beq $t3, '7', no_win
    beq $t3, '8', no_win
    beq $t3, '9', no_win
    
    li $v0, 1
    jr $ra

no_win:
    li $v0, 0
    jr $ra