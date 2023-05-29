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

const enPassant: Ref<boolean> = ref(false);

const lastMove: Ref<{
  from: { y: number; x: number };
  to: { y: number; x: number };
} | null> = ref(null);

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
      // en passant
      const last = lastMove.value;
      if (last) {
        const lastPiece = PieceBoard.value[last.to.y][last.to.x];
        const distance = Math.abs(last.to.y - last.from.y);
        if (
          lastPiece.team !== Team.White &&
          lastPiece.piece == Piece.Pawn &&
          distance == 2 &&
          y == last.to.y
        ) {
          if (last.to.x == x + 1) {
            viewBoard.value[y + 1][x + 1] = true;
            enPassant.value = true;
          } else if (last.to.x == x - 1) {
            viewBoard.value[y + 1][x - 1] = true;
            enPassant.value = true;
          }
        }
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
    // en passant
    const last = lastMove.value;
    if (last) {
      const lastPiece = PieceBoard.value[last.to.y][last.to.x];
      const distance = Math.abs(last.to.y - last.from.y);
      if (
        lastPiece.team !== Team.Black &&
        lastPiece.piece == Piece.Pawn &&
        distance == 2 &&
        y == last.to.y
      ) {
        if (last.to.x == x + 1) {
          viewBoard.value[y - 1][x + 1] = true;
          enPassant.value = true;
        } else if (last.to.x == x - 1) {
          viewBoard.value[y - 1][x - 1] = true;
          enPassant.value = true;
        }
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

  const lastCopy = {
    from: {
      x: selectedPiece.value.x,
      y: selectedPiece.value.y,
    },
    to: {
      x: x,
      y: y,
    },
  };

  const pieceCopy =
    PieceBoard.value[selectedPiece.value.y][selectedPiece.value.x];

  PieceBoard.value[selectedPiece.value.y][selectedPiece.value.x] = {
    piece: null,
    team: null,
  };

  PieceBoard.value[y][x] = pieceCopy;

  if (enPassant.value) {
    if (lastMove.value?.to.x == x) {
      if (turn.value == Team.White && lastMove.value?.to.y == y - 1) {
        PieceBoard.value[y - 1][x] = {
          piece: null,
          team: null,
        };
      } else if (turn.value == Team.Black && lastMove.value?.to.y == y + 1) {
        PieceBoard.value[y + 1][x] = {
          piece: null,
          team: null,
        };
      }
    }
  }

  selectedPiece.value = null;
  resetViewBoard();

  const turnIndex = turnOrder.value.indexOf(turn.value);
  turn.value = turnOrder.value[(turnIndex + 1) % turnOrder.value.length];

  lastMove.value = lastCopy;
};

const getIcon = (piece: Piece, team: Team | null) => {
  if (piece === null) return "";
  let url = "https://www.chess.com/chess-themes/pieces/neo/150/";
  if (team == Team.White) url += "w";
  if (team == Team.Black) url += "b";

  switch (piece) {
    case Piece.Pawn:
      url += "p";
      break;
    case Piece.Knight:
      url += "n";
      break;
    case Piece.Bishop:
      url += "b";
      break;
    case Piece.Rook:
      url += "r";
      break;
    case Piece.Queen:
      url += "q";
      break;
    case Piece.King:
      url += "k";
      break;
  }
  return url + ".png";
};
</script>

<template>
  <!-- {{ turn }} -->
  <div class="grid grid-cols-2">
    <div
      class="bg-[hsl(60,36%,80%)] flex justify-end gap-16 min-h-screen items-center pr-8"
    >
      <main class="grid grid-rows-[repeat(8,2rem)]">
        <div
          v-for="(row, i) in PieceBoard.slice().reverse()"
          :key="i"
          class="grid grid-cols-[repeat(8,2rem)]"
        >
          <div
            v-for="(column, j) in row"
            :key="j"
            class="flex justify-center items-center relative"
            :class="{
              'bg-[#4b7299]': (i + j) % 2 == 1,
              'bg-[#eaead2]': (i + j) % 2 == 0,
            }"
            @click="
              viewBoard[7 - i][j] ? movePiece(7 - i, j) : selectPiece(7 - i, j)
            "
          >
            <div
              class="w-8 h-8 absolute top-0 left-0 flex justify-center items-center z-100"
              v-if="viewBoard[7 - i][j]"
            >
              <div
                class="w-3 h-3 bg-black/10 rounded-full"
                v-if="PieceBoard[7 - i][j].piece === null"
              ></div>
              <div
                v-else
                class="w-8 h-8 border-4 border-black/10 rounded-full"
              ></div>
            </div>
            <div
              class="absolute w-8 h-8 top-0 left-0 bg-[#00a5ff] opacity-50"
              v-if="
                lastMove !== null &&
                ((lastMove.to.y === 7 - i && lastMove.to.x === j) ||
                  (lastMove?.from.y === 7 - i && lastMove?.from.x === j))
              "
            ></div>
            <img
              v-if="column.piece !== null"
              :src="getIcon(column.piece, column.team)"
              class="pointer-events-none relative z-10"
            />
          </div>
        </div>
      </main>
    </div>
    <div
      class="bg-[hsl(210,34%,50%)] flex justify-start gap-16 min-h-screen items-center pl-8"
    >
      <main class="grid grid-rows-[repeat(8,2rem)]">
        <div
          v-for="(row, i) in PieceBoard.slice()"
          :key="i"
          class="grid grid-cols-[repeat(8,2rem)]"
        >
          <div
            v-for="(column, j) in row"
            :key="j"
            class="flex justify-center items-center relative"
            :class="{
              'bg-[#4b7299]': (i + j) % 2 == 1,
              'bg-[#eaead2]': (i + j) % 2 == 0,
            }"
            @click="viewBoard[i][j] ? movePiece(i, j) : selectPiece(i, j)"
          >
            <div
              class="w-8 h-8 absolute top-0 left-0 flex justify-center items-center"
              v-if="viewBoard[i][j]"
            >
              <div
                class="w-3 h-3 bg-black/10 rounded-full"
                v-if="PieceBoard[i][j].piece === null"
              ></div>
              <div
                v-else
                class="w-8 h-8 border-4 border-black/10 rounded-full"
              ></div>
            </div>
            <div
              class="absolute w-8 h-8 top-0 left-0 bg-[#00a5ff] opacity-50"
              v-if="
                lastMove !== null &&
                ((lastMove.to.y === i && lastMove.to.x === j) ||
                  (lastMove?.from.y === i && lastMove?.from.x === j))
              "
            ></div>
            <img
              v-if="column.piece !== null"
              :src="getIcon(column.piece, column.team)"
              class="pointer-events-none relative z-10"
            />
          </div>
        </div>
      </main>
    </div>
  </div>
</template>
