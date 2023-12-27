<template>
  <div ref="containerRef" class="virtual-table" @scroll="onScroll">
    <div class="virtual-table__list" :style="areaStyle">
      <div
        v-for="cell in cells"
        :key="`${cell.x}-${cell.y}`"
        class="virtual-table__cell"
        :style="cellStyle + cell.style"
      >
        {{ cell.x + 1 }}-{{ cell.y + 1 }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  withDefaults,
  computed,
  reactive,
  ref,
  onMounted,
  onBeforeUnmount,
  watch,
} from 'vue';

interface VirtualTableProps {
  /** Số lượng hàng của bảng */
  rows: number;
  /** Số lượng cột của bảng */
  columns: number;
  /** Chiều ngang 1 ô */
  cellWidth: number;
  /** Chiều dọc 1 ô */
  cellHeight: number;
}

interface Cell {
  /** Tọa độ ô: x */
  x: number;
  /** Tọa độ ô: y */
  y: number;
  /** Style riêng của ô */
  style: string;
}

const props = withDefaults(defineProps<VirtualTableProps>(), {});

/** Thông tin view port */
const viewPort = reactive({
  top: 0,
  left: 0,
  width: 0,
  height: 0,
});

/** Container ref */
const containerRef = ref<HTMLDivElement>();

/** Style của container */
const areaStyle = computed(() => {
  const width = props.cellWidth * props.columns;
  const height = props.cellHeight * props.rows;
  return `width: ${width}px; height: ${height}px`;
});

/** Style chung của các ô */
const cellStyle = computed(() => {
  return `width: ${props.cellWidth}px; height: ${props.cellHeight}px;`;
});

/** Danh sách ô sẽ hiển thị */
const cells = computed(() => {
  const container = containerRef.value;
  if (!container) {
    return [];
  }

  const yStartIndex = Math.floor(viewPort.top / props.cellHeight);
  const numberOfRows = Math.ceil(viewPort.height / props.cellHeight);
  const yEndIndex = yStartIndex + numberOfRows;

  const xStartIndex = Math.floor(viewPort.left / props.cellWidth);
  const numberOfColumns = Math.ceil(viewPort.width / props.cellWidth);
  const xEndIndex = xStartIndex + numberOfColumns;

  const cells: Cell[] = [];
  for (let y = yStartIndex; y < yEndIndex; y++) {
    for (let x = xStartIndex; x < xEndIndex; x++) {
      const top = y * props.cellHeight;
      const left = x * props.cellWidth;
      const style = `position: absolute; top: ${top}px; left: ${left}px;`;
      const cell: Cell = { x, y, style };
      cells.push(cell);
    }
  }

  return cells;
});

/** Hàm xử lý cuộn bảng */
const onScroll = (event: UIEvent) => {
  const element = event.currentTarget as HTMLDivElement;
  viewPort.top = element.scrollTop;
  viewPort.left = element.scrollLeft;
};

/** Hàm xử lý resize cửa sổ */
const onResize = () => {
  const container = containerRef.value;
  viewPort.width = container?.offsetWidth || 0;
  viewPort.height = container?.offsetHeight || 0;
};

/** Cập nhật view port khi container thay đổi */
watch(containerRef, (container) => {
  viewPort.width = container?.offsetWidth || 0;
  viewPort.height = container?.offsetHeight || 0;
});

onMounted(() => {
  window.addEventListener('resize', onResize);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', onResize);
});
</script>

<style>
.virtual-table {
  width: 100vw;
  height: 100vh;
  overflow: auto;
}

.virtual-table__list {
  position: relative;
}

.virtual-table__cell {
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px dashed #aaa;
  box-sizing: border-box;
}
</style>
