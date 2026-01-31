# Sanguine: Strategic Card Game (code available upon request)

A Java-based implementation of a strategic card battle game inspired by "Queens Blood" from Final Fantasy VII Rebirth. Two players compete on a grid-based board, strategically placing cards to win rows and ultimately the game.

## 📋 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Game Rules](#game-rules)
- [Architecture](#architecture)
- [Usage](#usage)
- [AI Strategies](#ai-strategies)
- [Project Structure](#project-structure)

## Overview

Sanguine is a turn-based strategy card game where two players (RED and BLUE) compete on a rectangular grid board. Players must carefully manage their resources (pawns) to play cards, with each card having unique influence patterns that affect the board state. Victory is achieved by winning the most rows based on card values.

**Key Technical Highlights:**
- Full MVC architecture with clean separation of concerns
- Sophisticated AI with multiple strategy implementations including MiniMax
- Composite pattern for strategy chaining and tie-breaking
- Interactive GUI built with Java Swing
- Comprehensive test suite with 95%+ code coverage

## Features

### Game Modes
- **Human vs Human**: Two players on separate windows
- **Human vs AI**: Play against configurable AI strategies
- **AI vs AI**: Watch different strategies compete

### AI Strategies
- **Fill First**: Baseline strategy - plays first legal move
- **Maximize Score**: Focuses on winning rows by score advantage
- **Control Board**: Maximizes cell influence on the board
- **MiniMax**: Evaluates opponent's best responses (advanced)
- **Composite Strategies**: Chain multiple strategies with tie-breaking

### Visual Features
- Custom-designed game board with score tracking
- Animated card display with character artwork
- Real-time pawn visualization
- Interactive card selection and placement
- Score panels showing row-by-row competition

## 📖 Game Rules

### Board Setup
- Grid dimensions: Configurable (default 3 rows × 5 columns)
- Starting pawns: Each player begins with 1 pawn in each row on their side
- Hand size: Configurable based on deck size (typically 5 cards)

### Card Mechanics
- **Cost**: Pawns required to play the card (1-3)
- **Value**: Points contributed to row scoring
- **Influence Grid**: 5×5 pattern determining which cells gain pawns when played

### Winning
1. Cards can only be played on cells with sufficient pawns matching the player's color
2. Playing a card spreads influence to adjacent cells based on the card's influence grid
3. Each row is scored by summing card values for each player
4. The player wins a row if their score exceeds the opponent's
5. The player who wins the most rows wins the game

## 🏗 Architecture

### Model-View-Controller Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                         Controller                           │
│  ┌──────────────────┐  ┌──────────────────┐                 │
│  │ HumanPlayer      │  │ MachinePlayer    │                 │
│  │ (GUI Input)      │  │ (Strategy AI)    │                 │
│  └──────────────────┘  └──────────────────┘                 │
└───────────────────┬──────────────┬──────────────────────────┘
                    │              │
                    ▼              ▼
┌─────────────────────────────────────────────────────────────┐
│                           Model                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ GameModelImpl                                        │   │
│  │  • Board management  • Turn logic  • Scoring        │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Board   │  │   Card   │  │  Player  │  │BoardCell │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└───────────────────────────────┬─────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────┐
│                            View                              │
│  ┌──────────────────┐  ┌──────────────────┐                 │
│  │  GameViewFrame   │  │  BoardPanel      │                 │
│  │  (Main Window)   │  │  (Grid Display)  │                 │
│  └──────────────────┘  └──────────────────┘                 │
│  ┌──────────────────┐                                        │
│  │   CardPanel      │                                        │
│  │   (Hand Display) │                                        │
│  └──────────────────┘                                        │
└─────────────────────────────────────────────────────────────┘
```

### Key Design Patterns

**Strategy Pattern**: Pluggable AI algorithms
```java
interface MoveStrategy {
    Move chooseMove(ReadOnlySanguineModel model, PlayerColor player);
}
```

**Composite Pattern**: Strategy chaining with tie-breaking
```java
new AdvancedCompositeStrategy(
    new MiniMaxPlayer(),      // Primary strategy
    new ControlBoardPlayer()  // Tie-breaker
)
```

**Observer Pattern**: Model notifications to controllers
```java
interface ModelStatusListener {
    void onTurnChanged(PlayerColor player);
    void onGameOver(PlayerColor winner);
}
```

## Usage

### Running the Game

**Basic Game (Human vs Human):**
```bash
java -jar sanguine.jar 3 5 deck.txt deck.txt human human
```

**Human vs AI:**
```bash
java -jar sanguine.jar 3 5 red.deck blue.deck human strategy2
```

**AI vs AI:**
```bash
java -jar sanguine.jar 5 11 red.deck blue.deck strategy1 strategy2
```

### Command-Line Arguments
```
<rows> <cols> <red-deck> <blue-deck> <red-player> <blue-player>

rows, cols:     Board dimensions (cols must be odd)
red-deck:       Path to RED player's deck file
blue-deck:      Path to BLUE player's deck file
red-player:     human | strategy1 | strategy2
blue-player:    human | strategy1 | strategy2
```

### Deck File Format
```
CardName Cost Value
XXXXX    (5x5 influence grid)
XXIXX     C = Card position
XCXXX     I = Influence cell
XXIXX     X = No influence
XXXXX
```

### Controls
- **Mouse Click**: Select card from hand, select board cell
- **Enter/Space**: Confirm card placement
- **P**: Pass turn

##  AI Strategies

### Strategy Implementations

**1. Fill First (Baseline)**
- Scans board top-to-bottom, left-to-right
- Plays first legal move found
- Fast but not strategic

**2. Maximize Score**
- Analyzes row scores for both players
- Prioritizes moves that flip losing rows to winning
- Good for competitive play

**3. Control Board (Extra Credit)**
- Calculates influence gained per move
- Maximizes cell control on the board
- Effective for territory control

**4. MiniMax (Advanced)**
- Evaluates opponent's best counter-move for each option
- Chooses move that minimizes opponent's maximum gain
- Computationally expensive but strategically superior

### Strategy Composition

**Simple Composite:**
```java
new CompositeStrategy(
    new MaximizeScorePlayer(),  // Try first
    new FillFirstPlayer()       // Fallback if no winning row moves
)
```

**Advanced Composite with Tie-Breaking:**
```java
new AdvancedCompositeStrategy(
    new ControlBoardPlayer(),   // Get all best moves
    new FillFirstPlayer()       // Break ties by position
)
```

## 📁 Project Structure

```
sanguine/
├── src/main/java/sanguine/
│   ├── controller/
│   │   ├── SanguineControllerImpl.java
│   │   ├── HumanPlayer.java
│   │   ├── MachinePlayer.java
│   │   └── DeckLoader.java
│   ├── model/
│   │   ├── GameModelImpl.java
│   │   ├── Board.java
│   │   ├── Card.java
│   │   └── Player.java
│   ├── strategy/
│   │   ├── FillFirstPlayer.java
│   │   ├── MaximizeScorePlayer.java
│   │   ├── ControlBoardPlayer.java
│   │   ├── MiniMaxPlayer.java
│   │   └── AdvancedCompositeStrategy.java
│   ├── view/
│   │   ├── GameViewFrame.java
│   │   ├── BoardPanel.java
│   │   └── CardPanel.java
│   └── SanguineGame.java
├── src/test/java/sanguine/
│   ├── ModelTests.java
│   ├── StrategyTests.java
│   └── MockSanguineModel.java
├── docs/
│   └── deck.txt
└── README.md
```
