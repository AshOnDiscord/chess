<script setup lang="ts">
import { ref, type Ref } from "vue";

enum Piece {
  Pawn,
  Knight,
  Bishop,
  Rook,
  Queen,
  King,
}

enum Team {
  White,
  Black,
}

interface Square {
  piece: Piece | null;
  team: Team | null;
}

const PieceBoard: Ref<Square[][]> = ref([]);

const viewBoard: Ref<boolean[][]> = ref([]);

const turnOrder: Ref<Team[]> = ref([Team.White, Team.Black]);

const turn: Ref<Team> = ref(turnOrder.value[0]);

const setupBoard = () => {
  for (let i = 0; i < 8; i++) {
    PieceBoard.value[i] = [];
    for (let j = 0; j < 8; j++) {
      PieceBoard.value[i][j] = { piece: null, team: null };
    }
  }

  resetViewBoard();

  PieceBoard.value[0] = [
    { piece: Piece.Rook, team: Team.White },
    { piece: Piece.Knight, team: Team.White },
    { piece: Piece.Bishop, team: Team.White },
    { piece: Piece.Queen, team: Team.White },
    { piece: Piece.King, team: Team.White },
    { piece: Piece.Bishop, team: Team.White },
    { piece: Piece.Knight, team: Team.White },
    { piece: Piece.Rook, team: Team.White },
  ];
  PieceBoard.value[1] = [
    { piece: Piece.Pawn, team: Team.White },
    { piece: Piece.Pawn, team: Team.White },
    { piece: Piece.Pawn, team: Team.White },
    { piece: Piece.Pawn, team: Team.White },
    { piece: Piece.Pawn, team: Team.White },
    { piece: Piece.Pawn, team: Team.White },
    { piece: Piece.Pawn, team: Team.White },
    { piece: Piece.Pawn, team: Team.White },
  ];

  PieceBoard.value[6] = [
    { piece: Piece.Pawn, team: Team.Black },
    { piece: Piece.Pawn, team: Team.Black },
    { piece: Piece.Pawn, team: Team.Black },
    { piece: Piece.Pawn, team: Team.Black },
    { piece: Piece.Pawn, team: Team.Black },
    { piece: Piece.Pawn, team: Team.Black },
    { piece: Piece.Pawn, team: Team.Black },
    { piece: Piece.Pawn, team: Team.Black },
  ];
  PieceBoard.value[7] = [
    { piece: Piece.Rook, team: Team.Black },
    { piece: Piece.Knight, team: Team.Black },
    { piece: Piece.Bishop, team: Team.Black },
    { piece: Piece.Queen, team: Team.Black },
    { piece: Piece.King, team: Team.Black },
    { piece: Piece.Bishop, team: Team.Black },
    { piece: Piece.Knight, team: Team.Black },
    { piece: Piece.Rook, team: Team.Black },
  ];
};

const resetViewBoard = () => {
  for (let i = 0; i < 8; i++) {
    viewBoard.value[i] = [];
    for (let j = 0; j < 8; j++) {
      viewBoard.value[i][j] = false;
    }
  }
};

setupBoard();

const canMove = (y: number, x: number, team: Team | null) => {
  return (
    y >= 0 && y < 8 && x >= 0 && x < 8 && PieceBoard.value[y][x]?.team != team
  );
};

const selectedPiece: Ref<{
  y: number;
  x: number;
} | null> = ref(null);

const selectPiece = (y: number, x: number) => {
  resetViewBoard();
  const square = PieceBoard.value[y][x];
  if (square.piece == null) {
    return;
  } else if (square.team != turn.value) {
    return;
  } else if (square.piece == Piece.Pawn) {
    if (square.team == Team.White) {
      if (
        canMove(y + 1, x, Team.White) &&
        PieceBoard.value[y + 1][x].team == null
      ) {
        viewBoard.value[y + 1][x] = true;
      }
      if (
        y == 1 &&
        canMove(y + 2, x, Team.White) &&
        PieceBoard.value[y + 2][x].team == null
      ) {
        viewBoard.value[y + 2][x] = true;
      }
      if (
        canMove(y + 1, x + 1, Team.White) &&
        PieceBoard.value[y + 1][x + 1].team !== null
      ) {
        viewBoard.value[y + 1][x + 1] = true;
      }
      if (
        canMove(y + 1, x - 1, Team.White) &&
        PieceBoard.value[y + 1][x - 1].team !== null
      ) {
        viewBoard.value[y + 1][x - 1] = true;
      }
    } else {
      if (
        canMove(y - 1, x, Team.Black) &&
        PieceBoard.value[y - 1][x].team == null
      ) {
        viewBoard.value[y - 1][x] = true;
      }
      if (
        y == 6 &&
        canMove(y - 2, x, Team.Black) &&
        PieceBoard.value[y - 2][x].team == null
      ) {
        viewBoard.value[y - 2][x] = true;
      }
      if (
        canMove(y - 1, x + 1, Team.Black) &&
        PieceBoard.value[y - 1][x + 1].team !== null
      ) {
        viewBoard.value[y - 1][x + 1] = true;
      }
      if (
        canMove(y - 1, x - 1, Team.Black) &&
        PieceBoard.value[y - 1][x - 1].team !== null
      ) {
        viewBoard.value[y - 1][x - 1] = true;
      }
    }
  } else if (square.piece == Piece.Knight) {
    if (canMove(y + 2, x + 1, square.team)) {
      viewBoard.value[y + 2][x + 1] = true;
    }
    if (canMove(y + 2, x - 1, square.team)) {
      viewBoard.value[y + 2][x - 1] = true;
    }
    if (canMove(y - 2, x + 1, square.team)) {
      viewBoard.value[y - 2][x + 1] = true;
    }
    if (canMove(y - 2, x - 1, square.team)) {
      viewBoard.value[y - 2][x - 1] = true;
    }
    if (canMove(y + 1, x + 2, square.team)) {
      viewBoard.value[y + 1][x + 2] = true;
    }
    if (canMove(y + 1, x - 2, square.team)) {
      viewBoard.value[y + 1][x - 2] = true;
    }
    if (canMove(y - 1, x + 2, square.team)) {
      viewBoard.value[y - 1][x + 2] = true;
    }
    if (canMove(y - 1, x - 2, square.team)) {
      viewBoard.value[y - 1][x - 2] = true;
    }
  } else if (square.piece == Piece.King) {
    // top left
    if (canMove(y + 1, x + 1, square.team)) {
      viewBoard.value[y + 1][x + 1] = true;
    }
    // top middle
    if (canMove(y + 1, x, square.team)) {
      viewBoard.value[y + 1][x] = true;
    }
    // top right
    if (canMove(y + 1, x - 1, square.team)) {
      viewBoard.value[y + 1][x - 1] = true;
    }
    // middle left
    if (canMove(y, x + 1, square.team)) {
      viewBoard.value[y][x + 1] = true;
    }
    // middle right
    if (canMove(y, x - 1, square.team)) {
      viewBoard.value[y][x - 1] = true;
    }
    // bottom left
    if (canMove(y - 1, x + 1, square.team)) {
      viewBoard.value[y - 1][x + 1] = true;
    }
    // bottom middle
    if (canMove(y - 1, x, square.team)) {
      viewBoard.value[y - 1][x] = true;
    }
    // bottom right
    if (canMove(y - 1, x - 1, square.team)) {
      viewBoard.value[y - 1][x - 1] = true;
    }
  }
  if (square.piece == Piece.Bishop || square.piece == Piece.Queen) {
    // top left
    for (let i = 1; i < 8; i++) {
      if (canMove(y + i, x + i, square.team)) {
        viewBoard.value[y + i][x + i] = true;
      } else {
        break;
      }
    }
    // top right
    for (let i = 1; i < 8; i++) {
      if (canMove(y + i, x - i, square.team)) {
        viewBoard.value[y + i][x - i] = true;
      } else {
        break;
      }
    }
    // bottom left
    for (let i = 1; i < 8; i++) {
      if (canMove(y - i, x + i, square.team)) {
        viewBoard.value[y - i][x + i] = true;
      } else {
        break;
      }
    }
    // bottom right
    for (let i = 1; i < 8; i++) {
      if (canMove(y - i, x - i, square.team)) {
        viewBoard.value[y - i][x - i] = true;
      } else {
        break;
      }
    }
  }
  if (square.piece == Piece.Rook || square.piece == Piece.Queen) {
    // top
    for (let i = 1; i < 8; i++) {
      if (canMove(y + i, x, square.team)) {
        viewBoard.value[y + i][x] = true;
      } else {
        break;
      }
    }
    // bottom
    for (let i = 1; i < 8; i++) {
      if (canMove(y - i, x, square.team)) {
        viewBoard.value[y - i][x] = true;
      } else {
        break;
      }
    }
    // left
    for (let i = 1; i < 8; i++) {
      if (canMove(y, x + i, square.team)) {
        viewBoard.value[y][x + i] = true;
      } else {
        break;
      }
    }
    // right
    for (let i = 1; i < 8; i++) {
      if (canMove(y, x - i, square.team)) {
        viewBoard.value[y][x - i] = true;
      } else {
        break;
      }
    }
  }
  viewBoard.value[y][x] = false;
  selectedPiece.value = {
    x: x,
    y: y,
  };
};

const movePiece = (y: number, x: number) => {
  if (!selectedPiece.value) return;

  const pieceCopy =
    PieceBoard.value[selectedPiece.value.y][selectedPiece.value.x];

  PieceBoard.value[selectedPiece.value.y][selectedPiece.value.x] = {
    piece: null,
    team: null,
  };

  PieceBoard.value[y][x] = pieceCopy;

  selectedPiece.value = null;
  resetViewBoard();

  const turnIndex = turnOrder.value.indexOf(turn.value);
  turn.value = turnOrder.value[(turnIndex + 1) % turnOrder.value.length];
};
</script>

<template>
  {{ turn }}
  <main class="grid grid-rows-[repeat(8,2rem)]">
    <div
      v-for="(row, i) in PieceBoard.slice().reverse()"
      :key="i"
      class="grid grid-cols-[repeat(8,2rem)]"
    >
      <div
        v-for="(column, j) in row"
        :key="j"
        class="flex justify-center items-center"
        :class="{
          'bg-amber-600': (i + j) % 2 == 1,
          'bg-orange-200': (i + j) % 2 == 0,
          '!bg-green-500': viewBoard[7 - i][j],
        }"
        @click="
          viewBoard[7 - i][j] ? movePiece(7 - i, j) : selectPiece(7 - i, j)
        "
      >
        {{ column.piece == Piece.King && column.team == Team.White ? "♔" : "" }}
        {{
          column.piece == Piece.Queen && column.team == Team.White ? "♕" : ""
        }}
        {{
          column.piece == Piece.Bishop && column.team == Team.White ? "♗" : ""
        }}
        {{
          column.piece == Piece.Knight && column.team == Team.White ? "♘" : ""
        }}
        {{ column.piece == Piece.Rook && column.team == Team.White ? "♖" : "" }}
        {{ column.piece == Piece.Pawn && column.team == Team.White ? "♙" : "" }}
        {{ column.piece == Piece.King && column.team == Team.Black ? "♚" : "" }}
        {{
          column.piece == Piece.Queen && column.team == Team.Black ? "♛" : ""
        }}
        {{
          column.piece == Piece.Bishop && column.team == Team.Black ? "♝" : ""
        }}
        {{
          column.piece == Piece.Knight && column.team == Team.Black ? "♞" : ""
        }}
        {{ column.piece == Piece.Rook && column.team == Team.Black ? "♜" : "" }}
        {{ column.piece == Piece.Pawn && column.team == Team.Black ? "♟" : "" }}
      </div>
    </div>
  </main>
</template>
