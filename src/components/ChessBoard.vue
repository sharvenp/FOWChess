<template>
  <div class="chessboard">
    <h1>Fog of war Chess!</h1>
    <canvas id="board" width="600" height="600" class="board" 
      @mousemove="mouseMove"
      @mousedown="makeMove"></canvas>
    <p class="status">{{status}}</p>    
    <img src="../assets/pw.png" id="pw" style="display: none;">
    <img src="../assets/nw.png" id="nw" style="display: none;">
    <img src="../assets/bw.png" id="bw" style="display: none;">
    <img src="../assets/rw.png" id="rw" style="display: none;">
    <img src="../assets/qw.png" id="qw" style="display: none;">
    <img src="../assets/kw.png" id="kw" style="display: none;">
    <img src="../assets/pb.png" id="pb" style="display: none;">
    <img src="../assets/nb.png" id="nb" style="display: none;">
    <img src="../assets/bb.png" id="bb" style="display: none;">
    <img src="../assets/rb.png" id="rb" style="display: none;">
    <img src="../assets/qb.png" id="qb" style="display: none;">
    <img src="../assets/kb.png" id="kb" style="display: none;">
  </div>
</template>

<script>
export default {
  name: 'ChessBoard',
  data: function() {
  return {
      canvas: null,
      side: 'w',
      x: 0,
      y: 0,
      selectedPiece: null,
      status: null,
      gameOver: false
    };
  },
  methods: {
    getPieceXY(x, y) {
      var row = Math.floor(y / this.sqWidth);
      var col = Math.floor(x / this.sqWidth);

      var algebraic = this.col_alg[col] + this.row_alg[row];
      return this.chess.get(algebraic);
    },
    
    getPieceRowCol(row, col) {
      var algebraic = this.col_alg[col] + this.row_alg[row];
      return this.chess.get(algebraic);
    },

    makeBoard() {
      let ctx = this.canvas;
      var row, col;
      var whiteCol = "#e8ebef";
      var blackCol = "#7d8796";
      var selectedCol = "#fff06e";
      var legalMoveCol = "#0000ff";
      var alternate = true;

      // Draw squares
      for (row = 0; row < 8; row++) {
        for (col = 0; col < 8; col++) {
          ctx.fillStyle = (alternate ? whiteCol : blackCol);
          ctx.fillRect(col * this.sqWidth, row * this.sqWidth, this.sqWidth, this.sqWidth);
          alternate = !alternate;
        }
        alternate = !alternate;
      }

      // Draw  selected square
      ctx.strokeStyle = selectedCol;
      ctx.lineWidth = 5;
      var ro = Math.floor(this.y / this.sqWidth);
      var co = Math.floor(this.x / this.sqWidth);
      ctx.strokeRect(co * this.sqWidth + 2.5, ro * this.sqWidth + 2.5, this.sqWidth - 5, this.sqWidth - 5);

      if (this.selectedPiece != null) {
        ctx.fillStyle = selectedCol;
        ctx.fillRect(this.col * this.sqWidth, this.row * this.sqWidth, this.sqWidth, this.sqWidth);
      }

      // Draw pieces
      for (row = 0; row < 8; row++) {
        for (col = 0; col < 8; col++) {
          var result = this.getPieceRowCol(row, col);
          
          if (result == null) {
            continue;
          }
          var img = document.getElementById(result['type']+result['color']);

          // Check if player is in check
          if (this.chess.in_check()) {
            if (result['type'] == 'k' && result['color'] == this.chess.turn()) {
              ctx.fillStyle = "#ff0000";
              ctx.fillRect(col * this.sqWidth, row * this.sqWidth, this.sqWidth, this.sqWidth);
            }
          }

          ctx.drawImage(img, col * this.sqWidth, row * this.sqWidth, this.sqWidth, this.sqWidth);
        }
      }

      // Draw legal moves
      if (this.selectedPiece) {
        var i;
        for (i = 0; i < this.legalMoves.length; i++) {
          var move = this.legalMoves[i];

          move = move.replace("+", "").replace("#", "");

          var c = this.col_alg.indexOf(move[move.length - 2]);
          var r = this.row_alg.indexOf(move[move.length - 1]);
          
          if (move.indexOf("=") != -1) {
            // This is promotion move
            c = this.col_alg.indexOf(move[0]);
            r = this.row_alg.indexOf(move[1]);
          }
          
          ctx.strokeStyle = legalMoveCol;
          ctx.beginPath();
          ctx.lineWidth = 5;
          ctx.arc(c * this.sqWidth + this.sqWidth / 2, r * this.sqWidth + this.sqWidth / 2, 10, 0, 2 * Math.PI);
          ctx.stroke();
        }
      }
    },

    movePiece() {
      var playedMove;
      var i;
      for (i = 0; i < this.legalMoves.length; i++) {
        var move = this.legalMoves[i];
        move = move.replace("+", "").replace("#", "");
        var c = this.col_alg.indexOf(move[move.length - 2]);
        var r = this.row_alg.indexOf(move[move.length - 1]);
        if (move.indexOf("=") != -1) {
          // This is promotion move
          c = this.col_alg.indexOf(move[0]);
          r = this.row_alg.indexOf(move[1]);
        }

        if (c == this.col && r == this.row) {
          playedMove = move;
          break; 
        }
      }
      this.chess.move(playedMove);
      this.selectedPiece = null;
      this.makeBoard();

      if (this.chess.in_checkmate()) {
        this.status = "CHECKMATE!";
      } else if (this.chess.in_stalemate()) {
        this.status = "STALEMATE!";
      } else if (this.chess.in_threefold_repetition()) {
        this.status = "THREEFOLD REPETITION!";
      } else if (this.chess.insufficient_material()) {
        this.status = "INSUFFICIENT MATERIAL!";
      } else if (this.chess.in_draw()) {
        this.status = "DRAW!";
      }

      this.gameOver = this.chess.game_over();
    },

    mouseMove(e) {
      this.x = e.offsetX;
      this.y = e.offsetY;
      this.makeBoard();
    },
    
    makeMove(e) {
      this.mouseMove(e);
      this.row = Math.floor(this.y / this.sqWidth);
      this.col = Math.floor(this.x / this.sqWidth);

      if (this.gameOver) {
        return;
      }

      if (this.selectedPiece == null) {
        this.selectedPiece = this.getPieceXY(this.x, this.y);
  
        if (this.selectedPiece) {
          this.legalMoves = this.chess.moves({ square: this.col_alg[this.col] + this.row_alg[this.row] })
        }
      } else {
        this.movePiece();
      }

      this.makeBoard();
    }
  },

  mounted: function mounted() {
    var c = document.getElementById("board");
    this.canvas = c.getContext('2d');
    const Chess = require('chess.js');
    // const chess = new Chess("8/6k1/6r1/3N4/1K6/8/2Q5/4R3 w - - 0 1");
    const chess = new Chess();
    this.chess = chess;
    
    // Create converted
    this.row_alg = ["8", "7", "6", "5", "4", "3", "2", "1"];
    this.col_alg = ["a", "b", "c", "d", "e", "f", "g", "h"];
        
    // Load chess pieces
    this.pieceImages = {"b": {}, "w": {}};
    var i, j;
    var color = ['b', 'w'];
    var pieceTypes = ['p', 'n', 'b', 'r', 'q', 'k'];

    for (i = 0; i < color.length; i++) {
      for (j = 0; j < pieceTypes.length; j++) {
        var col = color[i];
        var pie = pieceTypes[j];
        var str = "./assets/"+col+pie+".png";
        var img = new Image();
        img.src = str;
        this.pieceImages[col][pie] = img;
      }
    }
    this.width = 600;
    this.sqWidth = Math.floor(this.width / 8);
    this.makeBoard();
  },
}
</script>


<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.board {
  border: 5px solid black;
}
.status {
  font-weight: bold;
  font-size: 20px;
}
</style>
