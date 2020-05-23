<template>
  <div>
    <Tile
      v-for="tile in manager.getTiles()"
      :key="tile.id"
      :x="tile.x"
      :y="tile.y"
      :value="tile.value"
    />
  </div>
</template>

<script lang="ts">
import * as THREE from 'three';
import { Component, Prop, Vue } from 'vue-property-decorator';
import { generateBoardGeometry, generateEdgeGeometry } from '../geometries';
import { boardMaterial, boardFloorMaterial } from '../materials';
import BoardManager from '../manager';
import Scene from './Scene.vue';
import Tile from './Tile.vue';

enum Edge {
  'bottom',
  'right',
  'top',
  'left',
}

@Component<Board>({
  components: {
    Tile,
  },
  mounted() {
    this.generateBoard();
    this.startGame();
  },
  destroyed() {
    this.$parent.scene.remove(this.object3D);
  },
})
export default class Board extends Vue {
  @Prop({ type: Number, default: 4 })
  private size!: number;

  private manager = new BoardManager(this.size);

  object3D!: THREE.Mesh;

  $parent!: Scene;

  private generateBoard() {
    this.$nextTick(() => {
      this.object3D = new THREE.Mesh(
        generateBoardGeometry(this.manager.getBoardSize()),
        boardFloorMaterial,
      );
      this.$parent.scene.add(this.object3D);
      this.generateEdges();
    });
  }

  private generateEdges() {
    const boardSize = this.manager.getBoardSize();
    const marginSize = this.manager.getMarginSize();
    const edgeGeometry = generateEdgeGeometry(boardSize, marginSize);
    const meshs: THREE.Mesh[] = [];

    for (let i = 0; i < 4; i += 1) {
      const mesh = new THREE.Mesh(edgeGeometry, boardMaterial);
      mesh.rotation.y = (Math.PI / 2) * i;
      switch (i) {
        case Edge.bottom: {
          mesh.position.z = -(boardSize / 2 + marginSize);
          break;
        }
        case Edge.right: {
          mesh.position.x = boardSize / 2 + marginSize;
          break;
        }
        case Edge.top: {
          mesh.position.z = boardSize / 2 + marginSize;
          break;
        }
        case Edge.left: {
          mesh.position.x = -(boardSize / 2 + marginSize);
          break;
        }
        default:
          break;
      }

      meshs.push(mesh);

      this.$parent.raycaster.on('pointerdown', [mesh], (e, inter, has) => {
        if (has) {
          this.manager.updateTiles(i);
        }
      });
      this.object3D.add(mesh);
    }

    let prev: THREE.Object3D | null = null;

    this.$parent.raycaster.on('pointermove', meshs, (e, inter, has) => {
      if (has) {
        prev = inter[0].object;
        // @ts-ignore
        inter[0].object.material.emissive.setHex(0x1979a9);
      } else if (prev) {
        // @ts-ignore
        prev.material.emissive.setHex(0x212121);
      }
    });
  }

  private startGame() {
    this.manager.resetTiles();
  }

  private endGame() {
    this.manager.resetTiles();
  }

  getBoardSize() {
    return this.manager.getBoardSize();
  }

  getMarginSize() {
    return this.manager.getMarginSize();
  }
}
</script>
