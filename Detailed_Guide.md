# AI Chess Game - Detailed Implementation Guide

Welcome to this detailed guide to the AI Chess game! This document will help you understand how the game works, its key features, and break down the important functions in the codebase. This guide is designed for beginners who want to understand the implementation of a classic board game with an intelligent AI opponent.

## Table of Contents

1. [Game Overview](#game-overview)
2. [Project Structure](#project-structure)
3. [HTML Structure](#html-structure)
4. [CSS Styling](#css-styling)
5. [JavaScript Implementation](#javascript-implementation)
   - [Game Setup and State Management](#game-setup-and-state-management)
   - [Board Rendering](#board-rendering)
   - [User Interaction](#user-interaction)
   - [Chess Rules Implementation](#chess-rules-implementation)
   - [AI Implementation](#ai-implementation)
6. [Deep Dive into the Minimax Algorithm](#deep-dive-into-the-minimax-algorithm)

## Game Overview

This is a Chess game where you play against an AI opponent that uses the minimax algorithm to make strategic moves. The game features:

- A clean, visually appealing chessboard with standard piece notation
- Player vs. AI gameplay with the human playing as White
- Visual feedback for selected pieces and valid moves
- Multiple difficulty levels through adjustable AI search depth
- Check and checkmate detection
- Stalemate recognition
- Piece promotion (pawns to queens)

In this implementation:
- You play as White and move first
- The AI plays as Black and responds to your moves
- The game follows standard chess rules
- The AI's strength can be adjusted using the difficulty selector

## Project Structure

The project follows a simple structure:

```
chess/
├── index.html        # Main HTML file with game structure
├── styles.css        # Styling for the game interface
├── script.js         # Game logic and AI implementation
└── README.md         # Basic information about the project
```

## HTML Structure

The `index.html` file sets up the basic structure for the chess game:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Chess Game</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>AI Chess Game</h1>
        <div class="game-info">
            <div id="status">Your turn (White)</div>
            <button id="restart-btn">Restart Game</button>
        </div>
        <div id="board" class="chess-board"></div>
        <div class="settings">
            <label for="difficulty">AI Difficulty (Search Depth):</label>
            <select id="difficulty">
                <option value="1">Easy (Depth 1)</option>
                <option value="2" selected>Medium (Depth 2)</option>
                <option value="3">Hard (Depth 3)</option>
            </select>
        </div>
    </div>
    <script src="script.js"></script>
</body>
</html>
```

Key components:
- **Game Title**: Simple heading that identifies the game
- **Game Info Area**: Contains the status message and restart button
- **Chessboard Container**: An empty div that will be populated with the chess squares
- **Difficulty Selector**: Dropdown menu to adjust the AI's search depth
- **Script Reference**: Links to the JavaScript file containing game logic

## CSS Styling

The `styles.css` file provides styling for the chess game interface. The styling includes:

- **Layout**: Centered container with responsive design
- **Chessboard Design**: Grid-based layout with alternating colors
- **Visual Feedback**: Highlights for selected pieces and valid moves
- **User Interface**: Styling for buttons, status display, and settings

### Key CSS Features

```css
body {
    font-family: Arial, sans-serif;
    background-color: #f0f0f0;
    display: flex;
    justify-content: center;
    padding: 20px;
}

.container {
    max-width: 600px;
    width: 100%;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    padding: 20px;
}

.chess-board {
    display: grid;
    grid-template-columns: repeat(8, 1fr);
    grid-template-rows: repeat(8, 1fr);
    width: 100%;
    aspect-ratio: 1 / 1;
    border: 2px solid #333;
    margin-bottom: 20px;
}

.square {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2rem;
    cursor: pointer;
    transition: background-color 0.2s;
}

.white {
    background-color: #f0d9b5;
}

.black {
    background-color: #b58863;
}

.square.selected {
    background-color: #aec6cf;
}

.square.valid-move::after {
    content: "";
    position: absolute;
    width: 25%;
    height: 25%;
    background-color: rgba(0, 0, 0, 0.2);
    border-radius: 50%;
}
```

Key styling features:
- **Grid Layout**: The chessboard uses CSS Grid to create an 8x8 board
- **Square Styling**: Different styles for white and black squares
- **Selected Piece**: Visual feedback when a piece is selected
- **Valid Move Indicator**: A subtle dot indicates valid move locations
- **Responsive Design**: The board scales with the container width

## JavaScript Implementation

The `script.js` file contains all the game logic and AI implementation. Let's explore the key components:

### Game Setup and State Management

```javascript
// Chess pieces Unicode characters
const PIECES = {
    'white': {
        'pawn': '♙',
        'rook': '♖',
        'knight': '♘',
        'bishop': '♗',
        'queen': '♕',
        'king': '♔'
    },
    'black': {
        'pawn': '♟',
        'rook': '♜',
        'knight': '♞',
        'bishop': '♝',
        'queen': '♛',
        'king': '♚'
    }
};

// Initial board setup
const INITIAL_BOARD = [
    ['br', 'bn', 'bb', 'bq', 'bk', 'bb', 'bn', 'br'],
    ['bp', 'bp', 'bp', 'bp', 'bp', 'bp', 'bp', 'bp'],
    ['', '', '', '', '', '', '', ''],
    ['', '', '', '', '', '', '', ''],
    ['', '', '', '', '', '', '', ''],
    ['', '', '', '', '', '', '', ''],
    ['wp', 'wp', 'wp', 'wp', 'wp', 'wp', 'wp', 'wp'],
    ['wr', 'wn', 'wb', 'wq', 'wk', 'wb', 'wn', 'wr']
];

// Game state variables
let board = [];
let selectedPiece = null;
let currentPlayer = 'white';
let validMoves = [];
let gameOver = false;
```

This section establishes the foundation of the game:
- **Piece Representation**: Unicode characters for visual display of pieces
- **Board Representation**: A 2D array where each cell is either empty or contains a piece code
- **Game State Variables**: Track the current state of the game (selected pieces, current player, valid moves)
- **Piece Encoding**: Each piece is encoded as a two-character string where the first character is the color ('w' or 'b') and the second is the piece type ('p', 'r', 'n', 'b', 'q', 'k')

### Board Rendering

```javascript
function renderBoard() {
    boardElement.innerHTML = '';

    for (let row = 0; row < 8; row++) {
        for (let col = 0; col < 8; col++) {
            const square = document.createElement('div');
            square.className = `square ${(row + col) % 2 === 0 ? 'white' : 'black'}`;
            square.dataset.row = row;
            square.dataset.col = col;

            if (board[row][col]) {  // If there's a piece on this square
                const piece = board[row][col];
                const color = piece[0] === 'w' ? 'white' : 'black';
                const type = getPieceType(piece[1]);
                square.textContent = PIECES[color][type];
            }

            if (selectedPiece && selectedPiece.row === row && selectedPiece.col === col) {
                square.classList.add('selected');
            }

            if (validMoves.some(move => move.row === row && move.col === col)) {
                square.classList.add('valid-move');
            }

            square.addEventListener('click', () => handleSquareClick(row, col));
            boardElement.appendChild(square);
        }
    }
}
```

This function creates and updates the visual representation of the chessboard:
1. Clears the existing board
2. Creates 64 div elements representing the squares
3. Adds appropriate classes for coloring (alternating white and black)
4. Places chess piece Unicode characters on squares with pieces
5. Highlights the selected piece (if any)
6. Shows valid move indicators
7. Attaches click event handlers to each square

### User Interaction

```javascript
function handleSquareClick(row, col) {
    if (gameOver || currentPlayer === 'black') return;

    const piece = board[row][col]; // Get the piece at the clicked square
    
    // If a piece is already selected
    if (selectedPiece) {
        // Check if clicked on a valid move
        const moveIndex = validMoves.findIndex(move => move.row === row && move.col === col);
        
        if (moveIndex !== -1) {
            // Make the move
            const move = validMoves[moveIndex];
            makeMove(selectedPiece.row, selectedPiece.col, move.row, move.col);
            
            // Reset selection
            selectedPiece = null;
            validMoves = [];
            
            // Check if game is over
            if (isCheckmate('black')) {
                gameOver = true;
                statusElement.textContent = 'Checkmate! You win!';
                return;
            }
            
            // AI's turn
            currentPlayer = 'black';
            statusElement.textContent = 'AI is thinking...';
            renderBoard();
            
            // Allow UI to update before AI makes a move
            setTimeout(makeAIMove, 500);
            return;
        }
        
        // Clicked on another square, reset selection
        selectedPiece = null;
        validMoves = [];
    }
    
    // Select a piece if it belongs to the current player
    if (piece && piece[0] === 'w') {
        selectedPiece = { row, col };
        validMoves = getValidMoves(row, col);
    }
    
    renderBoard();
}
```

This function handles the player's interaction with the board:
1. Prevents interaction when it's not the player's turn or the game is over
2. If a piece is already selected and the player clicks on a valid move square, it moves the piece
3. If the player clicks elsewhere, it deselects the current piece
4. If the player clicks on their own piece, it selects that piece and calculates valid moves
5. After the player makes a move, it checks for checkmate and switches to the AI's turn

### Chess Rules Implementation

The game implements all standard chess rules through several key functions:

#### Piece Movement Functions

Each chess piece has its own movement function that calculates potential moves:

```javascript
function getPawnMoves(row, col, color, moves) {
    const direction = color === 'w' ? -1 : 1;
    const startRow = color === 'w' ? 6 : 1;
    
    // Move forward one square
    if (isInBoard(row + direction, col) && !board[row + direction][col]) {
        moves.push({ row: row + direction, col });
        
        // Move forward two squares from starting position
        if (row === startRow && !board[row + 2 * direction][col]) {
            moves.push({ row: row + 2 * direction, col });
        }
    }
    
    // Capture diagonally
    for (let i = -1; i <= 1; i += 2) {
        if (isInBoard(row + direction, col + i) && 
            board[row + direction][col + i] && 
            board[row + direction][col + i][0] !== color) {
            moves.push({ row: row + direction, col: col + i });
        }
    }
}
```

Similar functions exist for all other piece types (`getRookMoves`, `getKnightMoves`, `getBishopMoves`, `getQueenMoves`, and `getKingMoves`). These functions:
1. Determine the possible moves for each piece according to chess rules
2. Consider the boundaries of the board
3. Handle capture scenarios
4. Account for special rules (like pawns moving two squares from their starting position)

#### Legal Move Filtering

```javascript
function getValidMoves(row, col) {
    // ... get potential moves based on piece type ...
    
    // Filter out moves that would put or leave the king in check
    return moves.filter(move => {
        // Make the move temporarily
        const savedPiece = board[move.row][move.col];
        board[move.row][move.col] = board[row][col];
        board[row][col] = '';
        
        // Check if the king is in check after the move
        const inCheck = isCheck(color);
        
        // Undo the move
        board[row][col] = board[move.row][move.col];
        board[move.row][move.col] = savedPiece;
        
        return !inCheck;
    });
}
```

This function ensures that only legal moves are allowed by:
1. Getting all potential moves for a piece based on its movement rules
2. Testing each potential move by temporarily applying it to the board
3. Checking if the player's king would be in check after the move
4. Removing illegal moves that would leave or put the king in check

#### Check and Checkmate Detection

```javascript
function isCheck(color) {
    // Find the king's position
    let kingRow = -1;
    let kingCol = -1;
    
    for (let row = 0; row < 8; row++) {
        for (let col = 0; col < 8; col++) {
            if (board[row][col] === `${color}k`) {
                kingRow = row;
                kingCol = col;
                break;
            }
        }
        if (kingRow !== -1) break;
    }
    
    // Check if any opponent's piece can capture the king
    const opponentColor = color === 'w' ? 'b' : 'w';
    
    // ... check if any opponent's piece can attack the king ...
    
    return false; // Not in check
}

function isCheckmate(color) {
    // Check if the player is in check
    if (!isCheck(color)) return false;
    
    // Check if any move can get the player out of check
    for (let row = 0; row < 8; row++) {
        for (let col = 0; col < 8; col++) {
            const piece = board[row][col];
            if (piece && piece[0] === color) {
                const moves = getValidMoves(row, col);
                if (moves.length > 0) {
                    return false; // At least one legal move exists
                }
            }
        }
    }
    
    return true; // No legal moves, it's checkmate
}
```

These functions implement the check and checkmate detection by:
1. Finding the position of the king
2. Determining if any opponent's piece can attack the king (check)
3. Checking if there are any legal moves that can get out of check
4. If there are no legal moves and the king is in check, it's checkmate

### AI Implementation

The AI uses the minimax algorithm with alpha-beta pruning to make decisions:

```javascript
function makeAIMove() {
    const depth = parseInt(difficultySelect.value);
    const bestMove = findBestMove(depth);
    
    if (bestMove) {
        makeMove(bestMove.fromRow, bestMove.fromCol, bestMove.toRow, bestMove.toCol);
        
        // Check if game is over
        if (isCheckmate('white')) {
            gameOver = true;
            statusElement.textContent = 'Checkmate! AI wins!';
            renderBoard();
            return;
        }
        
        currentPlayer = 'white';
        statusElement.textContent = 'Your turn (White)';
    } else {
        // No valid moves for AI
        if (isCheck('black')) { // If black is in check, and has no legal moves
            gameOver = true;
            statusElement.textContent = 'Checkmate! You win!';
        } else { // If black is not in check, but still has no moves
            gameOver = true;
            statusElement.textContent = 'Stalemate! Game is a draw!';
        }
    }
    
    renderBoard();
}
```

The AI move function:
1. Gets the selected difficulty level (search depth)
2. Finds the best move using the minimax algorithm
3. Makes the move and checks for game-ending conditions
4. Handles edge cases like stalemate

## Deep Dive into the Minimax Algorithm

### How Minimax Works in Chess

The minimax algorithm is the core of the AI's decision-making process. It's a recursive algorithm used in two-player zero-sum games to find the optimal move. In chess:

1. **Maximizing Player (AI/Black)**: Tries to maximize the board evaluation score
2. **Minimizing Player (Human/White)**: Tries to minimize the board evaluation score
3. **Search Depth**: Determines how many moves ahead the AI looks

```javascript
function minimax(depth, isMaximizing, alpha, beta) {
    // Base case: reached maximum depth
    if (depth === 0) {
        return evaluateBoard();
    }
    
    if (isMaximizing) {
        // AI's turn (maximizing)
        let maxScore = -Infinity;
        
        // For each piece of the AI
        for (let row = 0; row < 8; row++) {
            for (let col = 0; col < 8; col++) {
                const piece = board[row][col];
                if (piece && piece[0] === 'b') {
                    const moves = getValidMoves(row, col);
                    
                    // For each valid move
                    for (const move of moves) {
                        // Make the move temporarily
                        const savedPiece = board[move.row][move.col];
                        board[move.row][move.col] = board[row][col];
                        board[row][col] = '';
                        
                        // Recursively call minimax
                        const score = minimax(depth - 1, false, alpha, beta);
                        
                        // Undo the move
                        board[row][col] = board[move.row][move.col];
                        board[move.row][move.col] = savedPiece;
                        
                        // Update max score
                        maxScore = Math.max(maxScore, score);
                        alpha = Math.max(alpha, score);
                        
                        // Alpha-beta pruning
                        if (beta <= alpha) {
                            break;
                        }
                    }
                }
            }
        }
        
        return maxScore;
    } else {
        // Human's turn (minimizing)
        let minScore = Infinity;
        
        // ... similar logic for minimizing player ...
        
        return minScore;
    }
}
```

The minimax algorithm works by:
1. Creating a game tree where each node is a possible board state
2. Recursively exploring this tree down to the specified depth
3. Evaluating the board at leaf nodes (using the `evaluateBoard` function)
4. Propagating values up the tree, with the maximizing player choosing the maximum value and the minimizing player choosing the minimum value
5. Using alpha-beta pruning to eliminate branches that won't affect the final decision, improving efficiency

### Board Evaluation

The AI needs a way to evaluate how good a particular board position is. This is done through the `evaluateBoard` function:

```javascript
function evaluateBoard() {
    let score = 0;
    
    // Piece values
    const pieceValues = {
        'p': 10,
        'n': 30,
        'b': 30,
        'r': 50,
        'q': 90,
        'k': 900
    };
    
    // Count material for both sides
    for (let row = 0; row < 8; row++) {
        for (let col = 0; col < 8; col++) {
            const piece = board[row][col];
            if (piece) {
                const value = pieceValues[piece[1]];
                if (piece[0] === 'b') {
                    score += value;
                } else {
                    score -= value;
                }
            }
        }
    }
    
    // Add a small random factor to prevent repetitive moves
    score += (Math.random() - 0.5);
    
    return score;
}
```

This function assigns a numeric value to the current board state by:
1. Assigning standard values to each piece type
2. Adding the values of all Black pieces to the score
3. Subtracting the values of all White pieces from the score
4. Adding a small random factor to prevent repetitive play

### Alpha-Beta Pruning

Alpha-beta pruning is an optimization technique used to significantly reduce the number of nodes evaluated in the minimax algorithm:

1. **Alpha**: The best (highest) value found so far for the maximizing player
2. **Beta**: The best (lowest) value found so far for the minimizing player
3. **Pruning**: If beta ≤ alpha, the current player can skip evaluating further moves in this branch

This optimization allows the AI to search deeper in the same amount of time, making it more effective.

## Conclusion

This chess game implementation demonstrates several important programming concepts:

1. **Game State Management**: Tracking and updating the complex state of a chess game
2. **AI Decision Making**: Using the minimax algorithm with alpha-beta pruning
3. **Complex Rule Implementation**: Encoding all the rules of chess in a logical way
4. **UI Design**: Creating an intuitive interface for a classic board game
5. **Recursive Algorithms**: Using recursion to explore game possibilities

The minimax algorithm implementation makes this chess game surprisingly sophisticated, creating an AI opponent that can think several moves ahead. By understanding how this works, you've gained insights into a fundamental algorithm used in various games and decision-making systems.

Feel free to use this project as a starting point to build more advanced chess implementations or to experiment with different AI approaches!

---

Happy coding and gaming!