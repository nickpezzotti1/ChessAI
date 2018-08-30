# ChessAI
The following code is an example of Computer Chess, *"a software capable of playing chess autonomously without human guidance"* [*](https://en.wikipedia.org/wiki/Computer_chess), implemented using only Java and the open-source library [ChessLib](https://github.com/bhlangonijr/chesslib) that allows us to generate legal chess moves given a chessboard position, this avoids the hassle of programming the game of chess itself and the algorithms that determine which moves are legal. This allows us to focus on the more interesting task of creating a flexible algorithm that finds the best move given a board state.

## How to teach a computer to play Chess
To **teach a computer how to win at chess** we must teach it the **rules** and how make **decisions**. The playing part is going to be handled by our ChessLib library, that is just gonna give our program a set of rules. We will only be interfacing with the following method; it gives us the list of all *legal moves* we can perform at any given point `MoveGenerator.generateLegalMoves(Board board)`. *We don't even have to really know how to play chess to design an algorithm that can beat many novice players.*

The harder challenge is teaching it **how to make decisions**. The decision making process is one we humans are very familiar with since birth, I'll define as "given multiple options we choose (when acting rationally) which is best". It is not innate to computers though, so we must teach it how. Let's break the problem down.

Given the possible legal moves, we apply each of them to our board and we must decide which of the resulting boards is the most favorable for us, we then must return the move that lead to that board. How do we decide which is the most favorable? We must be able to rank the boards.
Chess is a *static game of complete information*, meaning we can measure how favorable a current game state is for each player at any point, even as an external spectator, without knowing what lead up to that moment or the players' future intentions. We can tehrefore design a **static evaluation function** that uses heuristics to rank the resulting boards (applying each move to the current board), and decide which move is most favorable. A simple evaluation function used by novice players is that of assigning a numerical value to board pieces (+1 for capturing a pawn, -1 for loosing one, +5 for a rook, etc.); another basic level of intelligence we are going to implement is that of fighting for center control, board mobility and king safety (pieces are more valuable when in favorable positions, such as a queen in the center squares). This level of intelligence, though, would only look one move into the future, resulting in a naive computer that captures enemy pieces when possible, without thinking of the consequences. "Consquences" are the harder thing to teach to computers, we would need to predict what our oponent's move will be in reaction to ours, and what our move in response will be to that, and his again, and so on, ideally until the end of the game. With an average of 35 legal moves per board state it doesn't take long before trying and evaluating each of the moves would take *WAY* too long.

The following is a diagram of the tree structure I used to represent the problem. Each MMNode holds information about the current board state and move that lead to the state. This is an example of a minimax tree, each level of the tree is either a minimising level or a maximizing level, in other words whether its the oponent's turn to play or yours; if it's yours you want to maximize the board value (or score), because we want a favorable board. WIth this strategy we can usually analyse up to a few moves forward, as it takes about O(30^n), where n is the depth of the tree. I, therefore, implemented alpha-beta pruning, and algorithm which allows us to cut off big portions of the game of the tree, saving us computing time (in particular we can expect to take O(30^n/2) = O(sqrt(30^n))).

![alt text](https://github.com/nickpezzotti1/ChessAI/blob/master/MinimaxTree.png "Example gamet-tree")

After numerous development iterations, I decided to develop it on the skeleton of the standard industry version of alpha beta pruning, as seen [here](https://en.wikipedia.org/wiki/Alpha%E2%80%93beta_pruning#Pseudocode). It had to be adapted as it uses recursive calls to build the game tree, and traces its way back up to the root node, returning the score of the best board we can achieve with one move. We must later loop through the legal moves until we find one that leads to that board.

```
int bestScore = alphabeta(root, depth, Integer.MIN_VALUE, Integer.MAX_VALUE, true);
MMNode bestChild = null;
for (MMNode child : root.getChildren()) {
    if (child.getScore() == bestScore) {
    bestChild = child;
}
```

# Possible improvements
1. Replacing the evaluation function with a more detailed and complicated one, for example favoring bishop pairs over knight pairs
2. The best chess computers have hardcoded opening strategies and end game strategies
3. Parallelize the search tree using Java ParallelStreams
